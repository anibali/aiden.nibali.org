+++
title = "Hello world of assembly"
date = "2015-01-17T00:00:00.000Z"
categories = ["misc"]
tags = ["assembly"]
comments = false
draft = false
showpagemeta = true
slug = ""
+++

I've found it surprisingly difficult to find current resources about learning
assembly, particularly for 64-bit systems. In fact, many of the guides I found
are still using the [outdated](http://stackoverflow.com/a/12806910) `int 80h`
instruction to invoke system calls.

By gleaning bits of information from varied sources I was finally able to
cobble together a working "Hello world" program in NASM assembly for 64-bit
Linux.

```x86asm
;;; hello.nasm ;;;
;
; 64-bit "Hello world" program for Linux
;
; Compiling and linking:
;    nasm -f elf64 hello.nasm
;    ld -o hello hello.o

; Preprocessor defines
%define SysWrite  0x01    ; System call code for writing
%define Stdout    1       ; File handle for standard output

%define SysExit   0x3C    ; System call code for exiting
%define Success   0       ; Successful exit status

%define Newline   10      ; The newline character, '\n'

; Constants section
section .data
  ; Message to print as a sequence of bytes
  Greeting:       db 'Hello there!', Newline
  ; GreetingLength = current address - address of GreetingCurrent
  GreetingLength: equ $ - Greeting

; Code section
section .text
global _start

; Code execution entry point
_start:
  mov rax, SysWrite         ; Set "Write" system call code
  mov rdi, Stdout           ; Set output file handle to stdout
  mov rsi, Greeting         ; Set output string to Greeting
  mov rdx, GreetingLength   ; Set output string length to GreetingLength
  syscall                   ; Perform system call

  mov rax, SysExit          ; Set "Exit" system call code
  mov rdi, Success          ; Set successful exit status
  syscall                   ; Perform system call
```

A key point to note is the usage of `syscall` rather than `int 80h`. This
instruction requires different sys codes, which is something that threw me
off for a while. For example, sys_write is 0x04 for `int 80h` but you need to
use 0x01 for `syscall`.

Fortunately, I found an excellent
[reference for syscall](https://filippo.io/linux-syscall-table/)
which helped a great deal.

I also discovered that different registers are used to pass arguments with
`syscall`. Thanks to
[a post on 64-bit shellcode](http://codinguy.net/2013/09/30/execve-shellcode-64-bit/)
I realised that the correct register-to-argument mapping is:

```
RAX = System call number
RDI = First argument
RSI = Second argument
RDX = Third argument
R10 = Fourth argument
R8  = Fifth argument
R9  = Sixth argument
```

This is quite different (and less intuitive) than `int 80h`, which simply uses
RAX, RBX, RCX, and so forth.
