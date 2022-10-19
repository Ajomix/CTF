# Dummy

- Đây là một bài reverse bootloader : 

![](https://i.imgur.com/kOrSQU4.png)

- Mình đã tham khảo một số trang github hay wu về những bài liên quan đến bootloader này : https://github.com/VoidHack/write-ups/blob/master/Square%20CTF%202017/reverse/floppy/README.md

- Khi mình làm theo và thử chạy với `qemu` thì chương trình báo cho mình như sau : 

![](https://i.imgur.com/eL7irKG.png)

- Đến đây mình thấy có vẻ là đã có đoạn nào đó check được trình giả lập mình đang dùng. Mình quyết định thử tìm đến offset đoạn string trên xem nó được dùng ở đâu , mình sử dụng lệnh : 
`strings -a -t x DummyOS.bin | grep HEY`

![](https://i.imgur.com/gRPRtl7.png)

- Ở IDA mình thử nhảy đến offset đó : 

 ![](https://i.imgur.com/RVPOKR9.png)

- Vậy là ở bên trên có lệnh `cpuid` lệnh này sẽ check xem chúng ta có đang dùng máy ảo hay không , nếu có thì sẽ in ra dòng `HEY...` kia và thoát chương trình , còn không thì sẽ chạy tiếp , đến đây mình thử patch đoạn sẽ thoát chương trình : 
 ![](https://i.imgur.com/DXrx7dT.png)

- Apply patch rồi chạy thử thì mình đã thành công : 

 ![](https://i.imgur.com/nu0RNvX.png)

- Các bước còn lại mình làm theo như wu như link mình đã gửi , debug tiếp thì mình tìm thấy được đoạn check flag như sau: 
![](https://i.imgur.com/mu0Wypi.png)

- Mình debug và lấy ra các giá trị để xor:

```python
d = [1,2,3,5,8 ,0xd ,0x15 ,0x22 ,0x37 ,0x59 ,0x90] 
d1 =[48, 52, 115, 105, 125, 126, 38, 16, 10, 109,0xa8]

for i in range(11):
	print(chr(d[i] ^ d1[i]),end = "")
```
`Password: 16plus32=48`

- Nhập vào password chúng ta có flag `ASCIS{dO_You_w@N7_tO_Bu1LD_@N_0S!!!}`
