### **Q1. stderr, stdin, stdout의 파일 지정자의 올바른 값을 고르시오.**

---

**정답** : **stderr = 2, stdin = 0, stdout = 1**

### **Q2. 실습 환경에서 execve로 셸을 획득하려고 할 때, 다음 셸코드의 (a)에 들어갈 값을 고르시오.**

---

```nasm
pwndbg> x/s 0x7fffffffc278
"/bin/sh\x00"

mov rdi, (a)
xor rsi, rsi
xor rdx, rdx
mov rax, (b)
syscall
```

**정답** : **0x7fffffffc278**

- **`execve(“/bin/sh”, null, null)`**

**Q3. 실습 환경에서 execve로 셸을 획득하려고 할 때, 다음 셸코드의 (b)에 들어갈 값을 고르시오.**

---

```nasm
pwndbg> x/s 0x7fffffffc278
"/bin/sh\x00"

mov rdi, 0x7fffffffc278
xor rsi, rsi 
xor rdx, rdx 
mov rax, (b)  
syscall
```

**정답** : **0x3b**

- `rax` → `0x3b`
