##### 原码

正数的原码不变。负数的原码符号位为1

##### 反码

正数的反码不变。负数的反码为，在原码的基础上，符号位不变，其余位取反

##### 补码

正数的补码不变，负数的补码为，在反码的基础上，加1

`在原码符号位不变，其他的从低位开始，直到遇见第一个1之前，什么都不变；遇见第一个1后保留这个1，以后按位取反`

示例

```

// 使用1个字节表示
// X = 43
// Y = -43
原码
X = 00101011
Y = 10101011
反码
X = 00101011
Y = 11010100
补码
X = 00101011
Y = 11010101

```

