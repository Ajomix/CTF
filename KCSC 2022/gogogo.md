# gogogo
- Đây là một bài `WARM UP` và được viết bằng Golang
- Vì khi ném vào IDA mình nhận thấy nó bị strip đi khá nhiều và nó không rõ ràng các đầu vào của các hàm nên mình quyết định sẽ đọc code assembly để làm bài 
- Để Reverse được Golang các bạn có thể tham khảo link sau : [Reverse Golang](https://suvaditya.one/blog/2021/reversing-go/) , Vì bài này là một bài `WARM UP` nên sẽ không sử dụng những kiểu như interface hay hchannel....
### Load file vào IDA và thử bật debug 
- Nhận được lỗi `Corrupt...`,Nếu bạn cũng gặp phải trường hợp tương tự thì đây chỉ là raise exception của Go khi check runtime , bạn có thể tích vào dấu `Dont show Again` và `ok` để tiếp tục debug 
## Bắt đầu Phân tích
- Chúng ta sẽ bắt đầu từ hàm main (`main_main`)
  ```assembly 
  .text:00000000004ABF3B sub     rsp, 208h
  .text:00000000004ABF42 mov     [rsp+208h+var_8], rbp
  .text:00000000004ABF4A lea     rbp, [rsp+208h+var_8]
  .text:00000000004ABF52 mov     byte ptr [rsp+208h+var_A8], 0
  .text:00000000004ABF5A mov     [rsp+208h+var_A0], 0
  .text:00000000004ABF66 xorps   xmm0, xmm0
  .text:00000000004ABF69 movups  [rsp+208h+var_98], xmm0
  .text:00000000004ABF71 lea     rax, [rsp+208h+var_A8]
  .text:00000000004ABF79 mov     [rsp+208h+var_208], rax ; __int64
  .text:00000000004ABF7D lea     rax, a3988473094896b ; "3988473094896b834b1521f132c43be025f1361"...
  .text:00000000004ABF84 mov     [rsp+208h+var_200], rax ; __int64
  .text:00000000004ABF89 mov     [rsp+208h+var_1F8], 40h ; '@' ; __int64
  .text:00000000004ABF92 mov     [rsp+208h+var_1F0], 10h ; __int64
  .text:00000000004ABF9B nop     dword ptr [rax+rax+00h]
  .text:00000000004ABFA0 call    math_big__ptr_Int_SetString
  ```
- Khi đọc bài RE Golang mình đã để bên trên, hãy để ý đến phần function call , tác giả có nói rằng Go call hàm bằng cách thay đổi trực tiếp argument ở stack mà không dùng push hay pop ... : 
- Vậy nên mỗi khi có sử dụng call một hàm nào đó trong go thì các bạn hãy để ý các tham số truyền vào sẽ thường là ở đỉnh của stack , trong trường hợp này sẽ là `[rsp]` hay `[rsp+8]` ... , mình convert lại các var trong ida bằng cách nhấn 'Q' :
 
  ```assembly 
  .text:00000000004ABF3B sub     rsp, 208h
  .text:00000000004ABF42 mov     [rsp+208h+var_8], rbp
  .text:00000000004ABF4A lea     rbp, [rsp+208h+var_8]
  .text:00000000004ABF52 mov     byte ptr [rsp+208h+var_A8], 0
  .text:00000000004ABF5A mov     [rsp+208h+var_A0], 0
  .text:00000000004ABF66 xorps   xmm0, xmm0
  .text:00000000004ABF69 movups  [rsp+208h+var_98], xmm0
  .text:00000000004ABF71 lea     rax, [rsp+208h+var_A8]
  .text:00000000004ABF79 mov     [rsp], rax      ; __int64
  .text:00000000004ABF7D lea     rax, a3988473094896b ; "3988473094896b834b1521f132c43be025f1361"...
  .text:00000000004ABF84 mov     [rsp+8], rax    ; __int64
  .text:00000000004ABF89 mov     qword ptr [rsp+10h], 40h ; '@' ; __int64
  .text:00000000004ABF92 mov     qword ptr [rsp+18h], 10h ; __int64
  .text:00000000004ABF9B nop     dword ptr [rax+rax+00h]
  .text:00000000004ABFA0 call    math_big__ptr_Int_SetString

  ```


- Chương trình sử dụng hàm `SetString` để lưu được một số lớn :
  ![image](https://user-images.githubusercontent.com/57254763/168319957-4439f8eb-cdf4-4af6-90a8-2490078cdafb.png)
- Tham số đầu vào sẽ là chuỗi string kia , chuyển theo hệ 16 , bên dưới cũng tương tự :
```assembly 
  call    math_big__ptr_Int_SetString
  .text:00000000004ABFA5 mov     rax, [rsp+208h+var_1E8]
  .text:00000000004ABFAA mov     [rsp+208h+var_168], rax
  .text:00000000004ABFB2 mov     byte ptr [rsp+208h+var_C8], 0
  .text:00000000004ABFBA mov     [rsp+208h+var_C0], 0
  .text:00000000004ABFC6 xorps   xmm0, xmm0
  .text:00000000004ABFC9 movups  [rsp+208h+var_B8], xmm0
  .text:00000000004ABFD1 lea     rcx, [rsp+208h+var_C8]
  .text:00000000004ABFD9 mov     [rsp+208h+var_208], rcx ; __int64
  .text:00000000004ABFDD lea     rcx, a1bfd52469f1521 ; "1bfd52469f1521fa1e66ee1147ac74ea35b47b6"...
  .text:00000000004ABFE4 mov     [rsp+208h+var_200], rcx ; __int64
  .text:00000000004ABFE9 mov     [rsp+208h+var_1F8], 40h ; '@' ; __int64
  .text:00000000004ABFF2 mov     [rsp+208h+var_1F0], 10h ; __int64
  .text:00000000004ABFFB nop     dword ptr [rax+rax+00h]
  .text:00000000004AC000 call    math_big__ptr_Int_SetString
  
```
- Mình sẽ tạo một Note và Reverse chỗ code kia về code Golang và cứ thế đọc tiếp (Vì đây chỉ là phỏng đoán nên code mình viết ra sẽ hơi bẩn và có thể không chính xác hoàn toàn) :

```go 
package "main"

  import (
    "fmt"
    "big/math"
  )

  func main(){
    x := new(big.Int)
    sample1,_ := x.SetString("3988473094896b834b1521f132c43be025f1361be5a8b647136cccffd18d4999",16)  
    y := new(big.Int)
    _,sample2 := y.SetString("1bfd52469f1521fa1e66ee1147ac74ea35b47b6f4876b30b970538676d1f9664",16)  
  }
```

- Xong hai phần sử dụng hàm `Setstring`
- Tiếp tục 
	```assembly
		.text:00000000004AC000 call    math_big__ptr_Int_SetString
		.text:00000000004AC005 mov     rax, [rsp+208h+var_1E8]
		.text:00000000004AC00A mov     [rsp+208h+var_170], rax
		.text:00000000004AC012 xorps   xmm0, xmm0
		.text:00000000004AC015 movups  xmmword ptr [rsp+208h+var_128], xmm0
		.text:00000000004AC01D lea     rcx, qword_4B85A0
		.text:00000000004AC024 mov     [rsp+208h+var_128], rcx
		.text:00000000004AC02C lea     rdx, off_4F2E98 ; "[-] Enter flag: "
		.text:00000000004AC033 mov     [rsp+208h+var_128+8], rdx
		.text:00000000004AC03B mov     rdx, qword ptr cs:unk_572DF8
		.text:00000000004AC042 lea     rbx, off_4F4588
		.text:00000000004AC049 mov     [rsp+208h+var_208], rbx ; __int64
		.text:00000000004AC04D mov     [rsp+208h+var_200], rdx ; __int64
		.text:00000000004AC052 lea     rdx, [rsp+208h+var_128]
		.text:00000000004AC05A mov     [rsp+208h+var_1F8], rdx ; __int64
		.text:00000000004AC05F mov     [rsp+208h+var_1F0], 1 ; __int64
		.text:00000000004AC068 mov     [rsp+208h+var_1E8], 1 ; __int64
		.text:00000000004AC071 call    fmt_Fprint
		.text:00000000004AC076 lea     rax, qword_4B85A0
		.text:00000000004AC07D mov     [rsp+208h+var_208], rax ; __int64
		.text:00000000004AC081 call    runtime_newobject
		.text:00000000004AC086 mov     rax, [rsp+208h+var_200]
		.text:00000000004AC08B mov     [rsp+208h+var_150], rax
		.text:00000000004AC093 xorps   xmm0, xmm0
		.text:00000000004AC096 movups  xmmword ptr [rsp+208h+var_138], xmm0
		.text:00000000004AC09E lea     rcx, asc_4B60C0 ; "\b"
		.text:00000000004AC0A5 mov     [rsp+208h+var_138], rcx
		.text:00000000004AC0AD mov     [rsp+208h+var_138+8], rax
		.text:00000000004AC0B5 mov     rcx, qword ptr cs:unk_572DF0
		.text:00000000004AC0BC lea     rdx, off_4F4568
		.text:00000000004AC0C3 mov     [rsp+208h+var_208], rdx ; __int64
		.text:00000000004AC0C7 mov     [rsp+208h+var_200], rcx ; __int64
		.text:00000000004AC0CC lea     rcx, [rsp+208h+var_138]
		.text:00000000004AC0D4 mov     [rsp+208h+var_1F8], rcx ; __int64
		.text:00000000004AC0D9 mov     [rsp+208h+var_1F0], 1 ; __int64
		.text:00000000004AC0E2 mov     [rsp+208h+var_1E8], 1 ; __int64
		.text:00000000004AC0EB call    fmt_Fscan
		.text:00000000004AC0F0 mov     rax, [rsp+208h+var_1D8]
		.text:00000000004AC0F5 mov     rcx, [rsp+208h+var_1D0]
		.text:00000000004AC0FA cmp     [rsp+208h+var_1D8], 0
		.text:00000000004AC100 jz      loc_4AC19B
	```
- đoạn code trên trông sẽ như này khi viết về Golang: 
```go
	fmt.Println("[-] Enter flag: ")
	var input string
	fmt.Scan(input)
```
- Tiếp tục trace trong IDA và viết lại :
```go
package "main"

import (
	"fmt"
	"big/math"
)

func main(){
	x := new(big.Int)
	_,sample1 := x.SetString("3988473094896b834b1521f132c43be025f1361be5a8b647136cccffd18d4999",16)  
	
	y := new(big.Int)
	_,sample2 := y.SetString("1bfd52469f1521fa1e66ee1147ac74ea35b47b6f4876b30b970538676d1f9664",16)  
	fmt.Println("Enter Flag")
	var input string
	fmt.Scan(input)
	if len(name) >=5 {
		if name[0:5] != "KCSC{" && name[len(name)-1]!="}"{
			fail()
		}
		var tmpBuf [32]byte
		b := stringtoslicebyte(tmpBuf,input)
		hex_input := encoding_hex_Encode(b)
		if len(hex_input)!=32 {
			fail()
		}
		tmp := new(big.int)
		_,num3 := tmp.SetString(hex_input,16)
		num4 := SetString(hex_input[0:32/2],16)
		num5 := SetString(hex_input[32/2 : end],16)
		
		rs:=num4*num5
		num4:=num4*num4
		num5:=num5*num5
		tmp2:=num4+num5 
		if tmp2!=sample1 && rs != sample2{
			fail()
		}
	}
}
```

- Đến đây thì quá rõ ràng rồi , chúng ta sẽ sử dụng z3 để giải bài , script mình dùng để giải :

```python 

from z3 import *
s=Solver()
sample1=0x3988473094896b834b1521f132c43be025f1361be5a8b647136cccffd18d4999
sample2=0x1bfd52469f1521fa1e66ee1147ac74ea35b47b6f4876b30b970538676d1f9664
part1=[BitVec("s1[%d]"%i,8)for i in range(16)]
for i in range(len(part1)):
    s.add(part1[i]>=0x21)
    s.add(part1[i]<=0x7e)

part2=[BitVec("s2[%d]"%i,8)for i in range(16)]
for i in range(len(part2)):
    s.add(part2[i]>=0x21)
    s.add(part2[i]<=0x7e)

sym1=Concat(part1)
sym2=Concat(part2)
rs=sym1*sym2
sym1=sym1*sym1
sym2=sym2*sym2
tmp2=sym1+sym2
s.add(tmp2==sample1)
s.add(rs==sample2)

if s.check():
    m = s.model()
    for i in range(16):
        print(chr(m[part1[i]].as_long()),end='')
    for i in range(16):
        print(chr(m[part2[i]].as_long()),end='')
else:
    print("Fail")
    
```
# Flag : KCSC{1+1=2_2+2=4_4+4=8_8+8=16!!}
