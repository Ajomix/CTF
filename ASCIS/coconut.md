# Coconut
Như thường lệ mình ném file vào CFF Explorer

![image](https://user-images.githubusercontent.com/57254763/196045292-35f61fe2-93cc-4cb1-b6dc-d384692b08d4.png)

Đây là file EXE được viết bằng .Net nên mình sẽ sử dụng Dnspy để decompile chương trình.
Khi lướt qua các hàm một lúc thì mình thấy khá nhiều những đoạn code bị lỗi mà Dnspy ko dịch được :

![image](https://user-images.githubusercontent.com/57254763/196045910-84c5089f-f5cc-4298-bbbd-35549e25fd7a.png)

Main:

![image](https://user-images.githubusercontent.com/57254763/196045944-b30bcadb-0a52-445c-ad4a-7d3e0e189e67.png)

Mình thử debug để hiểu rõ hơn luồng chương trình , hàm `Coconut_10` đầu tiên ở hàm `Main` set up rất nhiều mảng những byte khác nhau :

![image](https://user-images.githubusercontent.com/57254763/196046092-c4118c9e-7dfb-41c4-a313-26f2724781df.png)

Tiếp sau đó hàm `coconut_28`:
```c#
// Token: 0x06000004 RID: 4 RVA: 0x00002910 File Offset: 0x00000B10
		public static string coconut_28()
		{
			string result;
			try
			{
				result = Coconut.coconut_82();
			}
			catch (InvalidProgramException e)
			{
				result = (string)Coconut.coconut_25(e, new object[0], Coconut.meat_28, Coconut.water_28);
			}
			return result;
		}
```

`coconut_82` bị lỗi:

![image](https://user-images.githubusercontent.com/57254763/196046217-604b6b34-8d0f-40a1-b0af-c14c97200d06.png)

- Tóm lại là chương trình set một `try,catch Exception` cho `coconut_82`, khi chạy vào `coconut_82` bị lỗi thì sẽ nhảy vào `coconut_25` , chúng ta sẽ tiếp tục trace vào `coconut_25`:
- `coconut_25` nhận vào 3 đầu vào là những data đã được set up ở `coconut_10` ban đầu :
```c#
...
result = (string)Coconut.coconut_25(e, new object[0], Coconut.meat_28, Coconut.water_28);
```
- `Coconut25` resolve Metadata Token từ Exception
```c#

// Token: 0x06000003 RID: 3 RVA: 0x000025E4 File Offset: 0x000007E4
		public static object coconut_25(InvalidProgramException e, object[] args, Dictionary<uint, int> m, byte[] b)
		{
			int metadataToken = new StackTrace(e).GetFrame(0).GetMethod().MetadataToken; // Get Last Error
			Module module = typeof(Program).Module;
			MethodInfo methodInfo = (MethodInfo)module.ResolveMethod(metadataToken); // Resolve
			MethodBase methodBase = module.ResolveMethod(metadataToken);
			ParameterInfo[] parameters = methodInfo.GetParameters();
			Type[] array = new Type[parameters.Length];
			SignatureHelper localVarSigHelper = SignatureHelper.GetLocalVarSigHelper();
			for (int i = 0; i < array.Length; i++)
			{
				array[i] = parameters[i].ParameterType;
			}
			Type declaringType = methodBase.DeclaringType;
			DynamicMethod dynamicMethod = new DynamicMethod("", methodInfo.ReturnType, array, declaringType, true);
			DynamicILInfo dynamicILInfo = dynamicMethod.GetDynamicILInfo();
			MethodBody methodBody = methodInfo.GetMethodBody();
			foreach (LocalVariableInfo localVariableInfo in methodBody.LocalVariables)
			{
				localVarSigHelper.AddArgument(localVariableInfo.LocalType);
			}
			byte[] signature = localVarSigHelper.GetSignature();
			dynamicILInfo.SetLocalSignature(signature);
			foreach (KeyValuePair<uint, int> keyValuePair in m)
			{
				int value = keyValuePair.Value;
				uint key = keyValuePair.Key;
				int tokenFor;
				if (value >= 1879048192 && value < 1879113727)
				{
					tokenFor = dynamicILInfo.GetTokenFor(module.ResolveString(value));
				}
				else
				{
					MemberInfo memberInfo = declaringType.Module.ResolveMember(value, null, null);
					if (memberInfo.GetType().Name == "RtFieldInfo")
					{
						tokenFor = dynamicILInfo.GetTokenFor(((FieldInfo)memberInfo).FieldHandle, ((TypeInfo)((FieldInfo)memberInfo).DeclaringType).TypeHandle);
					}
					else if (memberInfo.GetType().Name == "RuntimeType")
					{
						tokenFor = dynamicILInfo.GetTokenFor(((TypeInfo)memberInfo).TypeHandle);
					}
					else if (memberInfo.Name == ".ctor" || memberInfo.Name == ".cctor")
					{
						tokenFor = dynamicILInfo.GetTokenFor(((ConstructorInfo)memberInfo).MethodHandle, ((TypeInfo)((ConstructorInfo)memberInfo).DeclaringType).TypeHandle);
					}
					else
					{
						tokenFor = dynamicILInfo.GetTokenFor(((MethodInfo)memberInfo).MethodHandle, ((TypeInfo)((MethodInfo)memberInfo).DeclaringType).TypeHandle);
					}
				}
				b[(int)key] = (byte)tokenFor;
				b[(int)(key + 1U)] = (byte)(tokenFor >> 8);
				b[(int)(key + 2U)] = (byte)(tokenFor >> 16);
				b[(int)(key + 3U)] = (byte)(tokenFor >> 24);
			}
			dynamicILInfo.SetCode(b, methodBody.MaxStackSize);
			return dynamicMethod.Invoke(null, args);
		}
```
- Ở dòng cuối cùng mình có thấy dòng `SetCode` và dòng `Invoke` dynamic IL Code, chúng ta có thể hiểu đoạn bên trên là chương trình sẽ sửa những đoạn code bị lỗi ở hàm trước đó thành các IL code đúng , sau đó thì invoke để chạy chương trình , nhưng có một vấn đề là sau khi sửa code và invoke thì chúng ta lại không debug được chương trình sẽ làm gì.
- Vậy đến đây chắc chắn chúng ta sẽ phải viết một `patcher` để patch lại những đoạn code đã bị lỗi kia . Và patch bằng IL code chuẩn thì sẽ là những phần tử con của biến `b` kia .
- Nhưng có một vấn đề các bạn cần để ý đó là `dynamic token` và `method header` của mỗi hàm. Có nghĩa là mỗi hàm trong C# đều có một `header` (khoảng 1,2 hoặc 10 byte đầu tùy chương trình), ví dụ một hàm:
![](https://i.imgur.com/21VoQjr.png)

- Chúng ta có thể thấy `header size` sẽ là 1 , vậy thì lúc patch chúng ta không được phép patch vào `header` của method.
- Vấn đề tiếp theo là `dynamic token`,chúng ta có thể thấy đoạn bên trên `setCode` chương trình đang thực hiện các kĩ thuật để resolve ra đc dynamic token cho chương trình , nhưng lúc chúng ta patch chương trình thì chúng ta không cần những bước đó , vì những bước đó là chỉ resolve ra token lúc  `dynamic` , còn lúc patch chúng ta sẽ hiểu là chúng ta đang chạy chương trình theo kiểu `static`.
- Đây là code để mình patch: 
```python

water_01=[1,14,10,15,15,12,12,5,0,1,2,9,4,2,4,8,7,3,9,12,9,7,6,5,6,6,15,1,5,4,7,7,15,8,11,0,0,14,13,8,11,6,11,15,9,10,11,12,14,2,3,6,7,0,1,13,13,15,6,13,1,0,10,12]
water_02=[0,13,1,5,3,10,14,3,12,3,9,10,0,14,2,12,7,8,2,1,13,14,13,14,4,6,5,13,7,12,3,1,9,2,0,8,2,14,6,12,0,0,0,2,4,3,7,5,15,10,15,2,14,10,11,11,0,7,10,2,10,0,12,10,1]
water_03=[0,11,11,3,13,13,8,6,7,12,2,4,8,12,12,11,1,12,1,2,2,12,5,11,0,14,5,11,13,0,14,2,13,0,15,11,6,0,12,11,8,9,15,6,8,11,10,5,2,14,15,14,8,0,13,13,7,11,4,1,1,4,15,8,0]
meat_28=[[1,1879048193],[6,167772164],[11,167772165]]
water_28=[114,0,0,0,0,40,0,0,0,0,40,0,0,0,0,42]
meat_15=[[1,167772220],[7,167772211],[12,167772221],[17,1879048389],[22,1879048217],[27,167772222]]
water_15=[40,0,0,0,0,2,111,0,0,0,0,40,0,0,0,0,114,0,0,0,0,114,0,0,0,0,111,0,0,0,0,42]
meat_25=[[7,167772223],[12,67108874],[17,100663318],[22,167772224],[27,67108875],[32,100663318],[37,167772225],[42,67108876],[47,100663318],[52,167772226]]
water_25=[2,32,3,2,0,0,40,0,0,0,0,126,0,0,0,0,40,0,0,0,0,40,0,0,0,0,126,0,0,0,0,40,0,0,0,0,40,0,0,0,0,126,0,0,0,0,40,0,0,0,0,40,0,0,0,0,57,2,0,0,0,23,42,22,42]
meat_89=[[1,1879048217],[38,167772238],[43,167772215],[64,167772238],[69,167772215],[91,167772223]]
water_89=[114,0,0,0,0,10,2,11,22,12,56,63,0,0,0,7,8,145,13,9,31,10,60,26,0,0,0,6,9,31,48,88,209,19,4,18,4,40,0,0,0,0,40,0,0,0,0,10,56,21,0,0,0,6,9,31,87,88,209,19,4,18,4,40,0,0,0,0,40,0,0,0,0,10,8,23,88,12,8,7,142,105,50,187,6,32,3,2,0,0,40,0,0,0,0,42]
meat_46=[[1,100663326],[7,100663314]]
water_46=[40,0,0,0,0,2,40,0,0,0,0,42]
meat_45=[[1,167772210],[7,167772211],[13,167772210],[18,1879048393],[23,167772211],[74,16777218],[80,167772227],[88,167772228],[95,167772229],[102,167772230],[108,167772231],[113,167772232],[121,167772233],[133,167772234],[142,167772235],[151,167772236],[161,167772237],[181,167772187],[196,167772187],[211,167772187],[224,167772187],[230,1879048427],[236,167772212],[241,100663308]]
water_45=[40,0,0,0,0,3,111,0,0,0,0,10,40,0,0,0,0,114,0,0,0,0,111,0,0,0,0,11,2,57,7,0,0,0,2,142,58,1,0,0,0,42,6,57,7,0,0,0,6,142,58,1,0,0,0,42,7,57,7,0,0,0,7,142,58,1,0,0,0,42,2,142,105,141,0,0,0,0,12,115,0,0,0,0,13,9,6,111,0,0,0,0,9,7,111,0,0,0,0,9,9,111,0,0,0,0,9,111,0,0,0,0,111,0,0,0,0,19,4,2,115,0,0,0,0,19,5,17,5,17,4,22,115,0,0,0,0,19,6,17,6,115,0,0,0,0,19,7,17,7,111,0,0,0,0,8,22,2,142,105,111,0,0,0,0,38,221,58,0,0,0,17,7,57,7,0,0,0,17,7,111,0,0,0,0,220,17,6,57,7,0,0,0,17,6,111,0,0,0,0,220,17,5,57,7,0,0,0,17,5,111,0,0,0,0,220,9,57,6,0,0,0,9,111,0,0,0,0,220,114,0,0,0,0,8,40,0,0,0,0,40,0,0,0,0,42]
meat_06=[[1,167772216],[9,167772217],[24,1879048331],[29,167772218],[39,100663304],[50,167772219]]
water_06=[40,0,0,0,0,10,18,0,40,0,0,0,0,32,206,11,0,0,60,15,0,0,0,114,0,0,0,0,40,0,0,0,0,56,6,0,0,0,40,0,0,0,0,42,32,0,92,38,5,40,0,0,0,0,43,200]
meat_36=[[1,100663325],[7,100663306],[13,167772210],[19,167772211],[25,100663302],[31,1879048293],[37,167772212]]
water_36=[40,0,0,0,0,10,40,0,0,0,0,11,40,0,0,0,0,7,111,0,0,0,0,6,40,0,0,0,0,12,114,0,0,0,0,8,40,0,0,0,0,42]
meat_16=[[1,167772213],[9,167772167],[14,167772168],[19,167772214],[26,167772167],[31,167772168],[36,167772214],[41,167772215]]
water_16=[115,0,0,0,0,10,6,24,111,0,0,0,0,111,0,0,0,0,111,0,0,0,0,6,23,111,0,0,0,0,111,0,0,0,0,111,0,0,0,0,40,0,0,0,0,42]
meat_26=[[6,16777250],[17,16777250],[26,16777218]]
water_26=[32,0,1,0,0,141,0,0,0,0,10,32,0,1,0,0,141,0,0,0,0,11,3,142,105,141,0,0,0,0,12,22,13,56,18,0,0,0,6,9,2,9,2,142,105,93,145,158,7,9,9,158,9,23,88,13,9,32,0,1,0,0,50,230,22,37,19,4,13,56,40,0,0,0,17,4,7,9,148,88,6,9,148,88,32,0,1,0,0,93,19,4,7,9,148,19,5,7,9,7,17,4,148,158,7,17,4,17,5,158,9,23,88,13,9,32,0,1,0,0,50,208,22,37,13,37,19,6,19,4,56,88,0,0,0,17,6,23,88,19,6,17,6,32,0,1,0,0,93,19,6,17,4,7,17,6,148,88,19,4,17,4,32,0,1,0,0,93,19,4,7,17,6,148,19,7,7,17,6,7,17,4,148,158,7,17,4,17,7,158,7,7,17,6,148,7,17,4,148,88,32,0,1,0,0,93,148,19,8,8,9,3,9,145,17,8,97,210,156,9,23,88,13,9,3,142,105,50,162,8,42]


#Coconut form : (m,b,offset,header_size)
list_coconut = [(meat_28,water_28,0x000007D0,1),(meat_15,water_15,0x00000E40,1),(meat_25,water_25,0x00000EB4,12),(meat_46,water_46,0x000010DC,1),(meat_89, water_89,0x00001134,12),(meat_45,water_45,0x00000F58,12),(meat_06,water_06,0x00000DB4,12),(meat_36,water_36,0x00000CA8,12),(meat_16,water_16,0x00000D28,12),(meat_26, water_26,0x00000B5C,12)]

def magic(coconut):
    m,b,offset,_ = coconut
    for key,value in m:
        b[key] = value&0xff
        b[key+1] = (value>>8)&0xff
        b[key+2] = (value>>16)&0xff
        b[key+3] = (value>>24)&0xff
    return b

def patch(file_sample,b,offset,h):
    for i in range(len(b)):
        file_sample[offset+h+i] = b[i]
    return file_sample

f = open("Coconut.exe","rb")
bin = f.read()
bin = list(bin)

for i in range(len(list_coconut)):
    t = list_coconut[i]
    bin = patch(bin,magic(t),t[2],t[3])

open("patch.exe","wb").write(bytes(bin))
f.close()
```

- Sau khi patch xong chương trình đã trở nên dễ đọc hơn rất nhiều, đây là khi chưa patch : 

![](https://i.imgur.com/z3di4X5.png)

- Sau khi patch : 

![](https://i.imgur.com/vS8eudg.png)

- Các bước tiếp theo chúng ta chỉ cần reverse để lấy key là được , đọc kĩ tiếp chúng ta sẽ thấy đoạn check key chúng ta cần phải nhập vào một key sao cho nó thỏa mãn phương trình :
```python

((key*13880328245145378785705410836767432968611776023726110634003371836796654391468)%94681236187809407884249877636270988187726197118196720312520904250574148078753)
== 84691773930568242145085772534897353407834382467231062104573594077086839033728
```
- Mình ném cho Crypto bên mình giải và được key là: 
`Ytd_is_history_Tmr_is_a_mystery!`
- Khi nhập vào : 

![](https://i.imgur.com/97Ss8T6.png)

- Chương trình đã generate ra nửa flag : 

![](https://i.imgur.com/6GKnhA7.png)

- Mình lập tức thử tìm đến đoạn code in ra dòng `Waiting for a thousand year.` kia : 

![](https://i.imgur.com/v0tkJNJ.png)

- Mình thử cho chương trình bỏ qua đoạn `sleep` : 

![](https://i.imgur.com/Md5DshX.png)

- Sau đó phần flag còn lại đã được generate: 

![](https://i.imgur.com/EnAXLK4.png)


FLAG:`ASCIS{7hat's_Why_7h3y_call_it_Prrrres3nt!!!!}` 
