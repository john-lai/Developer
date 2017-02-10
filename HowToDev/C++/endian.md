# 字节存放顺序：大尾，小尾 
字节存放有大尾和小尾之分。如果对应数据的高字节存放在低地址就是大尾，反之，高字节存放在高地址的就是小尾。

## 1/ 什么是大端模式，什么是小端模式？

所谓的大端模式（Big-endian），是指数据的高字节，保存在内存的低地址中，而数据的低字节，保存在内存的高地址中，这样的存储模式有点儿类似于把数据当作字符串顺序处理：地址由小向大增加，而数据从高位往低位放；

所谓小端模式（Little-endian）, 是指数据的高字节保存在内存的高地址中,而数据的低字节保存在内在的低地址中,这种存储模式将地址的高低和数据位 权有效结合起来,高地址部分权值高,低地址部分权值低,和我们的逻辑方法一致;

## 2/ 为什么有大小端之分:

因为在计算机系统中，我们是以字节为单位的，每个地址单元都对应着一个字节，一个字节为 8bit。但是在C语言中除了8bit的char之外，还有16bit的short型，32bit的long型（要看具体的编译器），另外，对于位数大于 8位的处理器，例如16位或者32位的处理器，由于寄存器宽度大于一个字节，那么必然存在着一个如何将多个字节安排的问题。因此就导致了大端存储模式和小端存储模式。我们常用的X86结构是小端模式，而KEIL C51则为大端模式。很多的ARM，DSP都为小端模式。有些ARM处理器还可以由硬件来选择是大端模式还是小端模式。


```
比如 short int a = 0x1234
大尾存放时：
偏移地址      存放内容
0x0000       0x12
0x0001       0x34

小尾存放：
偏移地址      存放内容
0x0000       0x34
0x0001       0x12
```

同样的如果数据是32位、64位也就是可以类推。
判断一个机器是大尾还是小尾我们可以通过程序进行测试：

```
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    short int a = 0x1234;
    char *p = (char *)&a;

    printf("p=%#hhx\n", *p);

    if (*p == 0x34) {
        printf("little endian\n");    
    } else if (*p == 0x12) {
        printf("big endia\n");    
    } else {
        printf("unknown endia\n");    
    }
    return 0;
}

```

在网络上传输数据我们采用的都是大尾。这就存在字节顺序的相互转换。
下面定义一个宏可以对16位数据进行字节转换

```

#define sw16(x) \
    ((short)( \
        (((short)(x) & (short)0x00ffU) << 8 ) | \
        (((short)(x) & (short)0xff00U) >> 8 ) ))
```

假设这里x＝0xaabb
(short)(x) & (short)0x00ffU  这里的与操作将16位数据x的高8位置为0得到0x00bb,然后在左移8位就得到了0xbb00
同理（short）(x) & (short)0xff00U >> 8 就得到了 0x00aa
最后将0xbb00 和 0x00aa 进行或运算就实现了高字节和低字节的相会交换。

转换32位的更精简的方法，直接上代码（2016年11.28补充），可以由大端转为小端，也可以由小端转为大端：

```
uint32_t swap_endian(uint32_t val) {
      val = ((val << 8) & 0xFF00FF00) | ((val >> 8) & 0xFF00FF);
      return (val << 16) | (val >> 16);
 }
```