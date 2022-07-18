# Rolling 
- Đây là một bài reverse android 
- Sau khi phân tích chúng ta dễ thấy được chương trình sử dụng Lib Native để check Flag nhập vào:
![image](https://user-images.githubusercontent.com/57254763/179475803-9a49be37-4e1c-4f72-870e-f4c9c65632dd.png)

- Chúng ta chỉ cần brute force để lấy được flag ( tool : Unicorn để giả lập arm ):


## Script Golang: 
```golang
package main

import (
	"fmt"
	"io/ioutil"

	uc "github.com/unicorn-engine/unicorn/bindings/go/unicorn"
)

var (
	STACK_BASE  = 0x0
	STACK_SIZE  = 5 * 1024 * 1024
	HEAP_BASE   = STACK_SIZE
	HEAP_SIZE   = 1024 * 1024
	base_module = 0x140000000
)

func addHooks(simulator uc.Unicorn) {
	simulator.HookAdd(uc.HOOK_CODE, func(mu uc.Unicorn, addr uint64, size uint32) {
		//fmt.Printf("Code: 0x%x, 0x%x\n", addr, size)
		//meatbox controller
		if addr == uint64(base_module)+0x174C { //bypass .strlen
			simulator.RegWrite(uc.ARM64_REG_X0, 1)
			simulator.RegWrite(uc.ARM64_REG_PC, addr+uint64(size))
		}
		if addr == uint64(base_module)+0xc9c { //bypass .step_meatbox
			mu.RegWrite(uc.ARM64_REG_X17, uint64(base_module)+0x12EC)
		}

		if addr == uint64(base_module)+0xCFC { //bypass .finalize
			simulator.RegWrite(uc.ARM64_REG_X17, uint64(base_module)+0x11AC)
		}

		if addr == uint64(base_module)+0x1AD4 { //bypass .malloc
			simulator.RegWrite(uc.ARM64_REG_PC, uint64(size)+addr)
		}

		//soulbox controller
		if addr == uint64(base_module)+0x246C {
			simulator.RegWrite(uc.ARM64_REG_X0, 1)
			simulator.RegWrite(uc.ARM64_REG_PC, addr+uint64(size))
		}
		if addr == uint64(base_module)+0xD7C { //bypass .soulboxstep
			simulator.RegWrite(uc.ARM64_REG_X17, uint64(base_module)+0x1ECC)
		}

		if addr == uint64(base_module)+0xCEC { //bypass .finalize
			simulator.RegWrite(uc.ARM64_REG_X17, uint64(base_module)+0x200C)
		}

		if addr == uint64(base_module)+0x27F4 { //bypass .malloc
			simulator.RegWrite(uc.ARM64_REG_PC, uint64(size)+addr)
		}

		//godbox controller
		if addr == uint64(base_module)+0x03194 {
			simulator.RegWrite(uc.ARM64_REG_X0, 1)
			simulator.RegWrite(uc.ARM64_REG_PC, addr+uint64(size))
		}
		if addr == uint64(base_module)+0xCAC {
			simulator.RegWrite(uc.ARM64_REG_X17, uint64(base_module)+0x2BEC)
		}
		if addr == uint64(base_module)+0xD3C {
			simulator.RegWrite(uc.ARM64_REG_X17, uint64(base_module)+0x02D30)
		}
		if addr == uint64(base_module)+0x351C { //bypass .malloc
			simulator.RegWrite(uc.ARM64_REG_PC, uint64(size)+addr)
		}
	}, 1, 0)
}

func main() {
	asc_box := []int{7, 24, 16, 15, 28, 18, 5, 10, 7, 11, 2, 15, 18, 6, 8, 19, 10, 7, 5, 9, 11, 6, 15, 15, 17, 4, 19, 19, 1, 14, 3, 11, 0, 1, 1, 9, 9, 2, 8, 19, 1, 14, 1, 1, 12, 9, 5, 16, 1, 18, 10, 8, 11, 18, 17, 4, 19, 1, 1, 12, 19, 1, 14, 18, 0, 14, 8, 11, 18, 1, 15, 11, 3, 11, 0, 1, 1, 12, 7, 5, 4, 8, 11, 18, 8, 24, 15, 8, 24, 15, 14, 28, 15, 1, 18, 10, 16, 21, 17, 1, 1, 12, 6, 22, 10, 8, 11, 18, 17, 4, 19, 1, 18, 10, 1, 1, 12, 14, 28, 15, 1, 18, 10, 1, 1, 12, 3, 11, 0, 9, 2, 8, 4, 13, 16, 1, 1, 12, 6, 22, 10, 4, 13, 16, 4, 13, 16, 17, 15, 5, 7, 23, 2}

	lib_bytes, e := ioutil.ReadFile("../libnative-lib.so")
	size_mod := 0x8000
	if e != nil {
		fmt.Println("Failed to Read file")
	}
	simulator, _ := uc.NewUnicorn(uc.ARCH_ARM64, uc.MODE_ARM)

	if e = simulator.MemMap(uint64(base_module), uint64(size_mod)); e != nil {
		fmt.Println(e)
	}
	if e = simulator.MemWrite(uint64(base_module), lib_bytes); e != nil {
		fmt.Println(e)
	}
	if e = simulator.MemMap(uint64(STACK_BASE), uint64(STACK_SIZE)); e != nil {
		fmt.Println(e)
	}
	if e = simulator.MemMap(uint64(HEAP_BASE), uint64(HEAP_SIZE)); e != nil {
		fmt.Println(e)
	}
	if e = simulator.RegWrite(uc.ARM64_REG_SP, uint64(STACK_BASE+STACK_SIZE-1)); e != nil {
		fmt.Println(e)
	}

	addHooks(simulator)
	counter := 0

	for i := 0; i < len(asc_box); i += 3 {

		for input := 0x20; input <= 0x7f; input++ {
			simulator.MemWrite(uint64(HEAP_BASE), []byte{byte(input)})

			start_address := uint64(base_module) + 0x1708
			end_address := uint64(base_module) + 0x1ADC
			simulator.RegWrite(uc.ARM64_REG_X0, uint64(HEAP_BASE))
			simulator.Start(start_address, end_address)
			ret_meatbox, _ := simulator.RegRead(uc.ARM64_REG_Q0)
			ret_meatbox = ret_meatbox & 0xff

			if ret_meatbox == uint64(asc_box[i]) {

				//fmt.Printf("Candicate soulbox :%c\n", input)
				start_address = uint64(base_module) + 0x2428
				end_address = uint64(base_module) + 0x27FC
				simulator.RegWrite(uc.ARM64_REG_X0, uint64(HEAP_BASE))
				simulator.Start(start_address, end_address)
				ret_soulbox, _ := simulator.RegRead(uc.ARM64_REG_Q0)
				ret_soulbox = ret_soulbox & 0xff

				if ret_soulbox == uint64(asc_box[i+1]) {
					//fmt.Printf("Candicate godbox :%c\n", input)
					start_address = uint64(base_module) + 0x314C
					end_address = uint64(base_module) + 0x3524
					simulator.RegWrite(uc.ARM64_REG_X0, uint64(HEAP_BASE))
					simulator.Start(start_address, end_address)
					ret_godbox, _ := simulator.RegRead(uc.ARM64_REG_Q0)
					ret_godbox &= 0xff

					if ret_godbox == uint64(asc_box[i+2]) {
						//fmt.Printf("%c", input)
						HEAP_BASE++
						counter++
						input = 0x7f + 1
					}
				}
			}
		}
	}

	flag, _ := simulator.MemRead(uint64(HEAP_BASE)-uint64(counter), uint64(counter))
	fmt.Println(string(flag))
}


```
## Script Python: 
```python

from unicorn import *
from unicorn.arm64_const import *
import ctypes
UC_MEM_ALIGN = 0x1000
module_base = 0x140000000
STACK_SIZE = 1024*1024*5
BASE_STACK = 0x0
BASE_HEAP = 1024*1024*5
HEAP_SIZE = 1024*1024

def align(addr, size, growl):
    to = ctypes.c_uint64(UC_MEM_ALIGN).value
    mask = ctypes.c_uint64(0xFFFFFFFFFFFFFFFF).value ^ ctypes.c_uint64(to - 1).value
    right = addr + size
    right = (right + to - 1) & mask
    addr &= mask
    size = right - addr
    if growl:
        size = (size + to - 1) & mask
    return addr, size

def hook_code_meatbox(simulator, address, size, user_data):
    #print(">>> Tracing instruction at 0x%x, instruction size = 0x%x" %(address, size))
    if address == module_base + 0x174C: # bypass .strlen
        simulator.reg_write(UC_ARM64_REG_X0,1)
        simulator.reg_write(UC_ARM64_REG_PC,address+size)
    if address == module_base + 0xC9C:#bypass .meatbox
        simulator.reg_write(UC_ARM64_REG_X17,module_base+0x12EC)
    if address == module_base + 0xCFC:#bypass .finalize
        simulator.reg_write(UC_ARM64_REG_X17,module_base+0x11AC)
    if address == module_base + 0x1AD4:#bypass .malloc
        simulator.reg_write(UC_ARM64_REG_PC,address+size)
    pass
def hook_code_soulbox(simulator, address, size, user_data):
    #print(">>> Tracing instruction at 0x%x, instruction size = 0x%x" %(address, size))
    if address == module_base+0x246C:# bypass .strlen
        simulator.reg_write(UC_ARM64_REG_X0,1)
        simulator.reg_write(UC_ARM64_REG_PC,address+size)
    if address == module_base+0xD7C:#bypass .soulboxstep
        simulator.reg_write(UC_ARM64_REG_X17,module_base+0x1ECC)
    if address == module_base + 0xCEC:#bypass .finalize
        simulator.reg_write(UC_ARM64_REG_X17,module_base+0x200C)
    if address == module_base + 0x27F4:#bypass .malloc
        simulator.reg_write(UC_ARM64_REG_PC,address+size)
    pass
def hook_code_godbox(simulator, address, size, user_data):
    #print(">>> Tracing instruction at 0x%x, instruction size = 0x%x" %(address, size))
    if address == module_base+0x3194:# bypass .strlen
        simulator.reg_write(UC_ARM64_REG_X0,1)
        simulator.reg_write(UC_ARM64_REG_PC,address+size)
    if address == module_base+0xCAC:#bypass .godboxstep
        simulator.reg_write(UC_ARM64_REG_X17,module_base+0x2BEC )
    if address == module_base + 0xD3C:#bypass .finalize
        simulator.reg_write(UC_ARM64_REG_X17,module_base+0x2D30)
    if address == module_base + 0x351C:#bypass .malloc
        simulator.reg_write(UC_ARM64_REG_PC,address+size)
    pass

try:
   
    asc_box = [7, 24, 16, 15, 28, 18, 5, 10, 7, 11, 2, 15, 18, 6, 8, 19, 10, 7, 5, 9, 11, 6, 15, 15, 17, 4, 19, 19, 1, 14, 3, 11, 0, 1, 1, 9, 9, 2, 8, 19, 1, 14, 1, 1, 12, 9, 5, 16, 1, 18, 10, 8, 11, 18, 17, 4, 19, 1, 1, 12, 19, 1, 14, 18, 0, 14, 8, 11, 18, 1, 15, 11, 3, 11, 0, 1, 1, 12, 7, 5, 4, 8, 11, 18, 8, 24, 15, 8, 24, 15, 14, 28, 15, 1, 18, 10, 16, 21, 17, 1, 1, 12, 6, 22, 10, 8, 11, 18, 17, 4, 19, 1, 18, 10, 1, 1, 12, 14, 28, 15, 1, 18, 10, 1, 1, 12, 3, 11, 0, 9, 2, 8, 4, 13, 16, 1, 1, 12, 6, 22, 10, 4, 13, 16, 4, 13, 16, 17, 15, 5, 7, 23, 2]

    with open("libnative-lib.so","rb") as fstream :
        lib_bytes = fstream.read()
        (ADDRESS,size) = align(module_base,len(lib_bytes),True)
        simulator = Uc(UC_ARCH_ARM64,UC_MODE_ARM)
        simulator.mem_map(ADDRESS,size,UC_PROT_ALL)
        simulator.mem_write(ADDRESS,lib_bytes)
        simulator.mem_map(BASE_STACK,STACK_SIZE)
        simulator.mem_map(BASE_HEAP,HEAP_SIZE)

        simulator.reg_write(UC_ARM64_REG_SP, BASE_STACK + STACK_SIZE - 1)

        counter = 0
        for iiii in range(0,len(asc_box),3):
            for input in range(0x20,0x7f+1):
                #meat_box
                simulator.mem_write(BASE_HEAP,input.to_bytes(1, 'little'))
                simulator.reg_write(UC_ARM64_REG_X0,BASE_HEAP)
                start_address = module_base + 0x1708
                end_address = module_base + 0x1ADC
                simulator.hook_add(UC_HOOK_CODE, hook_code_meatbox, start_address, end_address)
                simulator.emu_start(start_address,end_address)

                ret_meatbox = simulator.reg_read(UC_ARM64_REG_Q0)&0xff

                if ret_meatbox == asc_box[iiii]:
                    print("candicate meatbox: %c"%input)
                    #soul_box
                    simulator.mem_write(BASE_HEAP,input.to_bytes(1, 'little'))
                    simulator.reg_write(UC_ARM64_REG_X0,BASE_HEAP)
                    start_address = module_base + 0x2428
                    end_address = module_base + 0x27FC
                    simulator.hook_add(UC_HOOK_CODE, hook_code_soulbox, start_address, end_address)
                    simulator.emu_start(start_address,end_address)

                    ret_soulbox = simulator.reg_read(UC_ARM64_REG_Q0)&0xff
                    if ret_soulbox == asc_box[iiii+1]:
                        print("candicate soulbox: %c"%input)
                        #godbox 
                        simulator.mem_write(BASE_HEAP,input.to_bytes(1, 'little'))
                        simulator.reg_write(UC_ARM64_REG_X0,BASE_HEAP)
                        start_address = module_base + 0x314C
                        end_address = module_base + 0x3524
                        simulator.hook_add(UC_HOOK_CODE, hook_code_godbox, start_address, end_address)
                        simulator.emu_start(start_address,end_address)

                        ret_godbox = simulator.reg_read(UC_ARM64_REG_Q0)&0xff
                        if ret_godbox == asc_box[iiii+2]:
                            print("result : %c"%input)
                            counter += 1 
                            BASE_HEAP += 1
                            input=0x7f+2
        
                    
        flag = simulator.mem_read(BASE_HEAP-counter,counter)
        print(flag)
except UcError as e :
    print("Error {}".format(e))

    #LINECTF{watcha_kn0w_ab0ut_r0ll1ng_d0wn_1n_th3_d33p}

```
- Kết quả : 

![image](https://user-images.githubusercontent.com/57254763/179476137-5971496b-d6da-4fd2-aff5-aef196ab22a9.png)


