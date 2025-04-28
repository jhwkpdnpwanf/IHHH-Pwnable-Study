# **Mitigation: NX & ASLR**

### NX (No-eXecute)

- 실행에 사용되는 메모리 영역과 쓰기에 사용되는 메모리 영역을 분리하는 보호 기법
    - NX가 적용되면 각 메모리 영역에 필요한 권한만 부여 받음
- NX가 적용된 파일
    
    →  `0x7ffffffde000     0x7ffffffff000 rw-p    21000      0 [stack]`
    
- NX가 적용되지 않은 파일
    
    → `0x7ffffffde000     0x7ffffffff000 rwxp    21000      0 [stack]`
    
- checksec 으로도 NX 적용을 확인할 수 있음

> **5.4.0 미만 버전 리눅스 커널에서 NX**
> 

> 스택 영역 뿐만 아니라 힙, 데이터 영역 등 읽기 권한이 있는 모든 페이지에 실행 권한을 부여함. 
프로세스의 Personality에 읽기 권한이 있는 모든 페이지에 실행 권한을 부여하는 `READ_IMPLIES_EXEC` 플래그를 설정하기 때문
`READ_IMPLIES_EXEC` 설정 없이, 로더가 따로 스택 영역에만 실행 권한을 부여함
>

<br>

<img src="https://github.com/user-attachments/assets/2bb8260b-b455-4372-9ac6-d067b4428665" alt="Image" width="350">

<img src="https://github.com/user-attachments/assets/21ea03f3-97f0-4c8b-a641-fe6f84b903ff" alt="Image" width="350">

이렇게 'checksec'을 이용해서 nx 적용유무를 확인할 수 있음  

참고로 이렇게 nx가 적용된 코드에 익스플로잇 코드를 실행시키면  
스택영역에 실행권한이 없으니까 실행되지 않고 그냥 꺼진다  

<br>

### ASLR(Address Space Layout Randomization)

- 바이너리가 실행될 때 마다 스택, 힙, 공유 라이브러리 등을 임의의 주소에 할당하는 보호 기법
- 이전에 r2s 를 풀 때 buf 주소를 먼저 읽고 풀었었음 (그냥 출력됐음)
    - 만약 일반적인 바이너리였다면 buf 주소를 구했어야 함
    - r2s 를 풀 때 로컬에서는 aslr을 끄고 buf 주소를 읽었었는데, 그렇게 푸니까 실습 서버에서는 안풀렸었음
    - 그래서 출력되는 buf를 읽고 값을 바꾸니까 됐었음

<br>

**ALSR 확인하는 법**

```bash
$ cat /proc/sys/kernel/randomize_va_space
2
```

![image](https://github.com/user-attachments/assets/6e541006-59a7-49ca-97c3-6aab991784fc)

리눅스에서 0,1,2 값 중 하나를 가짐

- No ASLR(0) :  ASLR 적용 X
- Conservation Randomization(1) : 스택, 라이브러리, vdso 등
- Conservation Randomization(2) : (1)의 영역과 `brk`로 할당한 영역


<br>

**ASLR 예제 코드**

```c
// Name: addr.c
// Compile: gcc addr.c -o addr -ldl -no-pie -fno-PIE

#include <dlfcn.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
  char buf_stack[0x10];                   // 스택 버퍼
  char *buf_heap = (char *)malloc(0x10);  // 힙 버퍼

  printf("buf_stack addr: %p\n", buf_stack);
  printf("buf_heap addr: %p\n", buf_heap);
  printf("libc_base addr: %p\n",
         *(void **)dlopen("libc.so.6", RTLD_LAZY));  // 라이브러리 주소

  printf("printf addr: %p\n",
         dlsym(dlopen("libc.so.6", RTLD_LAZY),
               "printf"));  // 라이브러리 함수의 주소
  printf("main addr: %p\n", main);  // 코드 영역의 함수 주소
}
```

### ASLR 특징  
(addr.c 코드는 메모리 주소를 출력하는 코드)  
![image](https://github.com/user-attachments/assets/bd205a06-89f1-43f7-bce3-feb664245dcd)

스택 영역의 `buf_stack`,  
힙영역의 `buf_heap`,  
라이브러리 함수 `printf`,   
코드 영역의 함수 `main`,   
라이브러리 매핑 주소 `libc_base` 가 출력 됨  

**특징**  
- `main` 함수를 제외하고는 실행할 때 마다 변경된다.
- `libc_base` 와 `printf` 의 하위 12비트는 항상 같다. (라틀엔디안이니까 젤 마지막 숫자 3개)
    - 리눅스는 ASLR이 적용되면 파일을 페이지 단위로 임위 주소에 매핑함.   
      페이지의 크기는 12바이트라 12바이트 이하로는 주소가 변경되지 않음
- `libc_base` 와 `printf` 의 차이는 항상 같다.
    - 매핑된 주소로부터 라이브러리의 다른 심볼들 까지 offset은 동일함

