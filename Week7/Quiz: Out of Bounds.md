# Quiz: Out of Bounds

**Q1. OOB 취약점을 방어하기 위해 [A] 위치에 들어갈 올바른 검증 코드는?**

```c
#include <stdio.h>
int main() {
  int buf[0x10];
  unsigned int index;
  
  scanf("%d", &index);
  [A]
  printf("%d\n", buf[index]);
  return 0;
}
```

**정답** : **`if (index >= 0x10) {exit(-1);}`**
