# NikoGoKill
- Cài đặt file vào LDPLayer và chạy thử :
  ![image](https://user-images.githubusercontent.com/57254763/176722032-c29cae1c-3467-47c4-a387-0dbc6b9cc83d.png)
- Sử dụng JADX để Dịch ngược và thử tìm kiếm đoạn lời chào `"NIKO don't buy DE at NUKE pls "`

  ![image](https://user-images.githubusercontent.com/57254763/176722792-aa59d7b7-0d9d-49a4-8754-f0aa4a5dbfde.png)

- Đọc kĩ một chút thì bên trên hàm sẽ so sánh nếu `m3482Greeting$lambda1(mutableState)` bằng với biến `stringPlus` nào đó ở bên trên thì sẽ gán lời chào (`là biến str`) thành flag , nên ý tưởng bài này của mình sẽ là patch lại file apk này
## Mục tiêu :
- Vì ở đây hàm `decodeFlag` nhận vào hai tham số , đó là `numArr` và `m3482Greeting$lambda1(mutableState)` 

  ![image](https://user-images.githubusercontent.com/57254763/176723797-fa185493-435a-4c84-8b36-bb803cc7565e.png)

- Vậy để decode được flag chúng ta cần phải cho chương trình chạy được vào đoạn if decode flag :
  ![image](https://user-images.githubusercontent.com/57254763/176724219-aa1ca3ed-eefe-4662-89ff-f3f1a90dbfc0.png)
  Hay nói cách khác chúng ta sẽ chuyển chúng thành :
   ```java 
        if (TRUE) {
                try {
                    str2 = decodeFlag(numArr, m3482Greeting$lambda1(mutableState));
                } catch (Exception unused) {
                }
                str = str2;
            } else {
                str = "NIKO don't buy DE at NUKE pls ";
            }
   ```
 - Ở JADX chúng ta nhấn nút `TAB` để chuyển sang tab `byte-code` và tìm đến đoạn so sánh trên :
    ![image](https://user-images.githubusercontent.com/57254763/176726668-407003d7-8379-4ad8-a7fe-ed04ef6ec395.png)
 - `v4` và `v5` ở đây lần lượt sẽ là `m3482Greeting$lambda1(mutableState)` và `stringPlus`, nếu như đoạn `..areEqual` kia chúng ta sửa thành `Intrinsics.areEqual(stringPlus, stringPlus)` thì chúng sẽ luôn trả về `TRUE`, Hay nói cách khác nó sẽ thành :
 ```java
 invoke-static {v5,v5}, Lkotlin/jvm/internal/Intrinsics;->areEqual(Ljava/lang/Object;Ljava/lang/Object;)Z 
 ```
 - Đến đây chúng ta lại tiếp tục patch hàm `decodeFlag`:
   ```java
    str2 = decodeFlag(numArr, m3482Greeting$lambda1(mutableState));
   ```
 - Ở đây như flow chương trình chúng ta đã sửa ở bên trên ,`m3482Greeting$lambda1(mutableState)` sẽ phải bằng với `stringPlus` , thử chuyển sang tab byte-code để xem xét sửa đoạn `decodeFlag`:
  ![image](https://user-images.githubusercontent.com/57254763/176727819-6dd41a56-a2ee-47d2-8845-83d22e125dc0.png)
 - `v3` và `v4` ở đây lần lượt là `numArr` và `m3482Greeting$lambda1(mutableState)`, chúng ta sẽ sửa `v4` thành `v5(stringPlus)`:
   ```java
   invoke-static {v3, v5}, Lcom/example/nikogokill/MainActivityKt;->decodeFlag([Ljava/lang/Integer;Ljava/lang/String;)Ljava/lang/String;
   ```
 - Ok! Để patch được file trước tiên chúng ta cần decompile file apk , Mình sẽ sử dụng tool `APK Easy tool` để decompile cũng như build lại file
  ([DOWNLOAD](https://forum.xda-developers.com/t/discontinued-windows-apk-easy-tool-v1-60-2022-06-23.3333960/))
 - Load file và click vào Decompile:
   
  ![image](https://user-images.githubusercontent.com/57254763/176725668-7d048dd1-a65d-4330-b00b-434b57de4b20.png)
 
 - Vào đúng thư mục để sửa :
 
 ![image](https://user-images.githubusercontent.com/57254763/176728748-2ae7ee7a-08ba-4e8e-9f52-0729827c15ab.png)
 
 - SAVE và build lại file apk + cài đặt :
 
 ![image](https://user-images.githubusercontent.com/57254763/176729234-05f43742-0e79-45c8-bf03-ca5f1f23c0e4.png)
