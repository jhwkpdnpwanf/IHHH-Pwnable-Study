# **Memory Corruption: Out of Bounds**

### OOB(Out-of-Bounds)  
- 메모리 버퍼(배열, 스택)의 경계를 벗어나서 읽거나 쓰는 모든 동작이다.
<br>

### 배열의 속성  
- 배열은 연속된 메모리 공간을 점유한다.
- 점유하는 공간의 크기 :
    - `요소의 개수(배열 길이)` * `요소 자료형의 크기`
- 배열 각 요소의 주소 :
    - `배열의 주소` + `요소의 인덱스` * `요소 자료형의 크기`  
<br>

### **Out of Bounds**  
- 인덱스 값이 음수거나 배열의 길이를 벗어날 때 발생한다.
- 인덱스 범위 검사를 하지 않는다 = 계산한 주소가 배열 범위 안에 있는지 검사하지 않는다.
    - 인덱스 값을 임의 값으로 바꿔서, 배열 범위를 벗어나는 참조를 하는 것이 가능하다.
    - 이걸 OOB라 부름.  
<br>

### **Proof-of-Concept**  
```c
// Name: oob.c
// Compile: gcc -o oob oob.c

#include <stdio.h>

int main() {
  int arr[10];

  printf("In Bound: \n");
  printf("arr: %p\n", arr);
  printf("arr[0]: %p\n\n", &arr[0]);

  printf("Out of Bounds: \n");
  printf("arr[-1]: %p\n", &arr[-1]);
  printf("arr[100]: %p\n", &arr[100]);

  return 0;
}
```
<br>

**실행 결과**  
<img src = "https://github.com/user-attachments/assets/ae82785f-fd87-4f53-87d4-79d440477a11" width=650>

`int arr[10]` 배열을 선언 했지만,  
배열의 범위가 벗어난 `arr[-1]`과 `arr[100]`의 주소가 출력됐다.  
<br>

### 임의 주소 읽기  
- OOB로 임의 주소 값을 읽기 위해, 읽으려는 변수와 배열의 오프셋을  알아야 한다.
- 만약 배열과 변수가 같은 세그먼트에 있다면:
    - 둘 사이 오프셋은 일정하므로 디버깅을 통해 알아내기
- 만약 배열과 변수가 다른 세그먼트에 있다면:
    - 다른 취약점으로 주소를 먼저 구하고, 차이를 계산
<br>

**임의 주소 읽기 예제 코드**  
```c
// Name: oob_read.c
// Compile: gcc -o oob_read oob_read.c

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

char secret[256];

int read_secret() {
  FILE *fp;

  if ((fp = fopen("secret.txt", "r")) == NULL) {
    fprintf(stderr, "`secret.txt` does not exist");
    return -1;
  }

  fgets(secret, sizeof(secret), fp);
  fclose(fp);

  return 0;
}

int main() {
  char *docs[] = {"COMPANY INFORMATION", "MEMBER LIST", "MEMBER SALARY",
                  "COMMUNITY"};
  char *secret_code = secret;
  int idx;

  // Read the secret file
  if (read_secret() != 0) {
    exit(-1);
  }

  // Exploit OOB to print the secret
  puts("What do you want to read?");
  for (int i = 0; i < 4; i++) {
    printf("%d. %s\n", i + 1, docs[i]);
  }
  printf("> ");
  scanf("%d", &idx);

  if (idx > 4) {
    printf("Detect out-of-bounds");
    exit(-1);
  }

  puts(docs[idx - 1]);
  return 0;
}
```

**secret.txt**

```c
$ echo "THIS IS SECRET" > ./secret.txt
```

secret.txt를 만들어준다.   

read_secret 호출 이후 스택 구조를 살펴보면,  

![image](https://github.com/user-attachments/assets/e04ae120-9252-4597-b6b7-0d4eb16665eb)

![image](https://github.com/user-attachments/assets/fa32797e-b212-45d4-93cc-699dc57ccc43)

배열 위에 secret이 있다는 것을 알 수 있다.   

<br>


![image](https://github.com/user-attachments/assets/95f836e1-e17f-4722-b9bc-11f83b89ffea)


그리고 `if (idx > 4)` 로 idx가 1 보다 작을 때는 검증하지 않으므로,  
idx = 0 일 때 secret.txt 를 읽게 된다.  
<br>

### **임의 주소 쓰기**  
- 인덱스 검증이 미흡하면 값을 쓰는 것도 가능하다.  
<br>

**임의 주소 쓰기 예제 코드**  
```c
// Name: oob_write.c
// Compile: gcc -o oob_write oob_write.c

#include <stdio.h>
#include <stdlib.h>

struct Student {
  long attending;
  char *name;
  long age;
};

struct Student stu[10];
int isAdmin;

int main() {
  unsigned int idx;

  // Exploit OOB to read the secret
  puts("Who is present?");
  printf("(1-10)> ");
  scanf("%u", &idx);

  stu[idx - 1].attending = 1;

  if (isAdmin) printf("Access granted.\n");
  return 0;
}
```  
코드 마지막에 `isAdmin` 이 참인지 검사하는 부분이 있다.  
직접 값을 쓰는 코드는 없지만 isAdmin의 값을 OOB를 통해 조작할 수는 있다.  
 
그러기 위해 `stu`와 `isAdmin`의 주소 차이를 계산해보면,  
![image](https://github.com/user-attachments/assets/68247781-e7e3-41f5-8a66-ac1e7c8c7b37)

240 바이트 높은 주소에 isAdmin이 있다는 것을 알 수 있다.  

<br>

![image](https://github.com/user-attachments/assets/1174b206-5fda-44e1-9382-c0529315ff31)

`Student` 구조체는 24바이트니까, 10번 째 인덱스를 참조하면 참인 결과를 얻을 수 있다.  

