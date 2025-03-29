# Quiz**: x86 Assembly 2**

**Q1. end로 점프하면 프로그램이 종료된다고 가정하자. 프로그램이 종료됐을 때, 0x400000 부터 0x400019까지의 데이터를 대응되는 아스키 문자로 변환하면 어느 문자열이 나오는가?**

```nasm
[Register]
rcx = 0
rdx = 0
rsi = 0x400000
=======================
[Memory]
0x400000 | 0x67 0x55 0x5c 0x53 0x5f 0x5d 0x55 0x10
0x400008 | 0x44 0x5f 0x10 0x51 0x43 0x43 0x55 0x5d
0x400010 | 0x52 0x5c 0x49 0x10 0x47 0x5f 0x42 0x5c
0x400018 | 0x54 0x11 0x00 0x00 0x00 0x00 0x00 0x00
=======================
[code]
1: mov dl, BYTE PTR[rsi+rcx]
2: xor dl, 0x30
3: mov BYTE PTR[rsi+rcx], dl
4: inc rcx
5: cmp rcx, 0x19
6: jg end
7: jmp 1
```

**정답**

- Welcome to assembly world!

**해설**

1. `mov dl, BYTE PTR[rsi+rcx]` → `dl` = `0x67`
2. `xor dl, 0x30` → xor 연산하면 `0101 0111` , `dl` =`0x57`
3. `mov BYTE PTR[rsi+rcx], dl` → `dl`값을 `0x400000`에 저장, `0x67` → `0x57` 
4. `inc rcx` → `rcx` = 1
5. `cmp rcx, 0x19` → 1 과 `0x19`를 비교 (1과 25)
6. `jg end` → `rcx`가 25보다 크면  분기
7. `jmp 1` → 1번 명령어로

![image](https://github.com/user-attachments/assets/afaa1039-c389-4c7a-8d03-a4628c24c043)
