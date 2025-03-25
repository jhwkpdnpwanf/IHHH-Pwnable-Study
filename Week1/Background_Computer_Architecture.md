# 컴퓨터 구조

### **컴퓨터 구조(Computer Architecture)**

- 컴퓨터가 효율적으로 작동할 수 있도록 **하드웨어 및 소프트웨어의 기능**을 고안하고, 이들을 **구성하는 방법**을 말한다.
<br>

**컴퓨터 구조의 세부 분야** 

```markdown
- 기능 구조의 설계
    ✔️ 폰 노이만 구조
    - 하버드 구조
    - 수정된 하버드 구조
  
- 명령어 집합구조
    ✔️ x86, x86-64
    - ARM
    - MIPS
    - AVR
    
- 마이크로 아키텍처
    - 캐시 설계
    - 파이프라이닝
    - 슈퍼 스칼라
    - 분기 예측
    - 비순차적 명령어 처리
    
- 하드웨어 및 컴퓨팅 방법론
    - 직접 메모리 접근
```

- **기능 구조의 설계** :  프로그램 실행 방식
- **명령어 집합 구조** : CPU 명령어 체계
- **마이크로 아키텍처** : CPU 내부 설계
- **하드웨어 및 컴퓨팅 방법론 :** 하드웨어 및 입출력 시스템

👉 **폰 노이만 구조**와 **x86-64 아키텍처**에 대해 다뤄볼 살펴볼 예정이다.

<br>

**

<br>

### 폰 노이만 구조

- **연산, 제어, 저장** 세가지 핵심 기능이 존재하는 컴퓨터 설계 모델이다.



1. **CPU (중앙처리장치) →** *연산&제어*
**-  ALU**  :   연산 수행
-  **Register**  :   데이터 임시 저장
-  **Cache**  :   자주 사용하는 데이터를 빠르게 불러오기 위한 저장 공간
2. **MEMORY →** *저장*
-  프로그램과 데이터를 동일한 메모리에 저장하는 공간
-  CPU가 데이터를 읽고 쓰는 작업을 수행
3. **BUS
-** 장치간에 데이터나 제어 신호를 교환할 수 있게 해주는 전자통로
   
<br>

**

<br>

# 세그먼트

- **리눅스 메모리 구조** : 프로세스의 메모리를 크게 5가지로 구분한 것.
    - 코드 세그먼트
    - 데이터 세그먼트
    - BSS 세그먼트
    - 힙 세그먼트
    - 스택 세그먼트  

<br>
*

### 코드 세그먼트

- 실행 가능한 기계 코드가 위치하는 영역
- **텍스트 세그먼트**라고도 불림
    - **부여 권한** : 읽기, 쓰기


**예시**

`int main() {return 31337}` →  `554889e5b8697a00005dc3` 기계 코드로 변환됨.

- 이 기계 코드가 **코드 세그먼트**에 위치하게 됨


<br>
*


### 데이터 세그먼트

- 컴파일 시점에 정해진 **전역 변수 및 전역 상수**들이 위치함.
    - **부여 권한** : 읽기, (쓰기)
        - **쓰기 권한**에 따라 세그먼트가 **다시 분류**됨.
            - **data 세그먼트**
            - **rodata(read-only data) 세그먼트**



**예시**

```c
int data_num = 31337;                       // data
char data_rwstr[] = "writable_data";        // data
const char data_rostr[] = "readonly_data";  // rodata
char *str_ptr = "readonly";  // str_ptr은 data, 문자열은 rodata

int main() { ... }
```

- `str_ptr`은 `readonly`라는 문자열을 가리키고 있는데, 
이 문자열은 **상수 문자열**로 취급되어 ***rodata* 에 위치**하며, 이를 가리키는 `str_ptr`은 **전역 변수**로서 ***data*에 위치**한다.

<br>
*

### BSS 세그먼트

- **컴파일 시점에 값이 정해지지 않은 전역변수**가 위치하는 메모리 영역.
- **선언만 하고 초기화하지 않은** **전역변수**
- 이 세그먼트의 메모리 영역은 프로그램이 시작될 때, 모두 0으로 값이 초기화됨
    - **부여 권한** : 읽기, 쓰기

**예시**

```c
int bss_data;

int main() {
  printf("%d\n", bss_data);  // 0
  return 0;
}
```

- **초기화되지 않은 전역 변수**인 `bss_data`가 **BSS 세그먼트**에 위치

<br>
*

### 스택 세그먼트

- 프로세스의 **스택**이 위치하는 영역
- **함수의 인자**나 **지역 변수**와 같은 **임시 변수**들이 실행 중에 여기에 저장됨.
    - **부여 권한** : 읽기, 쓰기

- 스택 세그먼트는 **스택 프레임(Stack Frame)** 이라는 단위로 사용
    - **스택 프레임**은 함수가 호출될 때 생성되고, 반환 될 때 해제

**예시**

```c
void func() {
  int choice = 0;

  scanf("%d", &choice);

  if (choice)
    call_true();
  else
    call_false();

  return 0;
}
```

- 사용자의 선택에 따라 **호출되는 함수가 다름**
    
    **→** *미리 계산이 일반적으로 불가능*
    
    → 그래서 운영체제는 프로세스를 시작할 때 **작은 크기의 스택 세그먼트를**
         **먼저 할당**해주고, **부족해 질 때마다 이를 확장**
    
    - *스택은 낮은 주소로 확장됨*

<br>
*

### 힙 세그먼트

- **힙 데이터가 위치**하는 세그먼트.
- **동적으로 할당**될 수 있으며, 리눅스에서는 스택 세그먼트와 **반대 방향**으로 자람.
- C언어에서 `malloc()`, `calloc()` 등을 호출해서 할당 받는 메모리가 이 세그먼트에 위치
    - **부여 권한** : 읽기, 쓰기

**예시**

```c
int main() {
  int *heap_data_ptr =
      malloc(sizeof(*heap_data_ptr));  // 동적 할당한 힙 영역의 주소를 가리킴
  *heap_data_ptr = 31337;              // 힙 영역에 값을 씀
  printf("%d\n", *heap_data_ptr);  // 힙 영역의 값을 사용함
  return 0;
}
```

- `heap_data_ptr`에 `malloc()`으로 **동적 할당한 영역의 주소를 대입**하고, 
이 영역에 값을 쓴다.
- `heap_data_ptr`은 지역변수이므로 스택에 위치하며, `malloc`으로 **할당 받은 힙 세그먼트의 주소**를 가리킴.

**

---

### x64 어셈블리 언어

- 동사에 해당하는 **명령어(Operation Code, Opcode)**와 목적어에 해당하는 **피연산자(Operand)**로 구성
    
    
    **x86-64 어셈블리어 문법 구조**
    

![image.png](attachment:209a4920-5de5-4eff-8180-17f0cb028b1e:image.png)

### 명령어

---

- 매우 많은 명령어가 존재. 이중 중요한 21개의 명령어를 자세히 학습할 예정.

**명령 코드 예시**

| **명령 코드** |  |
| --- | --- |
| **데이터 이동(Data Transfer)** | `mov`, `lea` |
| **산술 연산(Arithmetic)** | `inc`, `dec`, `add`, `sub` |
| **논리 연산(Logical)** | `and`, `or`, `xor`, `not` |
| **비교(Comparison)** | `cmp`, `test` |
| **분기(Branch)** | `jmp`, `je`, `jg` |
| **스택(Stack)** | `push`, `pop` |
| **프로시져(Procedure)** | `call`, `ret`, `leave` |
| **시스템 콜(System call)** | `syscall` |

### 1. 데이터 이동

---

- 어떤 값을 **레지스터나 메모리에 옮기도록** 지시.
- **명령어 :** `mov` , `lea`

| **mov dst, src : src에 들어있는 값을 dst에 대입** |  |
| --- | --- |
| mov rdi, rsi | rsi의 값을 rdi에 대입 |
| mov QWORD PTR[rdi], rsi | rsi의 값을 rdi가 가리키는 주소에 대입 |
| mov QWORD PTR[rdi+8*rcx], rsi | rsi의 값을 rdi+8*rcx가 가리키는 주소에 대입 |

| **lea dst, src : src의 유효 주소(Effective Address, EA)를 dst에 저장합니다** |  |
| --- | --- |
| lea rsi, [rbx+8*rcx] | rbx+8*rcx 를 rsi에 대입 |

### 2. 산술 연산

---

- **예시** : `inc` ,`dec`, `add`, `sub`

| **add dst, src : dst에 src의 값을 더합니다.** |  |
| --- | --- |
| **add eax, 3** | eax += 3 |
| **add ax, WORD PTR[rdi]** | ax += *(WORD *)rdi |

| **sub dst, src: dst에서 src의 값을 뺍니다.** |  |
| --- | --- |
| **sub eax, 3** | eax -= 3 |
| **sub ax, WORD PTR[rdi]** | ax -= *(WORD *)rdi |

| **inc op: op의 값을 1 증가시킴** |  |
| --- | --- |
| **inc eax** | eax += 1 |

| **dec op: op의 값을 1 감소 시킴** |  |
| --- | --- |
| **dec eax** | eax -= 1 |

### 3-1. 논리 연산 - and & or

---

- 논리 연산은 비트 단위로 이루어짐
- **예시** : `and`, `or`

**and dst, src: dst와 src의 비트가 모두 1이면 1, 아니면 0**

```nasm
[Register]
eax = 0xffff0000
ebx = 0xcafebabe

[Code]
and eax, ebx

[Result]
eax = 0xcafe0000
```



**or dst, src: dst와 src의 비트 중 하나라도 1이면 1, 아니면 0**

```nasm
[Register]
eax = 0xffff0000
ebx = 0xcafebabe

[Code]
or eax, ebx

[Result]
eax = 0xffffbabe
```

### 3-2. 논리 연산 xor & not

---

- **예시** :  `xor`, `not`

**xor dst, src: dst와 src의 비트가 서로 다르면 1, 같으면 0**

```nasm
[Register]
eax = 0xffffffff
ebx = 0xcafebabe

[Code]
xor eax, ebx

[Result]
eax = 0x35014541
```

**not op: op의 비트 전부 반전**

```nasm
[Register]
eax = 0xffffffff

[Code]
not eax

[Result]
eax = 0x00000000
```

### 4. 비교

---

- 두 피연산자의 **값을 비교하고 플래그를 설정**
- **예시** : `cmp`, `test`

**cmp op1, op2: op1과 op2를 비교**

*두 피연산자를 빼서 대소를 비교. 연산 결과는 op1에 대입하지 않음*

```nasm
[Code]
1: mov rax, 0xA
2: mov rbx, 0xA
3: cmp rax, rbx ; ZF=1
```

**test op1, op2: op1과 op2를 비교**

*두 피연산자에 AND 비트 연산. 연산 결과는 op1에 대입하지 않음*

```nasm
[Code]
1: xor rax, rax
2: test rax, rax ; ZF=1
```

- 0이 된 `rax` **를** **op1과 op2로 삼아** `test`를 수행. 
→ 결과가 0이므로 `ZF` **플래그**가 설정됨. 
→ 이후 CPU는 **플래그**를 보고 ****`rax`가 0이었는지 판단.

### 5. 분기

---

- 분기 명령어는 `rip`를 **이동시켜 실행 흐름을 바꿈!**
- **예시** : `jmp`, `je`, `jg`

**jmp addr: addr로 rip를 이동시킵니다.**

```nasm
[Code]
1: xor rax, rax
2: jmp 1 ; jump to 1
```


**je addr: 직전에 비교한 두 피연산자가 같으면 점프 (jump if equal)**

```nasm
[Code]
1: mov rax, 0xcafebabe
2: mov rbx, 0xcafebabe
3: cmp rax, rbx ; rax == rbx
4: je 1 ; jump to 1
```


**jg addr: 직전에 비교한 두 연산자 중 전자가 더 크면 점프 (jump if greater)**

```nasm
[Code]
1: mov rax, 0x31337
2: mov rbx, 0x13337
3: cmp rax, rbx ; rax > rbx
4: jg 1  ; jump to 1
```

** 스택은 다음 챕터에 

### 피연산자

---

- 피연산자 3가지 종류
    - **상수**
    - **레지스터**
    - **메모리**

**메모리 피연산자 예시**

| **메모리 피연산자** |  |
| --- | --- |
| **QWORD PTR [0x8048000]** | 0x8048000의 데이터를 8바이트만큼 참조 |
| **DWORD PTR [0x8048000]** | 0x8048000의 데이터를 4바이트만큼 참조 |
| **WORD PTR [rax]** | rax가 가르키는 주소에서 데이터를 2바이트 만큼 참조 |






### 함수

- 프로그램이 **처리해야 할 명령어**들을 한 덩어리로 모아 놓은 **코드 블록**
    - **서브루틴(Subroutine), 프로시저(Procedure), 함수(Function)** 용어로 혼용
- **함수를 호출한 함수(Caller)**는 `call` 명령어로 함수를 불러서 사용
- **호출된 함수(Callee)**는 `ret` 명령어를 이용해서 다시 이전 함수에서 실행 중이던 코드로 돌아감

### 6. 스택 관련 명령어

---

> **스택(Stack)이란?**
> 

> 스택은 자료 구조의 한 종류로, **LIFO (Last In, First Out, 후입선출) 방식**으로 동작하는 구조입니다. 즉, **가장 나중에 삽입된 데이터가 가장 먼저 꺼내지는 구조**를 가지고 있습니다. 이를 쉽게 이해하기 위해 **접시를 쌓아 올리는 방식**을 떠올려볼 수 있습니다. 새로운 접시는 맨 위에 쌓이고, 꺼낼 때도 맨 위에 있는 접시부터 꺼내야 합니다.
> 

- 스택 자료구조에 따라 **데이터를 임시로 저장하고 관리**하는 데 사용하는 메모리 영역
    - **특히 함수 호출 및 복귀, 인자 전달, 함수의 지역 변수 저장** 등에 활용
- **데이터를 추가할 때마다 메모리 주소가 감소!**
- **데이터를 제거하면 다시 증가!**

**스택 포인터(Stack Pointer) 레지스터 :**  `rsp` 

▪️`push` 명령어를 실행   →   `rsp`가 **감소하면서 스택에 값이 저장**

▪️`pop`명령어 실행          →   `rsp`가 **증가하면서 스택에서 값을 꺼낼 수 있음**

**push : 스택에 값을 저장**

→ `rsp`가 감소하면서 스택에 값이 저장.

→ **형태 : `push val` - 값 `val`을 스택에 저장함**

**내부적으로 수행되는 연산**

```c
rsp -= 8
[rsp] = val
```

**예제**

```nasm
[Register]
rsp = 0x7fffffffc400

[Stack]
0x7fffffffc400 | 0x0  <- rsp
0x7fffffffc408 | 0x0

[Code]
push 0x31337
```

[](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjkiIGhlaWdodD0iMzQiIHZpZXdCb3g9IjAgMCAyOSAzNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTIxIDAuNUgzQzEuMzUgMC41IDAgMS44NSAwIDMuNVYyNC41SDNWMy41SDIxVjAuNVpNMjUuNSA2LjVIOUM3LjM1IDYuNSA2IDcuODUgNiA5LjVWMzAuNUM2IDMyLjE1IDcuMzUgMzMuNSA5IDMzLjVIMjUuNUMyNy4xNSAzMy41IDI4LjUgMzIuMTUgMjguNSAzMC41VjkuNUMyOC41IDcuODUgMjcuMTUgNi41IDI1LjUgNi41Wk0yNS41IDMwLjVIOVY5LjVIMjUuNVYzMC41WiIgZmlsbD0iIzFBMUExQiIvPgo8L3N2Zz4K)

**결과**

```nasm
[Register]
rsp = 0x7fffffffc3f8

[Stack]
0x7fffffffc3f8 | 0x31337 <- rsp 
0x7fffffffc400 | 0x0
0x7fffffffc408 | 0x0
```

**pop: 스택에서 값을 꺼냄**

→ `rsp`가 증가하면서 스택에 저장되어 있던 값을 꺼낼 수 있게 됨.

→ **형태: `pop reg` - 스택 최상단에 저장된 값을 꺼내서 레지스터인 `reg`에 대입**

**내부적으로 수행되는 연산**

```c
reg = [rsp]
rsp += 8
```

**예제**

```nasm
[Register]
rax = 0
rsp = 0x7fffffffc3f8

[Stack]
0x7fffffffc3f8 | 0x31337 <- rsp 
0x7fffffffc400 | 0x0
0x7fffffffc408 | 0x0

[Code]
pop rax
```

**결과**

```nasm
[Register]
rax = 0x31337
rsp = 0x7fffffffc400

[Stack]
0x7fffffffc400 | 0x0 <- rsp 
0x7fffffffc408 | 0x0
```

### 7. 함수 호출 및 반환 명령어

---

- 함수 호출 시엔, 함수를 실행하고 나서 원래의 실행 흐름으로 돌아와야 함.
    - `call` **다음의 명령어 주소(반환 주소)**를 스택에 저장
    → `rip` **이동**
- **예시** : `call`, `leave`, `ret`

**`call` : 함수를 호출**

**형태:** `call addr` - 주소 값 `addr`에 위치한 프로시져 호출

```nasm
push return_address  ; return address는 함수가 끝난 후 돌아갈 코드의 주소 값을 말함
jmp addr
```

**예제**

```nasm
[Register]
rip = 0x400000
rsp = 0x7fffffffc400 

[Stack]
0x7fffffffc3f8 | 0x0
0x7fffffffc400 | 0x0 <- rsp

[Code]
0x400000 | call 0x401000  <- rip
0x400005 | mov esi, eax
...
0x401000 | push rbp
```

**결과**

```nasm
[Register]
rip = 0x401000
rsp = 0x7fffffffc3f8

[Stack]
0x7fffffffc3f8 | 0x400005  <- rsp
0x7fffffffc400 | 0x0

[Code]
0x400000 | call 0x401000
0x400005 | mov esi, eax
...
0x401000 | push rbp  <- rip
```

- `rsp` 위치 : `0x400005`저장
- `rip` 레지스터 → `0x401000`으로 바뀜

**`leave`** :프로시저가 반환되기 전, **스택 프레임을 정리**

**형태:** `leave` - 스택프레임 정리

```nasm
mov rsp, rbp
pop rbp
```

**예제**

```nasm
[Register]
rsp = 0x7fffffffc400
rbp = 0x7fffffffc480

[Stack]
0x7fffffffc400 | 0x0 <- rsp
...
0x7fffffffc480 | 0x7fffffffc500 <- rbp
0x7fffffffc488 | 0x31337 

[Code]
leave
```

**결과**

```nasm
[Register]
rsp = 0x7fffffffc488
rbp = 0x7fffffffc500

[Stack]
0x7fffffffc400 | 0x0
...
0x7fffffffc480 | 0x7fffffffc500
0x7fffffffc488 | 0x31337 <- rsp
...
0x7fffffffc500 | 0x7fffffffc550 <- rbp
```

**`ret`** : 함수에서 반환하여 원래 실행하던 코드로 **돌아오는 명령어**

**형태:** `ret` **-** rip 레지스터의 값을 **return address로 변경**

```nasm
pop rip
```

**예제**

```nasm
[Register]
rip = 0x401008
rsp = 0x7fffffffc3f8

[Stack]
0x7fffffffc3f8 | 0x400005    <- rsp
0x7fffffffc400 | 0

[Code]
0x400000 | call 0x401000
0x400005 | mov esi, eax
...
0x401000 | mov rbp, rsp  
...
0x401007 | leave
0x401008 | ret  <- rip
```

**결과**

```nasm
[Register]
rip = 0x400005
rsp = 0x7fffffffc400

[Stack]
0x7fffffffc3f8 | 0x400005
0x7fffffffc400 | 0x0    <- rsp

[Code]
0x400000 | call 0x401000
0x400005 | mov esi, eax   <- rip
...
0x401000 | mov rbp, rsp  
...
0x401007 | leave
0x401008 | ret
```

### Opcode: 시스템 콜

---

- 현대 운영체제는 **사용자 모드**와 **커널모드**로 나뉘어 동작함
    - **내부 데이터를 보호**하기 위해 사용자와 커널을 구분

- **커널 모드**
    - 모든 메모리 영역에 접근 가능
    - 하드웨어에 직접적인 접근 가능
    - 파일시스템, I/O, 네트워크 통신, 메모리 관리 등 모든 저수준 작업이 커널모드에서 진행됨
    - 시스템의 모든 부분을 제어할 수 있음
- **사용자 모드**
    - 운영체제가 사용자에게 부여하는 권한
    - 접근 가능한 메모리 영역과 권한이 매우 한정적임
    - 하드웨어에 직접적인 접근 x

- **시스템 콜**
    - **유저 모드**에서 **커널모드**의 시스템 소프트웨어에게 **동작을 요청**하기 위해 사용
    - **예시 :**
        - `cat flag` 명령어 실행을 하려고 함
        - `flag`파일 읽기/출력이 필요
        - `flag`는 파일 시스템에 존재 ← 파일 시스템에 접근할 수 있어야함
        - **유저 모드**는 소프트웨어에 도움을 요청해야함
            
            → = **시스템 콜** : **도움이 필요하다는 요청**
            

### 8. 시스템 콜 사용

---

**x86**

```nasm
**int 0x80**
번호: eax

인자 순서: ebx -> ecx -> edx -> esi -> edi -> ebp → …
```

- x86에서는 `int 0x80` 명령어를 통해 시스템 콜을 함
- `eax` 레지스터에 호출하고자 하는 시스템 콜의 번호(각 기능에 고유하게 할당된 값)를 넣음
- 나머지 인자들은 `ebx`, `ecx`, `edx`, `esi`, `edi`, `ebp` 등으로 순서대로 전달 됨
- 그 후 시스템 콜의 반환값은 `eax`레지스터에 저장 됨

**예시** : `open`시스템 콜 호출, “dreamhacck.txt” 파일 열기

```nasm
section .data
    filename db "dreamhack.txt", 0
    buffer   times 100 db 0

section .text
    global _start

_start:
    mov eax, 5           
    mov ebx, filename    
    mov ecx, 0          
    int 0x80
```

**x64**

```nasm
**syscall**
요청하는 시스템 콜 번호: rax

인자 순서: rdi -> rsi -> rdx -> rcx -> r8 -> r9 -> 스택
```

- x64에서는 `syscall` 명령어를 통해 시스템 콜을 함
- `rax` 레지스터에 호출하고자 하는 시스템 콜의 번호(각 기능에 고유하게 할당된 값)를 넣음
- 나머지 인자들은 `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9` 등으로 순서대로 전달 됨
- 그 후 시스템 콜의 반환값은 `rax`레지스터에 저장 됨

```nasm
section .data
    filename db "dreamhack.txt", 0
    buffer   times 100 db 0

section .text
    global _start

_start:
    ; 파일 열기: open("dreamhack.txt", O_RDONLY, 0)
    mov rax, 2           
    lea rdi, [filename]
    mov rsi, 0           
    xor rdx, rdx      
    syscall
```

**예제**

```nasm
[Register]
rax = 0x1   
rdi = 0x1   
rsi = 0x401000  
rdx = 0xb   

[Memory]
0x401000 | "Hello Wo"   
0x401008 | "rld"    

[Code]  
syscall 
```

**결과**

```nasm
Hello World
```

- `rax`가 0x1일 때, 커널에 `write` 시스템 콜을 요청함.
- 인자를 담는 레지스터인 `rdi`, `rsi`, `rd`가 각각 `0x1`, `0x401000`, `0xb`이므로 
커널은 `write(0x1, 0x401000, 0xb)`를 수행하게 됨.
- `write` 시스템 콜 각 인자는 순서대로 출력 스트림, 출력 버퍼, 출력 길이를 나타냄.
- `0x1`은 `stdout`이며, 이는 일반적으로 표준 출력을 의미
- `0x401000`는 `“Hello World”`라는 문자열이 저장된 메모리의 주소 값이고
출력 길이는 `0xb`로 지정되어 있음
- 터미널 화면에 `“Hello World”`를 출력

### **x64 시스템 콜 테이블**

---

| **syscall** | **rax** | **arg0 (rdi)** | **arg1 (rsi)** | **arg2 (rdx)** |
| --- | --- | --- | --- | --- |
| read | 0x00 | unsigned int fd | char *buf | size_t count |
| write | 0x01 | unsigned int fd | const char *buf | size_t count |
| open | 0x02 | const char *filename | int flags | umode_t mode |
| close | 0x03 | unsigned int fd |  |  |
| mprotect | 0x0a | unsigned long start | size_t len | unsigned long prot |
| connect | 0x2a | int sockfd | struct sockaddr * addr | int addrlen |
| execve | 0x3b | const char *filename | const char *const *argv | const char *const *envp |
