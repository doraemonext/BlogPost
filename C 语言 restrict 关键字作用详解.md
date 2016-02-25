# C 语言 restrict 关键字作用详解

`restrict` 是 C99 中新增的关键字，仅用于限定指针。

使用该关键字修饰的指针会明确的告诉编译器：**所有修改该指针所指向内容的操作全都是基于该指针的。**换句话说，不存在除该指针外修改该指针指向内容的方式。

使用该关键字可以让编译器进行更好的代码优化，下面用具体示例来分析一下。

```cpp
#include <stdio.h>#include <stdlib.h>int calc(int *ptr1, int *ptr2){    *ptr1 = 10;    *ptr2 = 20;    return *ptr1;}int main(void){    int *a = malloc(sizeof(int));    int *b = malloc(sizeof(int));    printf("%d\n", calc(a, b));    return 0;}
```

注意上面代码中的 `calc` 函数，如果不仔细看的话会想当然的认为该函数会一直返回 10，然后认为编译器优化代码的时候会直接将其优化为常量。但结果并不是这个样子，下面是我们使用

```bash
gcc -S test.c -O2
```

生成优化后的汇编代码，可以看到，`calc` 对应的汇编代码如下：

```as
calc:.LFB39:	.cfi_startproc	movl	$10, (%rdi)	movl	$20, (%rsi)	movl	(%rdi), %eax	ret	.cfi_endproc
```

显然编译器并没有“聪明的”将返回值变为常量。为什么呢？

这个问题的关键在于，编译器并不知道传入的两个指针形参 `ptr1` 和 `ptr2` 是不是指向了相同的内存地址。**如果 `ptr1 == ptr2`，那么会发生什么？没错，返回值就变成了 20。**

在绝大多数情况下，我们调用函数并不会传入两个地址相同的指针去让它们重复操作，如果我们可以保证这一点，那么我们就可以使用 `restrict` 关键字修饰指针形参，告诉编译器，**所有修改该指针所指向内容的操作全都是基于该指针的，不会有别的指针来添乱**。

接下来我们的 `calc` 函数的定义就变成了下面这个样子：

```cpp
int calc(int *restrict ptr1, int *restrict ptr2){    *ptr1 = 10;    *ptr2 = 20;    return *ptr1;}
```

然后再使用

```bash
gcc -S test.c -std=c99 -O2
```

生成优化后的汇编代码（注意添加 `-std=c99`），可以看到，`calc` 新的对应的汇编代码如下：

```as
calc:.LFB23:	.cfi_startproc	movl	$10, (%rdi)	movl	$20, (%rsi)	movl	$10, %eax	ret	.cfi_endproc
```

现在编译器聪明的将返回语句优化为常量，进一步提升了效率。

PS: 如果函数形参中的指针被 `restrict` 修饰，然后你又作死的传入了相同地址的指针且在函数中进行修改，那么编译器可能会对代码做出错误的优化。比如上面的 `calc` 函数指针形参经过 `restrict` 修饰后，我还是调用 `calc(a, a)` 这种的话，将仍然会返回值 10，而不是正确答案 20。

