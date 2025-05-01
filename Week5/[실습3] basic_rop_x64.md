# basic_rop_x64

**실습 코드**

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void alarm_handler() {
    puts("TIME OUT");
    exit(-1);
}

void initialize() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);

    signal(SIGALRM, alarm_handler);
    alarm(30);
}

int main(int argc, char *argv[]) {
    char buf[0x40] = {};

    initialize();

    read(0, buf, 0x400);
    write(1, buf, sizeof(buf));

    return 0;
}

```

실습 코드를 쭉 보니 read에서 buf 에 오버플로우가 일어나니까  
그냥 read 하나로 system 부르고 셀까지 킬 수 있을 거 같다.  

<br>

![image](https://github.com/user-attachments/assets/74383a8a-9b6c-484d-a607-945724f69418)

적용된 보안 목록을 보니까 카나리가 없다.  
그럼 처음 read에 got 에서 read 함수 주소를 들고와야겠다.

<br>

![image](https://github.com/user-attachments/assets/7ecd8845-bc9b-446d-94cd-8f7cc6171859)

read 전에 대충 브레이크를 걸고 H 문자열을 넣어서 스택 구조를 봐준다  

<br>

![image](https://github.com/user-attachments/assets/3d152412-a7b8-406c-9580-c85f430b8609)

![image](https://github.com/user-attachments/assets/286f6232-0d55-47a8-a4ed-1e5f1d772238)

buf(64바이트) +SFP(8바이트) + 리턴주소(8바이트)  
인게 잘 보이니까 쓰레기값 72바이트 위에 바로 원하는 걸 넣으면 되겠다  

우선 got에 read 함수 주소를 알아야하니까   
덮어씌울 값 구조부터 떠올려보자   

read는 인자가 세개니까 rdi, rsi, rdx 순서대로 인자가 들어간다   

내가 원하는 건 read(0, read@got, 0x8) 이니까 각각 숫자를 넣을 수 있도록    
pop 리턴 가젯들을 찾아준다   

<br>


![image](https://github.com/user-attachments/assets/fb26f17c-2bf0-4a82-b9b0-a034e4fb9bbe)

근데   
pop rdi는 찾았고 rsi랑 rdx가 따로 있는게 없어서  
pop rsi ; pop r15 ; ret 얘를 써야겠다   
  
`pop rdi` : 0x400883  
`pop rsi ; pop r15 ; ret` : 0x400881  

<br>


![image](https://github.com/user-attachments/assets/2f36d30d-58a3-4eb6-b152-5b9d2aeb29cf)


<br>
