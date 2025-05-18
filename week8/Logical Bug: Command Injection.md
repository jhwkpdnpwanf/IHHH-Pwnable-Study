# 
### Command injenction

- 명령어를 실행해주는 함수를 잘못 사용해서 발생하는 취약점이다.
- 악의적인 데이터를 입력하여 시스템 명령어, 코드, 데이터베이스 쿼리 등으로 실행되게 하는 **Injection** 공격 기법 중, 시스템 명령어도 실행되게 하는 것을 **Command Injection** 이라 한다.

> **system 함수가 명령어를 실행하는 과정**
`system` 함수는 라이브러리 내부에서 `do_sysyem` 함수를 호출한다.
`do-system`은 “sh-c”와 `system` 함수의 인자를 결합하여 `execve` 시스템 콜을 호출한다.

<br>

**Commad injection이 발생하는 경우**

- 명령어를 실행하는 함수에 사용자가 임의 인자를 전달할 수 있을 때 발생한다.
- 사용자의 입력을 제대로 검사하지 않을 때 문제가 생길 수 있다.
    - Meta 문자
    - `$` : 쉘 환경변수
        - $ echo $PWD
        - \> /home/theori
    - `&&` : 이전 명령어 실행 후 다음 명령어 실행
        - $ echo hello && echo theori
        - \> hello
    - `;` : 명령어 구분자
        - $ echo hello ; echo theori
        - \> hello
        - \> theori
    - `|` : 명령어 파이핑
        - $ echo id | \bin\sh
        - \> uid=1001(theori) gid=1001(theori)
    - `*` : 와일드 카드
        - $ echo .*
        - \> . ..
    - \``` : 명령어 치환
        - $ echo \`echo hellotheori\`
        - \> hellotheori

- 위 메타 문자들을 고려해보면, `ping [user input]` 같은 상황에서 
`user input` 에 `a; \bin\sh` 를 전달하면 `ping a; \bin\sh` 가 명령어로 전달되어 두 명령어가 다 실행됨을 알 수 있다.

**예제**

```c
// Name: cmdi.c
// Compile: gcc -o cmdi cmdi.c

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

const int kMaxIpLen = 36;
const int kMaxCmdLen = 256;

int main() {
  char ip[kMaxIpLen];
  char cmd[kMaxCmdLen];

  // Initialize local vars
  memset(ip, '\0', kMaxIpLen);
  memset(cmd, '\0', kMaxCmdLen);
  strcpy(cmd, "ping -c 2 ");

  // Input IP
  printf("Health Check\n");
  printf("IP: ");
  fgets(ip, kMaxIpLen, stdin);

  // Construct command
  strncat(cmd, ip, kMaxCmdLen);
  printf("Execute: %s\n",cmd);

  // Do health-check
  system(cmd);

  return 0;
}
```

<br>

![image](https://github.com/user-attachments/assets/c24ebc2d-0184-4efb-9fd0-05a2b0b5990d)  
원하는 ip에 ping을 보내는 함수이다. 잘 작동이된다.  
하지만 입력검증이 없으므로 cat cmdi.c 함수를 뒤에 적어주면  

<br>

![image](https://github.com/user-attachments/assets/c2890767-4b2c-4389-9376-ba632d40ba6a)  
이렇게 원하는 코드가 실행이 되는 것을 볼 수 있다.
