# Simple_BMP
- DỮ LIỆU ĐỀ BÀI CHO BAO GỒM :
 + 1 file Binary 
 + 2 file bit map (.BMP) 
- Chạy thử chương trình : 

  ![image](https://user-images.githubusercontent.com/57254763/168132128-895b6745-223c-43ed-a6b1-9ac35171b11d.png)
  
- Khi load file Binary vào IDA và Decompile về code C chúng ta có :
```C
if ( argc >= 2 )
  {
    v4 = time(0i64);
    srand(v4);
    v5 = fopen(argv[1], "rb");
    v6 = v5;
    if ( !v5 )
    {
      puts("Cannot open file");
      exit(1);
    }
   ...... 
   .....
   .....
  else
  {
    printf("Usage: %s <filename>\n", *argv);
  }
```
- Chương trình sẽ check xem tham số đầu vào là bao nhiêu tham số 
- Tiếp đến là tạo random bất kì 1 số 
- Đọc file
`Để thuận tiện hơn chúng ta sẽ đổi tên v5 thành stream  :` 
```C
    v4 = time(0i64);
    srand(v4);
    stream = fopen(argv[1], "rb");
    v6 = stream;
    if ( !stream )
    {
      puts("Cannot open file");
      exit(1);
    }

```
 

```C
    ....
    ....
    fseek(stream, 0, 2);
    v7 = (int)common_ftell<long>(v6);//lấy size của file chúng ta đưa vào (`v7`)
    rewind(v6);
    v8 = (unsigned __int8 *)operator new((int)v7 + 1); //Tạo một vùng nhớ với size là size của file
    fread(v8, 1ui64, v7, v6); //Đọc file rồi lưu vào vùng nhớ vừa tạo (`lưu vào v8`)
    fclose(v6);
    v9 = sqrt((double)(int)v7) ;//Lấy Căn bậc hai của size và đưa vào biến `v9`
    dword_140023CAC = (int)ceilf(v9); // Sử dụng `Ceilf`để làm tròn số vừa căn bậc hai (`v9`) rồi lưu vào Memory
    dword_140023CA8 = dword_140023CAC;
    v10 = operator new(saturated_mul((unsigned int)dword_140023CAC, 8ui64));
    LODWORD(v12) = dword_140023CA8;
    v13 = 0i64;
    qword_140023CB0 = (__int64)v10;
    v14 = (__int64)v10;
    ....
    ...
```
- `Ở đây mình sẽ đổi lại tên cho dễ phân tích và dễ đọc hơn như sau`:
```C
    
    fseek(stream, 0, 2);
    size = (int)common_ftell<long>(v6);
    rewind(v6);
    new_space_1 = (unsigned __int8 *)operator new((int)size + 1);
    fread(new_space_1, 1ui64, size, v6);
    fclose(v6);
    size_sqrt = sqrt((double)(int)size);
    ceilf_sqrt = (int)ceilf(size_sqrt);
    ceilf_sqrt_coppy = ceilf_sqrt;
    new_space_2 = operator new(saturated_mul((unsigned int)ceilf_sqrt, 8ui64));
    LODWORD(tmp) = ceilf_sqrt_coppy;
    v13 = 0i64;
    mem_space = (__int64)new_space_2;
    v14 = (__int64)new_space_2;
    
```
```C
  if ( ceilf_sqrt_coppy )
    {
      do
      {
        v15 = operator new(saturated_mul((unsigned int)ceilf_sqrt, 3ui64));// set up thêm một vùng nhớ nữa bên trong vùng nhớ vừa được tạo bên trên (`mem_space`) , cái này có thể hiểu là đang tạo ra 1 mảng 2 chiều 
        v14 = mem_space;
        tmp = (unsigned int)ceilf_sqrt_coppy;
        *(_QWORD *)(mem_space + 8 * v13++) = v15;
      }
      while ( v13 < tmp );
    }
    v16 = (unsigned int)ceilf_sqrt;
    v17 = (unsigned int)tmp;
```
- `After: `
```C
 if ( ceilf_sqrt_coppy )
    {
      do
      {
        tmp2 = operator new(saturated_mul((unsigned int)ceilf_sqrt, 3ui64));
        tmp3 = mem_space;
        tmp = (unsigned int)ceilf_sqrt_coppy;
        *(_QWORD *)(mem_space + 8 * i++) = tmp2;
      }
      while ( i < tmp );
    }
    v16 = (unsigned int)ceilf_sqrt;
    v17 = (unsigned int)tmp;
```
- Tiếp tục sau đó toàn là những bước setup khởi tạo chuẩn bị cho công cuộc encrypt byte từ file của chúng ta 
- Đọc kĩ đến đoạn sau , chương trình đã encrypt từng byte của file như sau :
```C
....
...
              v24 = new_space_1[tmp3];
              v25 = rand() % 256;
              LODWORD(v16) = ceilf_sqrt;
              v26 = 0x6969 * (v24 + (v25 << 8));
              tmp3 = (unsigned int)(v26 / 0xFCD74B);
              v26 %= 0xFCD74B;
              v28 = v26;
              v11 = (unsigned int)(v26 >> 16);
              tmp = *(_QWORD *)(v23 + mem_space);
              *(_WORD *)(tmp + v21) = v28;
              *(_BYTE *)(tmp + v21 + 2) = v11;
              LODWORD(tmp) = ceilf_sqrt_coppy;
....
...
```
### Đổi lại các tên biến để dễ phân tích hơn : 
```C
              value = buf_file[tmp3];
              rand_number = rand() % 256;
              LODWORD(v16) = ceilf_sqrt;
              tmp4 = 0x6969 * (value + (rand_number << 8));
              tmp3 = (unsigned int)(tmp4 / 0xFCD74B);
              tmp4 %= 0xFCD74B;
              tmp5 = tmp4;
              tmp6 = (unsigned int)(tmp4 >> 16);
              tmp = *(_QWORD *)(p + mem_space);
              *(_WORD *)(tmp + j) = tmp5;
              *(_BYTE *)(tmp + j + 2) = tmp6;
              LODWORD(tmp) = ceilf_sqrt_coppy;
```

- Giải thích : 
- Đoạn code trên lấy ra từng byte của file chúng ta truyền vào 
- Tạo random byte 
- Trộn byte của file và byte random để tạo ra 3 byte bất kì để chút nữa sẽ save vào file mới 
-  Đọc kĩ một chút là chúng ta có thể tóm gọn được công thức encrypt của đoạn code trên : 
  
  ## `mem_space[i] (3 byte) = 0x6969 * (value +(rand_number << 8 ) % 0xFCD74B)  (3 byte)`
 - Hay chúng ta có thể viết tổng quát thành :
  ## `c = b * a mod p`
 - Theo như công thức tổng quát thì chúng ta đã có
 
 `b = 0x6969`
 
 `p = 0xFCD74B`
 
 `c là cipher chúng ta đã có`
 - Và `a` là ẩn số cần tìm , nếu tìm được `a` chúng ta sẽ lấy byte đầu tiên của `a` là được
 - Biến đổi một chút sẽ được công thức giải được công thức mã hóa trên :
 ## `a = c * pow(b,-1,p) mod p`
 # script giải bài các bạn có thể tham khảo : 
 ```python
 
 from PIL import Image

def getFlag(flag,filename):
    #a*b mod p = c
    #b = c*pow(a,-1,p)%p
    im = Image.open(flag)
    px = im.getdata()
    a=0x6969
    p=0xFCD74B
    buf = []
    for i in range(len(px)):
            c = (px[i][2]<<16)|(px[i][1]<<8)|px[i][0]
            b = c*pow(a,-1,p)%p
            b=b&0xff
            buf.append(b)
    file = open(filename,"wb")
    file.write(bytes(buf))    

getFlag("flag1.jpg.bmp","flag1.jpg")
getFlag("flag2.jpg.bmp","flag2.jpg")

 ```
