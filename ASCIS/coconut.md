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
- 
```c#

// Token: 0x06000003 RID: 3 RVA: 0x000025E4 File Offset: 0x000007E4
		public static object coconut_25(InvalidProgramException e, object[] args, Dictionary<uint, int> m, byte[] b)
		{
			int metadataToken = new StackTrace(e).GetFrame(0).GetMethod().MetadataToken;
			Module module = typeof(Program).Module;
			MethodInfo methodInfo = (MethodInfo)module.ResolveMethod(metadataToken);
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

