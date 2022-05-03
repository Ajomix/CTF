# FlagChecker
- Chạy thử chương trình :

  ![image](https://user-images.githubusercontent.com/57254763/166267071-ec1a5c1c-0a00-4498-b2f3-d47423342ac0.png)
- Có thể thấy chúng ta sẽ phải nhập flag đúng vào ô trống trên , theo thói quen , mình ném file vào `DIE`


  ![image](https://user-images.githubusercontent.com/57254763/166264225-66620b21-9811-4e08-8695-81775b86375d.png)

- File BIN : 
   +  Là file `EXE PE32` 
   +  Được Compile bằng `AUTOIT`
   +  Đã được `pack` bằng `UPX`
- Trước tiên việc cần làm là chúng ta phải `unpack` file BIN này , có thể dùng tool tại trang chủ của `upx` [tool](https://upx.github.io)
- Sau khi đã `unpack` được , chúng ta cần phải decompile file BIN này về code `autoIT` vì nó viết bằng `autoIT` và compile về `PE file` để chạy 
- Để convert chúng ta có thể sử dụng tool của tác giả `x0r19x91` :  [UnautoIT](https://github.com/x0r19x91/UnAutoIt)

## [+] Sau khi đã làm được tất cả các bước trên chúng ta được một script đã được compile về code `AutoIT`:
```autoit
......
While 0x1
    $nmsg = GUIGetMsg()
    Switch $nmsg
    Case $GUI_EVENT_CLOSE
        Exit
    Case $bntcheck
        checker()
        Sleep(0x64)
    EndSwitch
WEnd
Func checker()
    Local $input = GUICtrlRead($iflag)
    Local $len_input = StringLen($input)
    Local $opcode = "0x558bec83ec6c8d45d850e8aa05000083c4048d4d94518d55d8" & "52e8cb03000083c408e80c0000007573657233322e646c6c00" & "00ff55d88945f8837df8007505e9fb000000e80c0000004d65" & "7373616765426f7841008b45f850e8b306000083c4088945f0" & "8b4d0851ff55e883f81c740b8b550cc60200e9c4000000c645" & "bcf8c645bd50c645beccc645bfefc645c0e6c645c13cc645c2" & "35c645c396c645c41dc645c561c645c6aec645c7c0c645c8c5" & "c645c931c645cacec645cbb0c645cce7c645cd1dc645ceedc6" & "45cfbcc645d05dc645d181c645d269c645d38ac645d435c645" & "d574c645d657c645d7b68b4508508d4d94518d55d852e84700" & "000083c40c8945f4c745fc00000000eb098b45fc83c0018945" & "fc837dfc1c7d1f8b4df4034dfc0fb6118b45fc0fb64c05bc3b" & "d174088b550cc60200eb08ebd28b450cc600018be55dc3558b" & "ec83ec445657b90b000000e82c00000068747470733a2f2f77" & "77772e796f75747562652e636f6d2f77617463683f763d6451" & "773477395767586351005e8d7dbcf3a5c745f800000000c745" & "f400000000c745fc000000008b4510508b4d088b5110ffd289" & "45ec6a006a016a006a008d45f8508b4d0c8b11ffd285c07507" & "33c0e91b0200008d45f4506a006a0068048000008b4df8518b" & "550c8b420cffd085c075156a008b4df8518b550c8b4224ffd0" & "33c0e9e9010000837df40075156a008b4df8518b550c8b4224" & "ffd033c0e9ce0100006a008d4dbc518b55088b4210ffd0508d" & "4dbc518b55f4528b450c8b4808ffd185c075218b55f4528b45" & "0c8b481cffd16a008b55f8528b450c8b4824ffd133c0e98a01" & "00008d55fc526a008b45f45068016800008b4df8518b550c8b" & "4214ffd085c075218b4df4518b550c8b421cffd06a008b4df8" & "518b550c8b4224ffd033c0e94a0100008b4df4518b550c8b42" & "1cffd06a008d4dec516a006a006a016a008b55fc528b450c8b" & "4810ffd185c07527837dfc0074218b55fc528b450c8b4820ff" & "d16a008b55f8528b450c8b4824ffd133c0e9f90000006a0468" & "001000008b55ec83c201526a008b45088b4808ffd18945e883" & "7de8007527837dfc0074218b55fc528b450c8b4820ffd16a00" & "8b55f8528b450c8b4824ffd133c0e9b10000008b55ec83c201" & "528b45e850e8cc06000083c408c745f000000000eb098b4df0" & "83c101894df08b55f03b55ec73128b45e80345f08b4d10034d" & "f08a118810ebdd8b4510508b4d088b5110ffd2508d45ec508b" & "4de8516a006a016a008b55fc528b450c8b4810ffd185c07524" & "837dfc00741e8b55fc528b450c8b4820ffd16a008b55f8528b" & "450c8b4824ffd133c0eb23837dfc00741a8b55fc528b450c8b" & "4820ffd16a008b55f8528b450c8b4824ffd18b45e85f5e8be5" & "5dc3558bec51e81000000061647661706933322e646c6c0000" & "00008b45088b08ffd18945fc837dfc00750732c0e99b010000" & "e818000000437279707441637175697265436f6e7465787441" & "000000008b55fc52e8d102000083c4088b4d0c8901e8100000" & "00437279707443726561746548617368008b55fc52e8ab0200" & "0083c4088b4d0c89410ce8100000004372797074496d706f72" & "744b657900008b55fc52e88402000083c4088b4d0c894104e8" & "1000000043727970744465726976654b657900008b55fc52e8" & "5d02000083c4088b4d0c894114e81000000043727970744861" & "7368446174610000008b55fc52e83602000083c4088b4d0c89" & "4108e8100000004372797074456e6372797074000000008b55" & "fc52e80f02000083c4088b4d0c894110e81400000043727970" & "7447657448617368506172616d0000008b55fc52e8e4010000" & "83c4088b4d0c894118e814000000437279707444657374726f" & "7948617368000000008b55fc52e8b901000083c4088b4d0c89" & "411ce810000000437279707444657374726f794b6579008b55" & "fc52e89201000083c4088b4d0c894120e81400000043727970" & "7452656c65617365436f6e74657874008b55fc52e867010000" & "83c4088b4d0c894124b0018be55dc3558bec83ec18e81c0000" & "006b00650072006e0065006c00330032002e0064006c006c00" & "00000000e88602000083c4048945fc837dfc00750732c0e915" & "010000e8100000004c6f61644c69627261727941000000008b" & "45fc50e8fb00000083c4088945f8837df800750732c0e9e400" & "0000e81000000047657450726f634164647265737300008b4d" & "fc51e8ca00000083c4088945f4837df400750732c0e9b30000" & "00e8100000005669727475616c416c6c6f63000000008b55fc" & "52e89900000083c4088945f0837df000750732c0e982000000" & "e80c0000005669727475616c46726565008b45fc50e86c0000" & "0083c4088945ec837dec00750432c0eb58e80c0000006c7374" & "726c656e41000000008b4dfc51e84200000083c4088945e883" & "7de800750432c0eb2e8b55088b45f889028b4d088b55f48951" & "048b45088b4df08948088b55088b45ec89420c8b4d088b55e8" & "895110b0018be55dc3558bec83ec3c8b45088945ec8b4dec0f" & "b71181fa4d5a0000740733c0e9350100008b45ec8b4d080348" & "3c894de4ba080000006bc2008b4de48d5401788955e88b45e8" & "833800750733c0e9080100008b4de88b118955e08b45e00345" & "088945f48b4df48b51188955dc8b45f48b481c894dd08b55f4" & "8b42208945d88b4df48b51248955d4c745f800000000eb098b" & "45f883c0018945f88b4df83b4ddc0f83b30000008b55080355" & "d88b45f88d0c82894dc88b55080355d48b45f88d0c42894dcc" & "8b55080355d08b45cc0fb7088d148a8955c48b45c88b4d0803" & "08894df0c745fc00000000c745fc00000000eb098b55fc83c2" & "018955fc8b450c0345fc0fbe0885c974278b55f00355fc0fbe" & "0285c0741a8b4d0c034dfc0fbe118b45f00345fc0fbe083bd1" & "7402eb02ebc38b550c0355fc0fbe0285c075198b4df0034dfc" & "0fbe1185d2750c8b45c48b4d0803088bc1eb07e938ffffff33" & "c08be55dc3558bec83ec34c745e40000000064a13000000089" & "45e48b4de48b510c8955d88b45d88b480c8b5010894dcc8955" & "d08b45cc8945d48b4dd4894de8837de8000f845a0100008b55" & "e8837a18000f844d0100008b45e8837830007502ebde8b4de8" & "8b51308955ecc745f000000000c745f000000000eb098b45f0" & "83c0018945f08b4df08b55080fb7044a85c00f84dd0000008b" & "4df08b55ec0fb7044a85c00f84cb0000008b4df08b55080fb7" & "044a83f85a7f378b4df08b55080fb7044a83f8417c288b4df0" & "8b55080fb7044a83c0208945e08b4df08b5508668b45e06689" & "044a668b4de066894dfeeb0e8b55f08b4508668b0c5066894d" & "fe668b55fe668955f88b45f08b4dec0fb7144183fa5a7f378b" & "45f08b4dec0fb7144183fa417c288b45f08b4dec0fb7144183" & "c2208955dc8b45f08b4dec668b55dc66891441668b45dc6689" & "45fceb0e8b4df08b55ec668b044a668945fc668b4dfc66894d" & "f40fb755f80fb745f43bd07402eb05e908ffffff8b4df08b55" & "080fb7044a85c075168b4df08b55ec0fb7044a85c075088b4d" & "e88b4118eb0f8b55e88b028945e8e99cfeffff33c08be55dc3" & "558bec518b45088945fc837d0c00741a8b4dfcc601008b55fc" & "83c2018955fc8b450c83e80189450cebe08b45088be55dc300" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000000000" & "00000000000000000000000000000000000000000000"
    Local $opcode_buf = DllStructCreate("byte[" & BinaryLen($opcode) & "]")
    DllStructSetData($opcode_buf, 0x1, Binary($opcode))
    Local $input_buf = DllStructCreate("byte[" & BinaryLen($input) + 0x1 & "]")
    DllStructSetData($input_buf, 0x1, Binary($input))
    Local $is_flag = DllStructCreate("byte[1]")
    DllStructSetData($is_flag, 0x1, Binary("0x00"))
    DllCall("user32.dll", "none", "CallWindowProcA", "ptr", DllStructGetPtr($opcode_buf), "ptr", DllStructGetPtr($input_buf), "ptr", DllStructGetPtr($is_flag), "int", 0x0, "int", 0x0)
    If DllStructGetData($is_flag, 0x1) == "0x01" Then
        MsgBox(0x0, '', "Correct!")
    Else
        MsgBox($MB_ICONERROR, '', "Incorrect!")
    EndIf
EndFunc    ; -> checker

......
```
- Đọc kĩ hàm trên thì chúng ta sẽ thấy rằng chương trình sẽ sử dụng đống `opcode` là `shellcode` để check FLAG có đúng hay không
- Chương trình gọi `DllCall` và gọi ra hàm `CallWindowProcA` để thực thi `shellcode` check `Flag`
- Lợi dụng điều này mình dự định sẽ đặt 1 breakpoint vào hàm `CallWindowProcA` để lấy được `shellcode` và dễ dàng cho việc phân tích hơn:

  ![image](https://user-images.githubusercontent.com/57254763/166268471-5b941514-d0d5-43f4-9782-f34b753f7504.png)
- Nhưng khi bật debug mình gặp lỗi này :


  ![image](https://user-images.githubusercontent.com/57254763/166268585-ea9fe567-1dd4-4180-b61b-a53653f981fc.png)

- Trước tiên cần `bypass` qua auto check debug của autoIT: 

  ![image](https://user-images.githubusercontent.com/57254763/166268144-d9047ecd-211b-4038-b5ca-0af609190d0e.png)
  
 - Đặt BreakPoint tại hàm `CallWindowProcA` : 

  ![image](https://user-images.githubusercontent.com/57254763/166268817-6eb655f7-7a3f-42f0-8ae8-72ce86a00b94.png)
  
  - Cho chương trình chạy và nhập bừa flag rồi break tại hàm `CallWindowProcA` , tìm ở các `Angs` đầu vào là chúng ta sẽ thấy địa chỉ `shellcode` được load vào các param:


    ![image](https://user-images.githubusercontent.com/57254763/166269450-57f15102-ba9e-43b4-a35b-d79488ea5c55.png)
 
  ## Vậy mục tiêu của bài này là chúng ta sẽ phải phân tích được đoạn shellcode này 
  - (Cho những ai chưa biêt) `shellcode là một đoạn mã nhỏ được sử dụng làm trọng tải trong việc khai thác lỗ hổng phần mềm` , và nó phải là độc lập , vì độc lập thì nó mới có thể chạy được ở bất cứ đâu, 
  Ví dụ về đoạn code độc lập và không độc lập : 
  ### Độc Lập : 
  - (Image from `Practical Malware Analysis` Book)


    ![image](https://user-images.githubusercontent.com/57254763/166270160-ee546b74-215d-46f3-9363-8c0a69245c20.png)
   - Ta có thể thấy đoạn code trên string "Hello world" Nằm tại shellcode luôn.
  ### Không độc lập: 
  ```assembly 
  call sub_401000  
  jnz short loc_401044  
  mov edx, aHelloWorld ;"Hello World" 
  mov eax, [ebp-4] 
  ```
  - Đoạn code trên String "Hello World" nằm ở 1 địa chỉ khác và thuộc vào 1 chương trình. 
  - Và shellcode muốn độc lập thì chúng cần phải lấy được hàm `LoadLibraryA` và `GetProcAddress` từ `kernel32.dll` , một khi lấy được hai function này thi shellcode có thể làm mọi thứ như một chương trình hoàn hảo.
  - Vậy chắc chắn shellcode muốn load các hàm khác vào thì cần phải gọi hàm `GetProcAddress` giống như : 

    ```C
     GetProcAddress(GetModuleHandle(TEXT("kernel32.dll")), "GetNativeSystemInfo");
    ```
  - Hàm `GetProcAddress` có sử dụng tên hàm là một chuỗi ("GetNativeSystemInfo") nên tận dụng điều này chúng ta khi phân tích shellcode sẽ tránh biến những string như trên trong shellcode thành `opcode` 
  
  # Phân tích shellcode 
  - Sử dụng chức năng `make code` và `undefined` của IDA để phân tích
  - Sau khi né tránh các string chúng ta được 1 đoạn shellcode dễ đọc hơn :
  
	
	![image](https://user-images.githubusercontent.com/57254763/166288647-286da9b0-3ea0-47c7-b33c-c8e6ba374fcf.png)
	
  - Còn rất nhiều bên dưới chúng ta cũng làm tương tự 
  - Công việc còn lại là đọc shellcode thật trâu :3 
  ## Tóm tắt shellcode : 
  - Load các hàm CryptoAPI 
  ```C
    declare aHttpsWwwYoutub = 'https://www.youtube.com/watch?v=dQw4w9WgXcQ'
	declare hProv;
	CryptAcquireContextA(&hProv,0,0,PROV_RSA_FULL,0)

	declare phHash;
	CryptCreateHash(hProv,CALG_SHA,0,0,&phHash)

	declare lenhttp  = strlen(aHttpsWwwYoutub)
	declare hash;
	CryptHashData(hHash,aHttpsWwwYoutub,lenhttp,0)

	declare phKey;
	CryptDeriveKey(hProv,CALG_RC4,hash,0,&phKey)
	CryptDestroyHash(hHash)
	
	CryptEncrypt(phKey,0,1,0,0,0x1C,0) // ????? 
	
	newspace = VirtualAlloc(0,0x1D,0x1000,4)

	memset(newspace,0,0x1D)
	newspace[i...end] = input[i..end]
	len2 = strlen(newspace)
	CryptEncrypt(phKey,0,1,0,newspace,len1,len2)
   ```
   - Shellcode sử dụng thuật toán `RC4` với `key` là cái link youtube kia để Encrypt và so sánh với số byte đã có sẵn.
   - Để tăng độ chính xác mình đã viết trực tiếp hàm decrypt bằng CryptoAPI như của tác giả đã sử dụng :
   ```C 
#pragma comment(lib, "crypt32.lib")

#include <stdio.h>
#include <tchar.h>
#include <windows.h>
#include <Wincrypt.h>
#include<string.h>
unsigned char cipher[] =
{
  0xF8, 0x50, 0xCC, 0xEF, 0xE6, 0x3C, 0x35, 0x96, 0x1D, 0x61,
  0xAE, 0xC0, 0xC5, 0x31, 0xCE, 0xB0, 0xE7, 0x1D, 0xED, 0xBC,
  0x5D, 0x81, 0x69, 0x8A, 0x35, 0x74, 0x57, 0xB6
};
int main(void)
{
    HCRYPTPROV hProv;
    if (CryptAcquireContextA(&hProv, 0, 0, PROV_RSA_FULL, 0)) {
        fprintf(stdout, "Success CryptAcquireContextA\n");
        
    };
    HCRYPTHASH phHash;

    if (CryptCreateHash(hProv, CALG_SHA, 0, 0, &phHash)) {
        fprintf(stdout, "Success CreateHash\n");
        
    };
 
    BYTE text[] = "https://www.youtube.com/watch?v=dQw4w9WgXcQ";

    if (CryptHashData(phHash, text,strlen((const char*)text), 0)) {
        fprintf(stdout, "Success CryptHashData\n PassWord : %s ,\n Length : %d\n",text, strlen((const char*)text));
    };
    HCRYPTKEY phKey;
    if (CryptDeriveKey(hProv, CALG_RC4, phHash, 0, &phKey)) {
        fprintf(stdout, "Success CryptDeriveKey\n");
    };
    CryptDestroyHash(phHash);
    DWORD len1 = 0x1c;
    DWORD len2 = 0x1c;
    //CryptEncrypt(phKey, 0, 1, 0, (BYTE*)newspace, (DWORD*)&len1, len2);
    CryptDecrypt(phKey, 0, 1, 0, (BYTE*)cipher, (DWORD*)&len1);
    printf("flag : %s \n", cipher);



    if(CryptReleaseContext(hProv, 0))
    {
        printf("The handle has been released.\n");
    }
    else
    {
        printf("The handle could not be released.\n");
    }
    return 1;
}

 

 ```
 
 ![image](https://user-images.githubusercontent.com/57254763/166293492-7b501155-4a40-49b6-8c41-122175021660.png)


flag :`KCSC{rC4_8uT_1T_L00k2_W31Rd}`
