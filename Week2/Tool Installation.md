# Tool: gdb
### gdb

- 리눅스의 대표적인 디버거
    - **pwngdb : https://github.com/scwuaptx/Pwngdb**

깃허브 안내에 따라 설치를 해주고,

실습을 위해 `debugee.c`파일 하나를 만들어준다.

### 실습예제
```c
// Name: debugee.c
// Compile: gcc -o debugee debugee.c -no-pie

#include <stdio.h>
int main(void) {
  int sum = 0;
  int val1 = 1;
  int val2 = 2;

  sum = val1 + val2;

  printf("1 + 2 = %d\n", sum);

  return 0;
}
```


![image](https://github.com/user-attachments/assets/faed5bd5-593c-42c5-b10e-5b44144a7f52)

`$ gcc -o debugee debugee.c -no-pie`

`$ gdb debugee`

을 치면 이렇게 화면이 뜬다.

`pwndbg`가 잘 설치 되었다면 `pwndbg>`라고 뜬다.

`entry` 명령어를 통해 진입점부터 프로그램을 분석할 수 있음.

![image (1)](https://github.com/user-attachments/assets/0fbab45a-1834-4250-97dc-7fd6eaef02d5)

$ readelf -h debugee를 입력해서 진입점을 알아보기 쉽게 출력할 수 있다.

### **entry**

- 리눅스 실행파일은 **ELF(Executable and linkable Format)을 규정**하고 있다
- ELF는 크게 **헤더와 여러 섹션들로 구성되어 있음**
- **진입점(Entry Point, EP) 필드가 존재**하는데, ****ELF 실행할 때 **진입점의 값부터** 프로그램을 실행함

→ debugee의 진입점은 `0x401050` !



### break & continue

- 일반적으로 전체 프로그램 중 일부분의 동작에만 관심이 있기 때문에
- **break**는 특정 주소에 **중단점(breakpoint)을 설정**하는 기능
- **continue**는 중단된 프로그램을 계속 실행시키는 기능

![image (2)](https://github.com/user-attachments/assets/217dfdc1-49ad-4c8a-85e9-e8eb2d303eaa)

### **run**

- 단순히 실행을 시켜줌

### **gdb 명령어 축약**

- b: break
- c: continue
- r: run
- si: step into
- ni: next instruction
- i: info
- k: kill
- pd: pdisas


### navigate

- `ni` : **next instruction**

![image (3)](https://github.com/user-attachments/assets/5dd910ab-a7b8-4296-80b7-9b812a62134d)


- 현재 위치에서 한 줄을 실행함
- 만약 `call`, `bl`, `jal`과 같은 함수 호출 명령을 만나면 **안으로 들어가지 않음**
- 다음 줄인 다음 명령어로 점프함

si : step into
![image (4)](https://github.com/user-attachments/assets/b2debfb2-96cd-4a89-a1c1-749e13e82f3c)



- 현재 위치에서 한 줄을 실행함.
- 만약 `call`, `bl`, `jal`과 같은 함수 호출 명령을 만나면 그 함수 내부로 들어감

### **finish**

- `si`로 함수 내부에 들어가서 분석을 끝냈는데, 함수 규모가 커서 원래 실행 흐름으로 돌아가기 힘들 땐 `finish`로 함수 끝 부분까지 한번에 실행할 수 있음

### examine

- 가상 메모리에 존재하는 임의 주소의 값 관찰
- 특정 주소의 원하는 길이 만큼을 원하는 형식으로 볼 수 있음

> Format letters are o(octal), x(hex), d(decimal), u(unsigned decimal), t(binary), f(float), a(address), i(instruction), c(char), s(string) and z(hex, zero padded on the left). Size letters are b(byte), h(halfword), w(word), g(giant, 8 bytes).
> 

**1. rsp부터 80바이트를 8바이트씩 hex형식으로 출력**
![image](https://github.com/user-attachments/assets/ba1ba1eb-e3a9-4555-9a27-91e7faeafbfe)

2. rip부터 5줄의 어셈블리 명령어 출력

![image](https://github.com/user-attachments/assets/df3355cd-e523-4a62-94c6-d3a907e413a2)

3. 특정 주소의 문자열 출력

![image](https://github.com/user-attachments/assets/798800fa-c3f1-4b45-a1be-aa64b6aab88a)

telescope
![image](https://github.com/user-attachments/assets/c93571db-9669-46a7-a15c-207ea79dc337)

특정 주소의 메모리 값들을 보여주고, 참조하는 주소를 재귀적으로 탐색하여 값을 보여줌


vamap (Virtual Memory Map)

![image (5)](https://github.com/user-attachments/assets/7c2b7841-5815-4652-a62a-519c00979db1)

- 가상 메모리의 레이아웃을 보여줌
- 현재 디버깅 중인 프로세스의 **메모리 영역 분포**를 보여줌
- 각 영역이 어떤 용도로 쓰이는지 한눈에 확인 가능
- **어떤 주소 범위가 어디에서 왔는지** 파악 가능

### **gdb / python argv**

```bash
pwndbg> r $(python3 -c "print('\xff' * 100)")
Starting program: /home/dreamhack/debugee2 $(python3 -c "print('\xff' * 100)")
argv[1] ÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿ
```

- `print` 함수를 통해 출력한 값을 run(`r` ) 명령어의 인자로 전달하는 명령

### **gdb / python input**

```bash
pwndbg> r $(python3 -c "print('\xff' * 100)") <<< $(python3 -c "print('dreamhack')")
Starting program: /home/dreamhack/debugee2 $(python3 -c "print('\xff' * 100)") <<< $(python3 -c "print('dreamhack')")
argv[1] ÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿÿ
Name: dreamhack
```

- `argv[1]`에 임의의 값을 전달하고, 값을 입력하는 명령어




  
