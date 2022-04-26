## Crackmexx
- File thực thi là kiểu file ELF32 
Khi mở file bằng IDA hàm MAIN :
```C
int __fastcall main(int a1, char **a2, char **a3)
{
  int j; // [rsp+18h] [rbp-28h]
  int i; // [rsp+1Ch] [rbp-24h]

  if ( a1 <= 1 )
    puts("need a flag!");
  for ( i = 0; i <= 31336; ++i )
  {
    for ( j = 0; j <= 551; ++j )
      *((_BYTE *)&loc_4006E5 + j) = sub_400526((unsigned int)*((char *)&loc_4006E5 + j));
  }
  if ( ((__int64 (*)(void))loc_4006E5)() )
    return puts("Correct!");
  else
    return puts("Wrong!");
}

```
- Trước tiên chương trình check xem `agrv` của chúng ta có bao nhiêu phần tử  `` (a1 <=1 ) { ....`` vì thế trước tiên chúng ta sẽ cung cấp một flag giả để có thể hiểu được luồng chương trình : `./crackmexx fakeflag`
- Chúng ta có thể nhận thấy dễ dàng hai vòng `for` lồng nhau như trên đang thực hiện decode khoảng vài hơn 500 byte rồi sử dụng số byte đó như một "shellcode" để thực thi, nên Ý tưởng ở bài này sẽ là bật chức năng `debug` để có thể lấy được toàn bộ đoạn code đã được decode và từ đó chúng ta có thể trace tiếp.
# Hướng làm bài : 
  - `Debug` để lấy được hàm sẽ thực thi
  - Trace INPUT xem hàm đó sẽ làm gì với INPUT để ra được kết quả đúng
# Debug để lấy CODE :
  - Chúng ta sẽ đặt breakpoint ở ví trí trước khi thực thi CODE ( `CALL loc_4006E5`)
  ```assembly
.text:00000000004006BE                 cmp     [rbp+var_28], 227h
.text:00000000004006C5                 jle     short loc_400694
.text:00000000004006C7                 add     [rbp+var_24], 1
.text:00000000004006CB
.text:00000000004006CB loc_4006CB:                             ; CODE XREF: main+3B↑j
.text:00000000004006CB                 cmp     [rbp+var_24], 7A68h
.text:00000000004006D2                 jle     short loc_40068B
.text:00000000004006D4
.text:00000000004006D4 loc_4006D4:                             ; CODE XREF: .text:0000000000400727↓j
.text:00000000004006D4                 mov     rdx, [rbp+var_20]
.text:00000000004006D8                 mov     rax, rdx
.text:00000000004006DB                 call    loc_4006E5 <--------------------SET BREAK POINT
.text:00000000004006E0                 jmp     loc_40090D
  ```
 - Sau khi run chương trình chạy đến đoạn đã set breakpoint chúng ta sử dụng chức năng make code ( `press C` ) của IDA để đọc được CODE (`loc_4006E5`)
 - Kết quả : 
 ```assembly

.text:00000000004006E5 ; =============== S U B R O U T I N E =======================================
.text:00000000004006E5
.text:00000000004006E5 ; Attributes: bp-based frame
.text:00000000004006E5
.text:00000000004006E5 loc_4006E5 proc near                    ; CODE XREF: main+8D↑p
.text:00000000004006E5                                         ; DATA XREF: main+4B↑o
.text:00000000004006E5 push    rbp
.text:00000000004006E6 mov     rbp, rsp
.text:00000000004006E9 call    loc_400702
.text:00000000004006EE mov     cl, al
.text:00000000004006F0 mov     eax, 0AE5FE432h
.text:00000000004006F5 add     eax, 51A01BCEh
.text:00000000004006FA inc     eax
.text:00000000004006FC jnb     short near ptr loc_400705+1
.text:00000000004006FE mov     al, cl
.text:0000000000400700 leave
.text:0000000000400701 retn
.text:0000000000400701 loc_4006E5 endp
.text: .......
 ```
 - Sau khi đã có CODE chúng ta cần biết xem chỗ hàm này sẽ làm gì với `INPUT` của chúng ta 
 - Trước khi chạy hàm `loc_4006E5` chương trình đã đưa `INPUT` vào thanh ghi `RAX` :
 ```assembly
.text:00000000004006D4 mov     rdx, [rbp+var_20] <---------------- var_20 đang chứa INPUT 
.text:00000000004006D8 mov     rax, rdx
.text:00000000004006DB call    loc_4006E5
.text:00000000004006E0 jmp     loc_40090D
.text:00000000004006E0 main endp
 ```
 - Công việc tiếp theo của chúng ta sau đó sẽ là `debug` vì hàm này đã bị anti analysis nên chúng ta chỉ có thế đọc code Assembly và trace xem chương trình `ENCRYPT INPUT` của chúng ta ra sao.
 - Sau một lúc ngồi đọc CODE Asm của hàm này chúng ta thu được cách chương trình encrypt như sau : 
 ```
 FOR j in (0..1337) {
    FOR i in (0.. len(INPUT)) {
        tmp=0x50
        INPUT[i]^=tmp
        tmp^=INPUT[i]
    }
 }

```
- Sau khi Encrypt chương trình sẽ thực hiện so sánh với 33 byte có sẵn trong hàm nên bài này chúng ta sẽ sử dụng Z3 để giải:
```python 
from z3 import *
s=Solver()
data=[  
  0x48, 0x5F, 0x36, 0x35, 0x35, 0x25, 0x14, 0x2C, 0x1D, 0x01, 
  0x03, 0x2D, 0x0C, 0x6F, 0x35, 0x61, 0x7E, 0x34, 0x0A, 0x44, 
  0x24, 0x2C, 0x4A, 0x46, 0x19, 0x59, 0x5B, 0x0E, 0x78, 0x74, 
  0x29, 0x13, 0x2C
]
flag = [BitVec("%d"%i,8) for i in range(len(data))]
tmp = 0x50
for i in range(1337):
    for j in range(len(flag)):
        flag[j]^=tmp 
        tmp^=flag[j]
for i in range(len(flag)):
    s.add(flag[i]==data[i])

print(s.check())
m = s.model()
print(m)
# flag[18] = 111
# flag[20] = 102
# flag[31] = 33
# flag[32] = 125
# flag[30] = 101
# flag[13] = 110
# flag[17] = 111
# flag[10] = 116
# flag[15] = 95
# flag[21] = 97
# flag[7] = 95
# flag[23] = 99
# flag[5] = 111
# flag[6] = 107
# flag[26] = 116
# flag[8] = 110
# flag[0] = 112
# flag[2] = 116
# flag[12] = 105
# flag[3]= 102
# flag[16] = 116
# flag[28] = 101
# flag[14] = 103
# flag[27] = 104
# flag[29] = 114
# flag[22] = 110
# flag[9] = 111
# flag[11] = 104
# flag[1] = 99
# flag[25] = 95
# flag[19] = 95
# flag[24] = 121
# flag[4] = 123
# print(bytes(flag))

```
FLAG : `b'pctf{ok_nothing_too_fancy_there!}'`
