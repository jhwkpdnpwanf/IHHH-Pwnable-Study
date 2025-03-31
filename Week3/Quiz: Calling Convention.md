**Q1. 아래 코드를 컴파일 했을 때, 컴파일된 어셈블리 코드 중 (a)에 들어갈 것으로 가장 적절할 것을 고르시오.**

**Q2. 아래 코드를 컴파일 했을 때, 컴파일된 어셈블리 코드 중 (b)에 들어갈 것으로 가장 적절할 것을 고르시오.**

**Q3. 아래 코드를 컴파일 했을 때, 컴파일된 어셈블리 코드 중 (c)에 들어갈 것으로 가장 적절할 것을 고르시오.**

**Q4. 아래 코드를 컴파일 했을 때, 컴파일된 어셈블리 코드 중 (d)에 들어갈 것으로 가장 적절할 것을 고르시오.**

```c
// Name: callconv_quiz.c
// Compile: gcc -o callconv_quiz callconv_quiz.c -m32
int __attribute__((cdecl)) sum(int a1, int a2, int a3){
	return a1 + a2 + a3;
}
void main(){
	int total = 0;
	total = sum(1, 2, 3);
}

main:
   0x080483ed <+0>:	push   ebp
   0x080483ee <+1>:	mov    ebp,esp
   0x080483f0 <+3>:	sub    esp,0x10
   0x080483f3 <+6>:	mov    DWORD PTR [ebp-0x4],0x0
   0x080483fa <+13>:	(a)
   0x080483fc <+15>:	(b)
   0x080483fe <+17>:	(c)
   0x08048400 <+19>:	call   0x80483db <sum>
   0x08048405 <+24>:	(d)
   0x08048408 <+27>:	mov    DWORD PTR [ebp-0x4],eax
   0x0804840b <+30>:	nop
   0x0804840c <+31>:	leave
   0x0804840d <+32>:	ret
```

**Q1. 정답 : `push 0x3`**

**Q2. 정답 : `push 0x2`**

**Q3. 정답 : `push 0x1`**

**Q4. 정답 : `add esp, 0xc`**

- `sum(1, 2, 3);` 
인자들은 **오른쪽에서 왼쪽 순서**로 스택에 `push` 됨
- `caller`가 **인자 12바이트 스택 정리**

---

**Q5. SYSV를 적용하여 아래 코드를 컴파일 했을 때, 컴파일된 어셈블리 코드 중 (a)에 들어갈 것으로 가장 적절한 것을 고르시오.**

**Q6. SYSV를 적용하여 아래 코드를 컴파일 했을 때, 컴파일된 어셈블리 코드 중 (b)에 들어갈 것으로 가장 적절한 것을 고르시오.**

**Q7. SYSV를 적용하여 아래 코드를 컴파일 했을 때, 컴파일된 어셈블리 코드 중 (c)에 들어갈 것으로 가장 적절한 것을 고르시오.**

```c
// Name: callconv_quiz2
// Compile: gcc -o callconv_quiz2 callconv_quiz2.c
int sum(int a1, int a2, int a3){
	return a1 + a2 + a3;
}
void main(){
	int total = 0;
	total = sum(1, 2, 3);
}

main:
   0x00000000004004f2 <+0>:	push   rbp
   0x00000000004004f3 <+1>:	mov    rbp,rsp
   0x00000000004004f6 <+4>:	sub    rsp,0x10
   0x00000000004004fa <+8>:	mov    DWORD PTR [rbp-0x4],0x0
   0x0000000000400501 <+15>:	(a)
   0x0000000000400506 <+20>:	(b)
   0x000000000040050b <+25>:	(c)
   0x0000000000400510 <+30>:	call   0x4004d6 <sum>
   0x0000000000400515 <+35>:	mov    DWORD PTR [rbp-0x4],eax
   0x0000000000400518 <+38>:	nop
   0x0000000000400519 <+39>:	leave
   0x000000000040051a <+40>:	ret
```

**Q5. 정답 : mov edx, 0x3**

**Q6. 정답 : mov esi, 0x2**

**Q7. 정답 : mov edi, 0x1**

- `RDI`, `RSI`, `RDX`, `RCX`, `R8`, `R9` 순서대로 전

- `RDI`, `RSI`, `RDX`, `RCX`, `R8`, `R9` 순서대로 전달 됨
