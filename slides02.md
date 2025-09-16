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

## 操作系统和浏览器之间的区别？
- 浏览器能称得上一个操作系统吗？
- 如果浏览器不能称得上一个操作系统，那么 WSL 和 Lima 能称得上操作系统吗？
- WSL 能称得上操作系统，浏览器为什么不能？

---

### 浏览器里的操作系统
- [v86](https://copy.sh/v86/)
- [Puter](https://puter.com/)
- Many other...

---

### 操作系统和浏览器的区别到底是什么？
- 虚拟无处不在：操作系统和浏览器都有虚拟能力
- 虚实并无明显界限：浏览器里的操作系统并不知道它运行在浏览器里
- 操作系统认为自己直接运行在硬件之上并负责管理硬件
- 操作系统对硬件有要求，需要得到硬件某些方面的支持
- 操作系统得不到软件支持：没有库函数

---

### 从程序员的角度看
- 每个程序有可以执行代码的CPU，有能用来存储数据和代码的存储空间
- 多个程序可以并发执行
- 每个程序有自己的输入输出设备

---

### 虚拟 CPU: cpu.c

```c {*}{maxHeight:'400px'}
  #include <stdio.h>
  #include <stdlib.h>
  #include <sys/time.h>
  #include <sys/stat.h>
  #include <assert.h>
      
  double GetTime() {
    struct timeval t;
    int rc = gettimeofday(&t, NULL);
    assert(rc == 0);
    return (double) t.tv_sec + (double) t.tv_usec/1e6;
  }
  
  void Spin(int howlong) {
    double t = GetTime();
    while ((GetTime() - t) < (double) howlong)
    ; // do nothing in loop
  }
  
  int main(int argc, char *argv[])
  {
    if (argc != 2) {
      fprintf(stderr, "usage: cpu <string>\n");
      exit(1);
    }
    char *str = argv[1];
  
    while (1) {
      printf("%s\n", str);
      Spin(1);
    }
    return 0;
  }

```

---

```bash
gcc cpu.c -o cpu
./cpu "A"
```

该程序运行后，每隔1秒打印一个“A”并换行。

按 Ctrl + c 可结束程序。

---

```bash
./cpu "A" & ./cpu "B" & ./cpu "C" & ./cpu "D" &

killall cpu
```

---

### mem.c

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>

int main(int argc, char *argv[]) {
    if (argc != 2) { 
        fprintf(stderr, "usage: mem <value>\n"); 
        exit(1); 
    } 
    int *p; 
    p = malloc(sizeof(int));
    assert(p != NULL);
    printf("(%d) addr pointed to by p: %p\n", (int) getpid(), p);
    *p = atoi(argv[1]); // assign value to addr stored in p
    while (1) {
        sleep(1);
        *p = *p + 1;
        printf("(%d) value of p: %d\n", getpid(), *p);
    }
    return 0;
}
```

---

```bash
gcc mem.c -o mem
./mem "100"
```

该程序运行后，每隔1秒打印一行信息，其中包括进程号及某个内存单元中的值。

按 Ctrl + c 可结束程序。

---

### Address Space Layout Randomization

运行mem之前，首先禁用Linux的ASLR。命令如下：

```bash
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
```

详细信息请参考：[How can I temporarily disable ASLR?](https://askubuntu.com/questions/318315/how-can-i-temporarily-disable-aslr-address-space-layout-randomization)

---

### ./mem 100
```
          (16712) addr pointed to by p: 0x555555756260
          (16712) value of p: 101
          (16712) value of p: 102
          (16712) value of p: 103
          (16712) value of p: 104
          (16712) value of p: 105
          (16712) value of p: 106
          (16712) value of p: 107
          (16712) value of p: 108
```

---

### ./mem 1
```
          (16815) addr pointed to by p: 0x555555756260
          (16815) value of p: 2
          (16815) value of p: 3
          (16815) value of p: 4
          (16815) value of p: 5
          (16815) value of p: 6
          (16815) value of p: 7
          (16815) value of p: 8
          (16815) value of p: 9
```
        
相同的地址，不同的数据：虚拟存储器

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
- 安装 [Lima](https://lima-vm.io/docs/installation/)：
- 在 Linux 上安装 gcc 工具链：
```bash
sudo apt install build-essential
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

### 查看动态库的名字

```bash
yang@SurfaceLaptop4:~$ ldd ./hello
        linux-vdso.so.1 (0x00007ffc202f1000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007864d3600000)
        /lib64/ld-linux-x86-64.so.2 (0x00007864d38df000)
```
---

### 查找系统调用对应函数的入口

```{1,8,11}
yang@SurfaceLaptop4:~$ readelf -s /lib/x86_64-linux-gnu/libc.so.6 | grep write
   101: 0000000000126160   173 FUNC    GLOBAL DEFAULT   17 pwritev@@GLIBC_2.10
   129: 0000000000129850   149 FUNC    WEAK   DEFAULT   17 writev@@GLIBC_2.2.5
  1931: 0000000000126160   173 FUNC    GLOBAL DEFAULT   17 pwritev64@@GLIBC_2.10
  2182: 00000000000fa500   157 FUNC    WEAK   DEFAULT   17 pwrite@@GLIBC_2.2.5
  2213: 00000000000a7b90    26 FUNC    GLOBAL DEFAULT   17 aio_write@@GLIBC_2.34
  2217: 00000000000a7b90    26 FUNC    GLOBAL DEFAULT   17 aio_write@GLIBC_2.2.5
  2650: 000000000011c560   157 FUNC    WEAK   DEFAULT   17 __write@@GLIBC_2.2.5
  2680: 00000000000fa500   157 FUNC    WEAK   DEFAULT   17 pwrite64@@GLIBC_2.2.5
  2851: 0000000000086940   510 FUNC    WEAK   DEFAULT   17 fwrite@@GLIBC_2.2.5
  2948: 000000000011c560   157 FUNC    WEAK   DEFAULT   17 write@@GLIBC_2.2.5
  2961: 0000000000126210   347 FUNC    GLOBAL DEFAULT   17 pwritev2@@GLIBC_2.26
```

---

### 系统调用指令

```{1,10}
yang@SurfaceLaptop4:~$ objdump -D /lib/x86_64-linux-gnu/libc.so.6 --start-address=0x000000000011c560 --stop-address
=$((0x000000000011c560+100))
/lib/x86_64-linux-gnu/libc.so.6:     file format elf64-x86-64
Disassembly of section .text:
000000000011c560 <__write@@GLIBC_2.2.5>:
  11c560:       f3 0f 1e fa             endbr64
  11c564:       80 3d d5 ea 0e 00 00    cmpb   $0x0,0xeead5(%rip)        # 20b040 <__libc_single_threaded@@GLIBC_2.32>
  11c56b:       74 13                   je     11c580 <__write@@GLIBC_2.2.5+0x20>
  11c56d:       b8 01 00 00 00          mov    $0x1,%eax
  11c572:       0f 05                   syscall
  11c574:       48 3d 00 f0 ff ff       cmp    $0xfffffffffffff000,%rax
  11c57a:       77 54                   ja     11c5d0 <__write@@GLIBC_2.2.5+0x70>
  11c57c:       c3
```

---

### 查找系统调用对应函数的入口(ARM)
```{1,8,11}
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

### 系统调用指令(ARM)

```{1,13}
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

### fork 系统调用
```c {*}{maxHeight:'400px'}
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
  printf("hello (pid:%d)\n", (int) getpid());
  int rc = fork();
  if (rc < 0) {
    // fork failed
    fprintf(stderr, "fork failed\n");
    exit(1);
  } else if (rc == 0) {
    // child (new process)
    printf("child (pid:%d)\n", (int) getpid());
  } else {
    // parent goes down this path (main)
    printf("parent of %d (pid:%d)\n", rc, (int) getpid());
  }

  return 0;
}
```

---

### wait 系统调用
```c {*}{maxHeight:'400px'}
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
  printf("hello (pid:%d)\n", (int) getpid());
  int rc = fork();
  if (rc < 0) {
    // fork failed
    fprintf(stderr, "fork failed\n");
    exit(1);
  } else if (rc == 0) {
    // child (new process)
    printf("child (pid:%d)\n", (int) getpid());
  } else {
    // parent goes down this path (main)
    int rc_wait = wait(NULL);
    printf("parent of %d (rc_wait:%d) (pid:%d)\n", rc, rc_wait, (int) getpid());
  }

  return 0;
}
```

---

### exec 系统调用
```c {*}{maxHeight:'400px'}
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
  printf("hello (pid:%d)\n", (int) getpid());
  int rc = fork();
  if (rc < 0) {
    // fork failed
    fprintf(stderr, "fork failed\n");
    exit(1);
  } else if (rc == 0) {
    // child (new process)
    printf("child (pid:%d)\n", (int) getpid());
    char *myargs[3];
    myargs[0] = strdup("wc"); // program: "wc"
    myargs[1] = strdup("p3.c"); // arg: input file
    myargs[2] = NULL; // mark end of array
    execvp(myargs[0], myargs); // runs word count
    printf("this shouldn’t print out");
  } else {
    // parent goes down this path (main)
    int rc_wait = wait(NULL);
    printf("parent of %d (rc_wait:%d) (pid:%d)\n", rc, rc_wait, (int) getpid());
  }

  return 0;
}
```

---

### 为什么要分成两个步骤？
灵活。在 shell 里非常容易实现重定向：

```c {*}{maxHeight:'400px'}
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
  int rc = fork();
  if (rc < 0) {
    // fork failed
    fprintf(stderr, "fork failed\n");
    exit(1);
  } else if (rc == 0) {
    // child: redirect standard output to a file
    close(STDOUT_FILENO);
    open("./p4.output", O_CREAT|O_WRONLY|O_TRUNC, S_IRWXU);
    // now exec "wc"...
    char *myargs[3];
    myargs[0] = strdup("wc"); // program: "wc"
    myargs[1] = strdup("p4.c"); // arg: input file
    myargs[2] = NULL;
    // mark end of array
    execvp(myargs[0], myargs); // runs word count
    printf("this shouldn’t print out");
  } else {
    // parent goes down this path (main)
    int rc_wait = wait(NULL);
  }

  return 0;
}
```

---

## xv6

[Xv6, a simple Unix-like teaching operating system](https://pdos.csail.mit.edu/6.828/2025/xv6.html)

---

### xv6 环境准备

```bash
sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
```

---

### xv6 概览
```bash
git clone git://g.csail.mit.edu/xv6-labs-2025
cd cd xv6-labs-2025
git ls-files | xargs wc -l
```
---

### 用户态概览
- user 目录里的代码运行在用户态
- 里面有很多常用命令
- 其中最重要的是 shell (sh.c)
- 用户态代码一般不认为是操作系统的组成部分

---

### 系统调用：从用户程序到内核代码
- user/user.h: 定义系统调用原型
- user/usys.pl: perl脚本，生成系统调用汇编指令（也可直接手写），存放在usys.S中
- user/usys.S中使用了SYS_fork, SYS_exit等符号
- 这些符号在 kernel/syscall.h 中定义
- 在 kernel/syscall.c 中，这些符号关联到具体的内核函数

---

### 在 xv6 里写第一个程序
参考材料：
- [Lab: Xv6 and Unix utilities](https://pdos.csail.mit.edu/6.828/2025/labs/util.html)
- [参考代码](https://github.com/YangWujie/xv6-lab-answer)

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