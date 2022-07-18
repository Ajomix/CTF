# Rolling 
- Đây là một bài reverse android 
- Sau khi phân tích chúng ta dễ thấy được chương trình sử dụng Lib Native để check Flag nhập vào:
![image](https://user-images.githubusercontent.com/57254763/179475803-9a49be37-4e1c-4f72-870e-f4c9c65632dd.png)

- Chúng ta chỉ cần brute force để lấy được flag ( tool : Unicorn để giả lập arm ):
## Script: 
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


