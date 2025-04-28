# Background: Library  - Static Link vs. Dynamic Link

### 라이브러리  
- C언어를 비롯하여 많은 컴파일 언어들은 자주 사용하는 함수들의 정의를 묶어서 하나의 라이브러리 파일로 만들고, 이를 여러 프로그램이 공유해서 사용할 수 있도록 지원한다
    - `printf`, `scanf`, `strlen`, `memcpy`, `malloc`
- 각 언어에서 범용적으로 많이 사용되는 함수들은 표준 라이브러리가 제작되어 있음
    - C 표준 라이브러리 : `libc`
<br>

**링크**  
- 컴파일의 마지막 단계, 호출된 함수랑 실제 라이브러리 함수가 연결
- 리눅스에서 C 소스 코드는 전처리, 컴파일, 어셈블 과정을 거쳐 ELF 형식을 갖춘 
오브젝트 파일(Object file)로 번역된다.
![image](https://github.com/user-attachments/assets/506627f1-3e56-4043-9940-88c9d6603d52)

libc를 같이 컴파일하지 않았음에도 libc에서 해당 심볼을 탐색한 것은,  
libc가 있는 `/libc/x86_64-linux-gnu/`가 표준 라이브러리 경로에 포함되어 있기 때문이다.  

<br>

**표준 라이브러리 경로 확인 방법**     
![image](https://github.com/user-attachments/assets/864f0bc7-aece-49dc-83e5-d3ab69bfb6bb)

```bash
ld --verbose | grep SEARCH_DIR | tr -s ' ;' '\n'
```
링크를 거치고 나면 프로그램에서 puts을 호출할 때,   

puts 정의가 있는 libc에서 puts의 코드를 찾고 해당 코드를 실행하게 된다

<br>

**라이브러리 링크와 종류**  
- **동적 링크 (Dynamic Link)** : 동적 라이브러리를 링크
    - 동적 링크된 바이너리를 실행 → 동적 라이브러리가 프로세스의 메모리에 매핑 
    & 실행 중에 라이브러리 함수를 호출 → 매핑된 라이브러리에서 호출할 함수의 주소 탐색, 실행
- **정적 링크 (Static Link)** : 정적 라이브러리를 링크
    - 바이너리에 정적 라이브러리의 필요한 모든 함수가 포함
    - 함수 호출 → 자신의 함수를 호출하듯이 호출 (라이브러리에서 찾는 거 아님)
    - 여러 바이너리에서 사용 → 여러번 복제, 용량 낭비
    - cf) 컴파일 옵션에 따라 include 한 헤더의 함수가 모두 포함될 수도, 아닐 수도 있음

<br>
<br>

### 동적 링크 vs. 정적 링크

비교를 위해 두 가지로 컴파일해준다

```bash
$ gcc -o static hello-world.c -static
$ gcc -o dynamic hello-world.c -no-pie
```

![image](https://github.com/user-attachments/assets/64cb79d5-a4de-49c4-8ca4-86dcddc63ef5)

**용량**

용량을 비교해보면 `static` 이 약 50배 용량을 많이 차지한다.  
static  
![image](https://github.com/user-attachments/assets/f9f65196-3be7-4bc2-a362-3daffd568a72)

dynamic  
![image](https://github.com/user-attachments/assets/34bd44d6-3d0c-43d3-b563-b79c74194eab)

static은 <puts>이 있는 0x404d30을 직접 호출하지만  
dynamic은 <puts>의 pit 주소인 0x401040을 호출한다.  

<br>

### PLT(Produce Linking Table) & **GOT(Global Offset Table)**

- **plt(Produce Linking Table)** 과 **GOT(Global Offset Table)**는 라이브러리에서 동적 링크된 심볼의 주소를 찾을 때 사용하는 테이블
    - **runtime resolve 과정**
    - 바이너리가 실행되면 ASLR에 의해 라이브러리가 임의의 주소에 매핑됨
    - 이 상태에서 라이브러리를 호출하면, 함수의 이름을 바탕으로 라이브러리에서 심볼들을 탐색
    - 해당 함수의 정의를 발견하면, 그 주소로 실행 흐름을 옮김
    - 반복되는 호출되는 함수는 GOT 테이블에 저장

**got 예제 코드**

```c
// Name: got.c
// Compile: gcc -o got got.c -no-pie

#include <stdio.h>

int main() {
  puts("Resolving address of 'puts'.");
  puts("Get address from GOT");
}
```  

**resolve 되기 전**
![image](https://github.com/user-attachments/assets/b184b5fd-51b3-4414-9228-ef7ac5d51ff8)

**Global Offset Table**의 상태를 보기 위해 `got` 명령어를 사용해보면,    
아직 아무일도 일어나지 않는다.   
(Section .plt 0x401020-0x401040 인 .plt 섹션 어딘가의 주소0x401030이 적혀있음)  

이제 main에서 `puts@plt`를 호출하는 지점에 브레이크를 걸고 내부에 들어가보자  
![image](https://github.com/user-attachments/assets/81f235a0-e4c0-4051-bd90-57cb36f9cc44)

들어가서 보면  
뒤에 <_dl_runtime_resolve_fxsave> 함수가 실행되는게 보인다.   
이 함수가 puts 주소를 구하고 GOT 엔트리에 주소를 쓰는 거라고 한다

저 함수 안에 들어가서 빠져나와보면  
![image](https://github.com/user-attachments/assets/c8d02827-e530-492f-bc96-e947336ddd80)

실제로 puts 의 GOT 엔트리에 libc 영역 내 실제 puts 주소인 0x7ffff7e2abe0 이 쓰여져 있는 모습을 확인할 수 있다.

<br>

**resolve 된 후**
![image](https://github.com/user-attachments/assets/212bc141-810f-41c5-8ed8-6703713627b6)

![image](https://github.com/user-attachments/assets/9d935860-c246-4255-8b3b-19f5e5d5be48)

호출할 때 브레이크를 걸고 두 번째 호출을 보면  
GOT 엔트리에 실제 puts 주소가 들어간 것을 볼 수 있다.   

<br>

### 시스템 해캉 관점에서 본 PLT와 GOT  
- GOT의 값을 검증하지 않는다면 취약점이 발생 가능
- 위 예시에서 puts 의 GOT 엔트리를 변경할 수 있다면, 원하는 코드를 실행되게 할 수 있음

  <br>
  
![image](https://github.com/user-attachments/assets/054a7f88-f1d3-4dd6-a2d2-ef5994e81946)

이렇게 실제로 GOT 를  수정하여 이렇게 실행 흐름을 바꿀 수 있다.  
