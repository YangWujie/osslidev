---
theme: seriph
class: text-center
background: https://cover.sli.dev

title: 操作系统引论
---

# 操作系统引论

---

# 1.1 操作系统的目标和作用

---

## 1.1.1 操作系统的目标

---

* 方便性
  * 通过OS命令操纵计算机，方便用户
* 有效性
  * 提高系统资源的利用率
  * 提高系统吞吐量
* 可扩充性
  * OS必须具有很好的可扩充性
  * 与OS的结构有紧密的联系
* 开放性
  * 遵循世界标准规范。特别是开放系统互连OSI

---

### 纠错

---

**“用户如果想直接在计算机硬件上运行自己所编写的程序，就必须用机器语言编写程序”**  
   - ❌ 不准确。  
   - 即使没有操作系统，也完全可以在别的机器上通过交叉编译的方式利用 **汇编器** 或 **编译器**，把高级语言翻译成机器代码，再写入“裸机”上并执行。  
   - 操作系统不是编译高级语言的必要条件。  

---

**“近年来OS已广泛采用微内核结构”**
   - ❌ 不准确。  
   - **微内核并非广泛应用：** 尽管微内核结构（如 Mach、Minix）在学术界和特定领域（如实时操作系统、嵌入式系统）得到应用，但主流的通用操作系统（如 Linux、Windows、macOS）仍然是宏内核（monolithic kernel）或混合式内核（hybrid kernel）架构。宏内核因其性能优势和历史积累，在桌面和服务器市场占据主导地位。

---

**“所谓开放性，是指系统能够遵循国际标准，特别是遵循开放系统互连（open system interconnect，OSI）参考模型”**
   - ❌ 对开放性定义的错误。
   - **OSI 模型是网络协议栈标准：** OSI 参考模型是一个描述网络通信的七层抽象模型。它是一个网络协议标准，与操作系统的开放性（即其内核接口、系统调用、文件系统等）是完全不同的概念。
   - **开放性与 POSIX 标准更相关：** 操作系统的开放性通常指的是它遵循 POSIX 标准，旨在保证不同操作系统间程序可移植性，从而实现源代码级别的开放和可移植。这是操作系统开放性最核心的体现。
　
---

## 操作系统的作用

---

### 用户与计算机硬件之间的接口
- 命令方式（UNIX、DOS命令）；
- 系统调用方式（API）；
- GUI方式（Windows、LINUX）

---
class: bg-white
---

![OS作为接口的示意](/os_interface.svg)

---

### 纠错

- ❌ 作者将“命令方式”、“系统调用方式”和“图形/窗口方式”描述为用户使用计算机的“3 种并列方式”，这种分类方式是不正确的。

- 系统调用是基础机制：系统调用（System Calls）是应用程序与操作系统内核通信的唯一方式。无论是通过命令行、图形界面还是其他任何方式，最终要访问硬件资源或 OS 服务，都必须通过系统调用。

---

### 计算机系统资源的管理者
- 处理机管理
- 存储器管理
- I/O设备管理
- 文件管理

---

### 实现对计算机资源的抽象
- 裸机：无软件的计算机系统。
- 虚拟机：覆盖了软件的机器，向用户提供一个对硬件操作的抽象模型。

---

## 推动操作系统发展的主要动力

- 不断提高计算机系统资源的利用率
- 方便用户
- 器件不断更新换代
- 计算机体系结构不断发展
- 不断提出新的应用需求

---

## 建议

对于 Windows 用户：
- 开启 WSL2：[How to install Linux on Windows with WSL](https://learn.microsoft.com/en-us/windows/wsl/install)
- 在 Linux 上安装 gcc 工具链：
```bash
sudo apt install build-essential
```

对于 Mac 用户：
- 安装 Xcode 命令行工具：
```bash
xcode-select --install
```

---

## Hello World!
编译源文件 hello.c：
```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```

编译和运行：
```bash
gcc hello.c -o hello
./hello
```

---

### 命令依赖于系统调用
hello、strace、gcc都是命令。我们通常不认为命令是操作系统的一部分。命令的执行依赖于系统调用，如下命令可以证实：
- strace ./hello
- strace strace
- strace gcc

```
write(1, "Hello, World!\n", 14Hello, World!
)         = 14
exit_group(0)                           = ?
+++ exited with 0 +++
```

---

### 查找系统调用对应函数的入口
```{8,11}
yang@wsl:~$ readelf -s /lib/aarch64-linux-gnu/libc.so.6 | grep write
   104: 00000000000e8170   244 FUNC    GLOBAL DEFAULT   12 pwritev@@GLIBC_2.17
   129: 00000000000eb5e0   208 FUNC    WEAK   DEFAULT   12 writev@@GLIBC_2.17
  1871: 00000000000e8170   244 FUNC    GLOBAL DEFAULT   12 pwritev64@@GLIBC_2.17
  2118: 00000000000c6d10   212 FUNC    WEAK   DEFAULT   12 pwrite@@GLIBC_2.17
  2149: 0000000000090c20    32 FUNC    GLOBAL DEFAULT   12 aio_write@@GLIBC_2.34
  2151: 0000000000090c20    32 FUNC    GLOBAL DEFAULT   12 aio_write@GLIBC_2.17
  2579: 00000000000e1d20   204 FUNC    WEAK   DEFAULT   12 __write@@GLIBC_2.17
  2609: 00000000000c6d10   212 FUNC    WEAK   DEFAULT   12 pwrite64@@GLIBC_2.17
  2776: 00000000000708a0   584 FUNC    WEAK   DEFAULT   12 fwrite@@GLIBC_2.17
  2871: 00000000000e1d20   204 FUNC    WEAK   DEFAULT   12 write@@GLIBC_2.17
  2883: 00000000000e8270   412 FUNC    GLOBAL DEFAULT   12 pwritev2@@GLIBC_2.26
```

---

### 系统调用指令

```{13}
yang@wsl:~$ objdump -D /lib/aarch64-linux-gnu/libc.so.6 --start-address=0x000e1d20 --stop-address=$((0x000e1d20+100))

00000000000e1d20 <__write@@GLIBC_2.17>:
   e1d20:       a9bd7bfd        stp     x29, x30, [sp, #-48]!
   e1d24:       d00006a3        adrp    x3, 1b7000 <re_syntax_options@@GLIBC_2.17+0x518>
   e1d28:       910003fd        mov     x29, sp
   e1d2c:       3954a063        ldrb    w3, [x3, #1320]
   e1d30:       a90153f3        stp     x19, x20, [sp, #16]
   e1d34:       93407c13        sxtw    x19, w0
   e1d38:       34000163        cbz     w3, e1d64 <__write@@GLIBC_2.17+0x44>
   e1d3c:       aa1303e0        mov     x0, x19
   e1d40:       d2800808        mov     x8, #0x40                       // #64
   e1d44:       d4000001        svc     #0x0
   e1d48:       aa0003f3        mov     x19, x0
   e1d4c:       b140041f        cmn     x0, #0x1, lsl #12
   e1d50:       54000328        b.hi    e1db4 <__write@@GLIBC_2.17+0x94>  // b.pmore
   e1d54:       aa1303e0        mov     x0, x19
   e1d58:       a94153f3        ldp     x19, x20, [sp, #16]
   e1d5c:       a8c37bfd        ldp     x29, x30, [sp], #48
   e1d60:       d65f03c0        ret
```

---

### svc

svc 指令的全称是 Supervisor Call，翻译过来就是“管理模式调用”或“特权模式调用”。

它的主要作用是触发一个软件中断，将程序的执行模式从用户空间（User Mode）切换到特权模式（Supervisor Mode）。在 ARM 架构中，操作系统内核就运行在特权模式下，拥有对系统资源的完全控制权。

---

### 应用看到的资源和设备是高度抽象化的
应用程序使用的标准输出设备高度抽象：它可以是控制台，可以是乌有乡，可以是磁盘文件，可以是管道。它们共有一个抽象特征：接收字节流
```bash
strace ./hello
strace ./hello 1> /dev/null
strace ./hello 1> out.txt
./hello | wc
```

---

## 引导扇区

汇编源文件 myfirst.asm：
```asm {*}{maxHeight:'300px'}
org 7C00H                      ; the program will be loaded at 7C00H

start:
  mov eax, string_start
  mov ch, 1                    ; ch contains color of text
  mov ebx, 0B8000H + 718H      ; B8000H is VGA memory
                               ; 718H is offset to approx center

print:
  mov cl, [eax]                ; load char into cl
  mov [ebx], cx                ; store [color:char] from cx into VGA
  add ch, 1                    ; change color to (ch+1) mod 16
  and ch, 0x0F
  add eax, 1                   ; advance string pointer
  add ebx, 2                   ; advance VGA pointer
  cmp eax, string_end          ; until the end of string
  jg stop
  jmp print

stop:
  jmp stop                     ; infinite loop after printing

  string_start db 'My colorful new OS!'
  string_end equ $

  times 510-($-$$) db 0        ; pad remainder of boot sector with 0s
  dw 0xAA55                    ; standard PC boot signature
```

---


### 没有 printf 可以调用。为什么？

---

### 验证

汇编并用 qemu 创建虚拟机引导：
```bash
sudo apt install nasm
sudo apt install qemu

nasm -f bin -o myfirst.bin myfirst.asm
qemu-system-i386 -hda myfirst.bin
```

---

## 用 C 语言写的引导扇区

C 源文件 test.c：
```c {*}{maxHeight:'300px'}
__asm__(".code16gcc\n");
__asm__("jmpl $0x0000, $main\n");

#define MAX_COLS     320
#define MAX_ROWS     200

void drawPixel(unsigned char color, int col, int row) {
     __asm__ __volatile__ (
          "int $0x10" : : "a"(0x0c00 | color), "c"(col), "d"(row)
     );
}

void initEnvironment() {
     __asm__ __volatile__ (
          "int $0x10" : : "a"(0x03)
     );
     __asm__ __volatile__ (
          "int $0x10" : : "a"(0x0013)
     );
}

void initGraphics() {
     int i = 0, j = 0;
     int m = 0;
     int cnt1 = 0, cnt2 =0;
     unsigned char color = 10;

     for(;;) {
          if(m < (MAX_ROWS - m)) {
               ++cnt1;
          }
          if(m < (MAX_COLS - m - 3)) {
               ++cnt2;
          }

          if(cnt1 != cnt2) {
               cnt1  = 0;
               cnt2  = 0;
               m     = 0;
               if(++color > 255) color= 0;
          }

          j = 0;
          for(i = m; i < MAX_ROWS - m; ++i) {
               drawPixel(color, j+m, i);
          }
          for(j = m; j < MAX_COLS - m; ++j) {
               drawPixel(color, j, i);
          }

          for(i = MAX_ROWS - m - 1 ; i >= m; --i) {
               drawPixel(color, MAX_COLS - m - 1, i);
          }
          for(j = MAX_COLS - m - 1; j >= m; --j) {
               drawPixel(color, j, m);
          }
          m += 6;
          if(++color > 255)  color = 0;
     }
}

void main() {
     initEnvironment();
     initGraphics();
}
```
---

### 链接脚本

test.ld：
```asm {*}{maxHeight:'300px'}
OUTPUT_FORMAT("elf32-i386")
OUTPUT_ARCH(i386)
ENTRY(main)

SECTIONS
{
    . = 0x7c00; # Define the start address of the boot sector

    .text :
    {
        *(.text)
        *(.text.*)
    }

    /* Potentially other sections are implicitly included or explicitly defined here */

    .sig 0x7dfe : AT(0x7dfe) {
        SHORT(0xAA55)
    }
    /DISCARD/ : { *(.comment) *(.eh_frame) *(.note.gnu.property) *(.*debug*) } # Discard unwanted sections
}
```

---

### 验证

安装交叉编译工具并编译、链接、运行：
```bash
sudo apt install gcc-i686-linux-gnu

i686-linux-gnu-gcc -c -Os -m16 -ffreestanding -Wall -Werror -o test.o test.c
i686-linux-gnu-ld -static -Ttest.ld -melf_i386 -nostdlib --nmagic -o test.elf test.o
i686-linux-gnu-objcopy -O binary test.elf test.bin

qemu-system-i386 -hda test.bin
```

---

## 操作系统的发展过程
- 无操作系统的计算机系统
- 单道批处理系统
- 多道批处理系统
- 分时系统
- 实时系统
- 微机操作系统
- 嵌入式操作系统
- 网络操作系统
- 分布式操作系统

---

<style global>
.slidev-layout {
  font-size: 28px;
}

.slidev-layout h1 {
  font-size: 64px; /* Set font size for h1 (main title) */
}

.slidev-layout h2 {
  font-size: 52px; /* Set font size for h2 (subtitles) */
}

.slidev-layout h3 {
  font-size: 40px; /* Set font size for h3, if needed */
}

/* 确保标题和内容的间距 */
.slidev-layout h1, 
.slidev-layout h2, 
.slidev-layout h3 {
  margin-top: 2.5rem;
  margin-bottom: 3.5rem;
}

.slidev-layout p {
  line-height: 1.5;
}

</style>