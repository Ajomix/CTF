# Encryptor

- Chạy thử file .exe thì đây là một bài nhận đầu vào là file bất kì rồi encrypt nó 

    ![image](https://user-images.githubusercontent.com/57254763/182172293-69b3f532-c10e-4600-b612-1244ef482da6.png)
    
- Và tác giả đã cho chúng ta 1 file flag.bin , file này là file flag đã bị encrypt. 
- Cho file thực thi vào ida và tìm đến hàm encrypt.

    ![image](https://user-images.githubusercontent.com/57254763/182173662-e3f31a61-ef58-435b-a132-110539b4afb6.png)
    
- Đầu vào của hàm `ENCRYPT` là 1 vùng nhớ trống và buf (file chúng ta nhập lúc đầu).
- Đục sâu vào hàm `ENCRYPT`:

    ![image](https://user-images.githubusercontent.com/57254763/182174115-cfec8ff7-de92-44b4-bc39-dee30d5d789b.png)
 
- Chúng ta có thể thấy ngay ở đoạn đầu tác giả đã gọi ra hàm `srand` để chút nữa sẽ sử dụng vào việc encrypt file.
- Tiếp tục đọc phần bên dưới :
```C
......
  p_space = space;
  v4 = 0;
  seed = time64(0);
  srand(seed);
  *p_space++ = 0;
  *p_space = 0;
  p_space[1] = 0;
  *space = 0;
  space[1] = 0;
  space[2] = 0;
  v14 = 0;
  end_of_buf = buf[1];
  start_of_buf = *buf;
  v10 = end_of_buf;
  v12 = *buf;
  if ( *buf != end_of_buf )
  {
    shifter = -6;
    do
    {
      v4 = *start_of_buf + (v4 << 8);
      shifter += 8;
      if ( shifter >= 0 )
      {
        do
        {
          v13[0] = 2 * ((v4 >> shifter) & 0x3F) + rand() % 2;
          sub_7C1D2D(space, v13);
          shifter -= 6;
        }
        while ( shifter >= 0 );
        end_of_buf = v10;
      }
      start_of_buf = v12 + 1;
      v12 = start_of_buf;
    }
    while ( start_of_buf != end_of_buf );
    if ( shifter > -6 )
    {
      v13[0] = 2 * (((_BYTE)v4 << (shifter + 6)) & 0x3F) + rand() % 2;
      sub_7C1D2D(space, v13);
    }
  }
......

```
- Chúng ta có thể thấy 2 vòng lặp while lồng nhau, Phân tích vòng while đầu tiên :
```C
v12 = *buf
if ( *buf != end_of_buf )
  {
    shifter = -6;
    do
    {
      v4 = *start_of_buf + (v4 << 8);
      shifter += 8;
      if ( shifter >= 0 )
      {
        do{
            .... // code 
            shifter-=6;  
        }while(shifter >= 0)
        
      }
      start_of_buf = v12 + 1;
      v12 = start_of_buf;
    }
```

- Ở đây vòng while đầu tiên sẽ đặt `shifter = -6` , và thực hiện liên tục công việc phía sau là `shifter+=8` ,  nếu `shifter >= 0`nó sẽ thực hiện vòng `shifter-=6` cho đến khi `shifter < 0` thì thôi. Đoạn này nói thì khá là phức tạp nên chúng ta sẽ thử viết lại 1 đoạn y hệt như vậy trong code C rồi in shifter ra để xem kết quả xem nó có gì đặc biệt , Code của mình như sau :

```C

#include <stdio.h>
 
int main()
{
	int c = 10;
	int shifter;
	
	shifter = -6;
	do{
		shifter += 8;
		if(shifter >= 0){
			do{
				//Encrypted Formula
				printf("%d ",shifter);
				shifter-=6;
			}while(shifter>=0);
		}
		
	}while(c--);
	return 0;
}
```
- Kết quả :
   
   ![image](https://user-images.githubusercontent.com/57254763/182178170-c868b65d-dc18-496e-a3b4-b7d563e97311.png)

- Vậy qua đây chúng ta thấy được shifter sẽ chỉ là 0 2 4 hoặc 6 .
- Tiếp tục quay lại phân tích phần còn lại của vòng while đầu tiên : `v4 = *start_of_buf + (v4 << 8);`
- `v4` ban đầu được gán giá trị là 0 , vậy ở đây chúng sẽ shift lên 8 bit để nhét kí tự tiếp theo vào , Ví dụ như này :
Chúng ta nhập vào chữ `abcd` hay nói cách khác `start_of_buf[] = "abcd"` , khi sử dụng công thức bên trên , đầu tiên `v4` đang là `0` sẽ được shift lên 8 bit tức là sau lần đầu tiên v4 sẽ là `0x61` (chữ `a`) , sau lần thứ 2 sử dụng `v4` sẽ là `0x6162` (nhét thêm chữ `b`) , công việc cứ liên tục như vậy cho đến khi hết vòng while đầu tiên , và chúng ta phải nhớ rằng biến `v4` sẽ chỉ chứa được 4 kí tự , vì nó được định dạng là kiểu `int(4byte)` nên khi v4 bị shift đến mức đầy rồi thì nó sẽ tự bị strip đi , ví dụ như mình đã load đầy `v4=0x61626364` khi thực hiện tiếp công thức nhét kí tự bên trên (giả sử mình nhập thêm chữ `d`) thì `v4=0x62636465`. 

- Tiếp đến vòng lặp thứ 2 : 
```C
        do
        {
          v13[0] = 2 * ((v4 >> shifter) & 0x3F) + rand() % 2;
          sub_7C1D2D(space, v13);
          shifter -= 6;
        }
        while ( shifter >= 0 );
        
```
- Hàm  `sub_7C1D2D(space, v13);` sau khi debug mình thấy nó gán giá trị của `v13` vào `space` (cái vùng nhớ lúc truyền đầu vào).
- Ngay bây giờ chúng ta đã thấy `shifter` và `rand()` đã được sử dụng , shifter chỉ là các giá trị `0 2 4 6` , mà `v4` lại là giá trị được nhồi nhét `input` bên trong, 
, Và nó liên tục shift xuống theo giá trị `0 2 4 6` , sau đó nó lại được   `AND 0x3F` , phép `AND` ở đây là phép toán để mask bit , khi đổi 0x3f sang số hệ 2 thì mình thấy nó là `0b111111`, vậy kết luận đoạn ` ((v4 >> shifter) & 0x3F)`là nó sẽ shift xuống `0 2 4 6` bit , tiếp đến nó AND với 0x3f hay nói cách khác đoạn AND này là nó sẽ lấy 6 bit của cái đống đã shift xuống kia thôi. Tiếp đến nó cộng với `rand()%2` , đoạn `rand()%2` này nói cách khác nó sẽ chỉ là 0 hoặc 1. Nghe đến đây thuật toán encrypt có vẻ khá là phức tạp , nhưng đến đây công việc rất đơn giản là các bạn chỉ cần brute force thôi , tại sao mình lại nghĩ đến cách này vì khi đọc đến đoạn `rand()%2` mình thấy không có cách nào có thể lấy lại được giá trị `rand%2` này để reverse lại kết quả được. 
- Tổng kết thuật toán Encrypt như sau : (Mình sẽ thực hiện lại từng bước 1 của thuật toán encrypt này)
  + Giả sử như mình đang có `v4 = 0x61626364`  
  + Đổi nó sang bit thì `v4 = 01100001011000100110001101100100`
  + Đầu tiên chúng ta có `shifter = 2`.
  + Vậy  `(v4 >> shifter) & 0x3f  + rand()%2 = 0b011001 + 0 hoặc 1`
  
  	![image](https://user-images.githubusercontent.com/57254763/182188021-101521c9-455a-4929-8bb9-9a5927b18196.png)
										 	 
  + Tiếp theo `shifter = 4`:
 
  	![image](https://user-images.githubusercontent.com/57254763/182188302-60ae1413-98d6-4947-bc63-50b0381ebc05.png)	
	
  + Tiếp theo `shifter = 6`:
  
  	![image](https://user-images.githubusercontent.com/57254763/182188688-cd5550cb-7027-4eac-9fcc-63f7b5f38a9f.png)	
 ..........
 TÓM LẠI LÀ NÓ SẼ LẤY TỪNG 6 BIT RA MỘT ĐỂ NÓ ENCRYPT , VẬY NÊN CHÚNG TA SẼ BRUTEFORCE............


- Vậy bruteforce như thế nào?
- Sau những công việc phân tích trên , các bạn có để ý đến đoạn mình nói phép `AND` **CHỈ LẤY 6 BIT CỦA ĐỐNG SHIFTER** , Vậy tại sao chúng ta không thử hết giá trị từ 0 cho đến max của 6 bit đó ? , và khi cộng với phép `rand()%2` chúng ta lại tiếp tục thử vì giá trị của nó chỉ có thể là 0 hoặc 1 thôi. 
- Tóm lại công thức brute ở đây sẽ là `2*j + m` với `j` chạy từ 0 cho đế kịch kim 6 bit ( kịch kim 6 bit sẽ là từ `0 cho đến 0x3f` ), và `m` sẽ là 0 hoặc 1 , nếu thử encrypt xong mà nó bằng cipher đã encrypted thì chúng ta sẽ lấy giá trị `j` đó, dưới đây là script viết bằng go của mình.


```go

package main

import (
    "bufio"
    "fmt"
    "log"
    "math/big"
    "os"
)


func main() {
    f, err := os.Open("flag.bin")
    if err != nil {
        fmt.Println("Failed", err)
    }
    defer f.Close()
    stat, stat_err := f.Stat()
    if stat_err != nil {
        fmt.Println(stat_err)
    }
    size := stat.Size()
    b := make([]byte, size+1)
    bufio.NewReader(f).Read(b)

    flag := big.NewInt(0)
    for i := 0; i < int(size); i++ {
        for j := 0; j <= 0x3f; j++ {
            for m := 0; m <= 1; m++ {
                if ((2 * j) + m) == int(b[i]) {
                    flag.Lsh(flag, 6)
                    flag.Add(flag, big.NewInt(int64(j)))
                    m = 2
                    j = 0x3f + 1
                }
            }
        }
    }
    f2, _ := os.OpenFile(
        "flag.jpg",
        os.O_WRONLY|os.O_TRUNC|os.O_CREATE,
        0666,
    )
    defer f2.Close()
    bytesWritten, err := f2.Write(flag.Bytes())
    if err != nil {
        log.Fatal(err)
    }
    log.Printf("Wrote %d bytes.\n", bytesWritten)
    
}
```
