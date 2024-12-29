## bomb-lab

### 实验说明

本实验通过逆向的方式模拟拆炸弹的过程，炸弹共有6道锁，我们需要逐一破解每一道锁，最终拆除炸弹。我们可以通过执行./bomb开始输入密码，也可以把密码输入到任意文件中作为参数传递给./bomb，例如./bomb psol.txt

一些需要用到的工具：

- gdb（GNU调试器）

你可以逐行跟踪程序，检查内存和寄存器，查看源代码和汇编代码（我们不会给你炸弹的大部分源代码），设置断点、设置内存观察点，并编写脚本。

CMU提供了一个gdb快速参考页：[gdb参考文档](https://www.cs.cmu.edu/~gilpin/tutorial/)

gdb的一些小技巧：设置断点；设置断点可以防止程序因输入错误而爆炸。

在 gdb 命令提示符下输入 help，或在 Unix 提示符下输入 man gdb 或 info gdb 查看在线文档。

- objdump -t

这个工具可以打印出炸弹程序的符号表。

符号表包含了炸弹中所有函数和全局变量的名称、炸弹调用的所有函数的名称及其地址。

可以观察这个函数名称获得一些有用的信息。

- objdump -d

这个工具可以反汇编炸弹的所有代码，也可以选择只查看某些特定函数的代码。

阅读汇编代码可以帮助理解炸弹的工作原理。

- strings

这个工具会显示炸弹中所有可打印的字符串。

- man/info查看工具的用法

### 准备步骤

1.生成符号表

```
objdump -t bomb > ./symbol_table.txt
```

如下所示：

```
bomb:     file format elf64-x86-64


SYMBOL TABLE:
0000000000400238 l    d  .interp	0000000000000000              .interp
0000000000400254 l    d  .note.ABI-tag	0000000000000000              .note.ABI-tag
0000000000400274 l    d  .note.gnu.build-id	0000000000000000              .note.gnu.build-id
0000000000400298 l    d  .gnu.hash	0000000000000000              .gnu.hash
00000000004002c8 l    d  .dynsym	0000000000000000              .dynsym
00000000004005c8 l    d  .dynstr	0000000000000000              .dynstr
```

使用如下命令：

```
man objdump
```

可以查看每一列的含义：

第一列是地址；第二列是标志位，这里列出几种常见的：l代表本地，g代表全局，f代表文件，F代表函数，O代表对象；第三列是符号所属的段名称，或者是ABS，表示该段是绝对的，与任何段无关。或者是UND，表示没有定义在当前程序中；第四段是对齐方式/大小。最后会显示符号的名称。

2.生成所有汇编代码

```
objdump -d bomb > ./bomb.s
```

### 开始分析

#### 第一个密码

查看bomb.c文件，有如下代码：

```
    /* Do all sorts of secret stuff that makes the bomb harder to defuse. */
    initialize_bomb();

    printf("Welcome to my fiendish little bomb. You have 6 phases with\n");
    printf("which to blow yourself up. Have a nice day!\n");

    /* Hmm...  Six phases must be more secure than one phase! */
    input = read_line();             /* Get input                   */
    phase_1(input);                  /* Run the phase               */
    phase_defused();                 /* Drat!  They figured it out!
				      * Let me know how they did it. */
    printf("Phase 1 defused. How about the next one?\n");
```

可以看到phase_1函数判断input是否正确，查看bomb.s中phase_1的汇编代码：

```
0000000000400ee0 <phase_1>:
  400ee0:	48 83 ec 08          	sub    $0x8,%rsp
  400ee4:	be 00 24 40 00       	mov    $0x402400,%esi // esi是第二个参数
  400ee9:	e8 4a 04 00 00       	callq  401338 <strings_not_equal>
  400eee:	85 c0                	test   %eax,%eax
  400ef0:	74 05                	je     400ef7 <phase_1+0x17> // 如果返回值为0，正常退出
  400ef2:	e8 43 05 00 00       	callq  40143a <explode_bomb> // 返回值不为0，炸弹爆炸
  400ef7:	48 83 c4 08          	add    $0x8,%rsp
  400efb:	c3                   	retq   
```

可以看到，phase_1实际上是调用了strings_not_equal判断两个输入参数是否相等，是则解决，反之爆炸。可以写出phase_1的伪代码：

```
phase_1(rdi) {
  esi = 0x402400;
  eax = strings_not_equal(rdi, esi);
  if (!eax) {
    explode_bomb();
  }
}
```

查看bomb.s中main函数，定位到调用phase_1之前，查看rdi的内容：

```
  400e19:	e8 84 05 00 00       	callq  4013a2 <initialize_bomb>
  400e1e:	bf 38 23 40 00       	mov    $0x402338,%edi
  400e23:	e8 e8 fc ff ff       	callq  400b10 <puts@plt>
  400e28:	bf 78 23 40 00       	mov    $0x402378,%edi
  400e2d:	e8 de fc ff ff       	callq  400b10 <puts@plt>
  400e32:	e8 67 06 00 00       	callq  40149e <read_line>
  400e37:	48 89 c7             	mov    %rax,%rdi // %rax是read_line的返回值，也就是输入的字符串,作为参数传递给phase_1
  400e3a:	e8 a1 00 00 00       	callq  400ee0 <phase_1>
```

可知，rdi就是我们输入的字符串。大胆猜测strings_not_equal这里就是在判断我们的输入与0x402400处的字符串是否相等。查看strings_not_equal汇编代码求证，其写成伪代码逻辑为：

```
strings_not_equals(rdi, rsi){
	//第一个字符串
	rbx = rdi;
	//第二个字符串
	rbp = rsi;
	eax = string_length(rdi);
	//r12d 存长度
	r12d = eax;
	rdi = rbp;
	eax = string_length(rdi);
  if(eax != r12d) retrun eax = edx = 1;
	eax = *(rbx);
	for(al != 0){
		if(al != *(rbp)) retrun eax = edx = 1;
		rbx++;
		rbp++;
		eax = *(rbx);
	}
	return eax = edx = 0;
}
```

就是一个判断两个字符串是否相等的函数，所以第一个密码就是0x402400处的字符串。可以通过gdb输出：

```
(gdb) x/s 0x402400
0x402400:       "Border relations with Canada have never been better."
```

x: 表示执行“检查内存”的命令;/s: 指定检查内存的格式为字符串 (string);0x402338: 指定要检查的内存地址。

#### 第二个密码

定位到phase_2调用的汇编代码：

```
  400e4e:	e8 4b 06 00 00       	callq  40149e <read_line>
  400e53:	48 89 c7             	mov    %rax,%rdi // 同样字符串输入作为参数rdi
  400e56:	e8 a1 00 00 00       	callq  400efc <phase_2> 
  400e5b:	e8 64 07 00 00       	callq  4015c4 <phase_defused>
```

查看phase_2的汇编代码：

```
0000000000400efc <phase_2>:
  400efc:	55                   	push   %rbp
  400efd:	53                   	push   %rbx
  400efe:	48 83 ec 28          	sub    $0x28,%rsp // 开辟栈操作
  400f02:	48 89 e6             	mov    %rsp,%rsi // 此时的rsp栈指针作为第二个参数rsi
  400f05:	e8 52 05 00 00       	callq  40145c <read_six_numbers>
  400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp) // 比较(%rsp)与0x1
  400f0e:	74 20                	je     400f30 <phase_2+0x34> // 相等则跳转到400f30
  400f10:	e8 25 05 00 00       	callq  40143a <explode_bomb> // 不相等炸弹爆炸
  400f15:	eb 19                	jmp    400f30 <phase_2+0x34>
  400f17:	8b 43 fc             	mov    -0x4(%rbx),%eax // eax = (rbx - 0x4)
  400f1a:	01 c0                	add    %eax,%eax // eax = eax + eax
  400f1c:	39 03                	cmp    %eax,(%rbx) // 
  400f1e:	74 05                	je     400f25 <phase_2+0x29>
  400f20:	e8 15 05 00 00       	callq  40143a <explode_bomb>
  400f25:	48 83 c3 04          	add    $0x4,%rbx
  400f29:	48 39 eb             	cmp    %rbp,%rbx
  400f2c:	75 e9                	jne    400f17 <phase_2+0x1b>
  400f2e:	eb 0c                	jmp    400f3c <phase_2+0x40>
  400f30:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx // rbx = 0x4 + rsp
  400f35:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp // rbp = 0x18 + rsp
  400f3a:	eb db                	jmp    400f17 <phase_2+0x1b> // 跳转到400f17
  400f3c:	48 83 c4 28          	add    $0x28,%rsp // 销毁栈操作
  400f40:	5b                   	pop    %rbx
  400f41:	5d                   	pop    %rbp
  400f42:	c3                   	retq   
```

可知，phase_2调用read_six_numbers，rdi、rsi作为两个输入参数，查看其汇编代码：

```
000000000040145c <read_six_numbers>:
  40145c:	48 83 ec 18          	sub    $0x18,%rsp
  401460:	48 89 f2             	mov    %rsi,%rdx
  401463:	48 8d 4e 04          	lea    0x4(%rsi),%rcx
  401467:	48 8d 46 14          	lea    0x14(%rsi),%rax
  40146b:	48 89 44 24 08       	mov    %rax,0x8(%rsp)
  401470:	48 8d 46 10          	lea    0x10(%rsi),%rax
  401474:	48 89 04 24          	mov    %rax,(%rsp)
  401478:	4c 8d 4e 0c          	lea    0xc(%rsi),%r9
  40147c:	4c 8d 46 08          	lea    0x8(%rsi),%r8
  401480:	be c3 25 40 00       	mov    $0x4025c3,%esi
  401485:	b8 00 00 00 00       	mov    $0x0,%eax
  40148a:	e8 61 f7 ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  40148f:	83 f8 05             	cmp    $0x5,%eax
  401492:	7f 05                	jg     401499 <read_six_numbers+0x3d>
  401494:	e8 a1 ff ff ff       	callq  40143a <explode_bomb>
  401499:	48 83 c4 18          	add    $0x18,%rsp
  40149d:	c3                   	retq   
```

可知，在40148a处，调用了sscanf函数，sscanf函数的原型如下：

```
int sscanf(const char *str, const char *format, ...);
```

第一个str表示输入字符串的地址，sscanf 会从这个字符串中解析数据。第二个format是格式化字符串，定义如何解析输入数据，例如："%d %d"。后面的...表示可变参数，是一个或多个指针，用于存储解析出的数据。每个指针对应 format 中的一个格式说明符。

实际上，这个考察了当参数数量超过可用寄存器数量（6个，分别是rdi、rsi、rdx、rcx、r8、r9），超出6个的部分就必须要通过栈来传递。read_six_numbers中的汇编代码大部分都是在初始化这些参数，包括初始化寄存器和栈上需要传递的值。由地址401480可知，0x4025c3作为第二个参数format，通过gdb可以查看该参数的值：

```
(gdb) x/s 0x4025c3
0x4025c3:       "%d %d %d %d %d %d"
```

可知，这里需要解析6个int整型数据。所以我们知道这里实际上在为sscanf准备8个参数（str、format和6个存储整型数据的指针），可见下面的分析图：

![bomb_2.png](pictures/bomb_2.png)

