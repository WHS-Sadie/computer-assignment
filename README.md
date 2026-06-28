# computer-assignment
---

## 과제 1. 깃허브 코드스페이스로 리눅스 개발환경 만들기

GitHub에서 `computer-assignment` 레포지토리를 생성하고 Codespace를 실행했다.  
GCC 컴파일러가 정상적으로 설치되어 있음을 확인했다.

![](<과제 1.png>)

---

## 과제 2. 오버플로 예제를 언더플로로 바꿔서 해보기

char 타입은 저장할 수 있는 값의 범위가 -128부터 127까지이다. 이때 가장 작은 값인 -128에서 1을 더 빼면 더 이상 저장할 수 없어서 값이 가장 큰 127로 바뀐다. 이것을 언더플로(Underflow)라고 한다.

#### 추가설명
char가 저장할 수 있는 범위: -128 ~ 127
현재 값: -128
여기서 1을 더 빼면 → -129가 되어야 하는데, -129는 저장할 수 없기 때문에 127로 돌아간다.

Char는 **8비트(1바이트)**로 값을 저장함.
즉, 저장할 수 있는 경우의 수가 **256개(2⁸)**뿐!

부호 있는 char는 이 256개를 이렇게 사용한다.

-128 ~ -1
0 ~ 127

즉, 가장 작은 값은 -128, 가장 큰 값은 127임.

이걸 원처럼 생각하면 쉽다!
... → 125 → 126 → 127
                    ↓
-128 ← -127 ← -126 ← ...

127에서 1을 더하면 → 저장 공간이 끝났으니까 -128로 돌아감. (오버플로)
-128에서 1을 빼면 → 반대로 끝을 넘어가니까 127로 돌아감. (언더플로)

왜 127이 되냐면, 컴퓨터가 8비트 안에서만 계산하기 때문이야.
이렇게 값이 끝에서 반대쪽으로 넘어가는 것을 언더플로!

### 코드 (char_underflow.c)

```c
#include <stdio.h>
#include <limits.h>

int main() {
    char value = CHAR_MIN;
    printf("Original value: %d\n", value);
    value = value - 1;
    printf("Value after subtracting 1: %d\n", value);
    return 0;
}
```

### 실행 결과

### 설명

`char`는 8비트 부호 있는 정수(-128 ~ 127)이다.  
-128에서 1을 빼면 비트가 `10000000` → `01111111`로 바뀌어 127이 된다.

![](<과제 2.png>)

---

## 과제 3. 비트 연산 프로그램 바꿔보기

특정 위치의 비트를 끄는 `clear_bit` 함수를 구현했다.

### 코드 (bit_operation.c)

```c
#include <stdio.h>

int is_bit_set(unsigned char value, int position) {
    return (value & (1 << position)) != 0;
}

unsigned char set_bit(unsigned char value, int position) {
    return value | (1 << position);
}

unsigned char clear_bit(unsigned char value, int position) {
    return value & ~(1 << position);
}

int main() {
    unsigned char value = 0b00001000;

    if (is_bit_set(value, 3)) printf("3rd bit is set!\n");
    else printf("3rd bit is not set!\n");

    value = set_bit(value, 2);
    printf("Value after setting 2nd bit: %d\n", value);

    value = clear_bit(value, 3);
    printf("Value after clearing 3rd bit: %d\n", value);

    return 0;
}
```

### 실행 결과

### 설명

- `is_bit_set`: AND 연산으로 특정 비트 확인
- `set_bit`: OR 연산으로 특정 비트를 1로 설정
- `clear_bit`: NOT + AND 연산으로 특정 비트를 0으로 해제

![](<과제 3.png>)

### 추가. 사용자 입력 받도록 수정

값과 비트 위치를 직접 입력받도록 코드를 수정했다.

```c
int main() {
    unsigned char value;
    int input_value, position;

    printf("값을 입력하세요 (정수): ");
    scanf("%d", &input_value);
    value = (unsigned char)input_value;

    printf("비트 위치를 입력하세요 (0-7): ");
    scanf("%d", &position);

    printf("현재 값: %d\n", value);

    if (is_bit_set(value, position)) {
        printf("%d번째 비트가 설정되어 있습니다!\n", position);
    } else {
        printf("%d번째 비트가 설정되어 있지 않습니다!\n", position);
    }

    value = set_bit(value, position);
    printf("%d번째 비트 설정 후 값: %d\n", position, value);

    value = clear_bit(value, position);
    printf("%d번째 비트 해제 후 값: %d\n", position, value);

    return 0;
}
```

### 실행 결과 (입력: 값=4, 위치=3)

![](<추가.png>)

---

## 과제 4. C언어가 기계어가 되는 과정

`hello_world.c`를 단계별로 컴파일해 각 중간 파일을 확인했다.

### 소스 코드 (hello_world.c)

```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```

### 컴파일 파이프라인

| 단계 | 명령어 | 출력 파일 | 설명 |
|------|--------|-----------|------|
| 전처리 | `gcc -E hello_world.c -o hello_world.i` | hello_world.i | 헤더 파일 포함, 매크로 치환 |
| 컴파일 | `gcc -S hello_world.i -o hello_world.s` | hello_world.s | 어셈블리 코드 생성 |
| 어셈블 | `gcc -c hello_world.s -o hello_world.o` | hello_world.o | 오브젝트 파일 생성 |
| 링킹 | `gcc hello_world.o -o hello_world` | hello_world | 실행 파일 생성 |

### 실행 결과

### 파일 크기 비교

| 파일 | 크기 | 설명 |
|------|------|------|
| hello_world.c | 79 bytes | 소스 코드 |
| hello_world.i | 21KB | 전처리 후 (헤더 포함) |
| hello_world.s | 674 bytes | 어셈블리 코드 |
| hello_world.o | 1.5KB | 오브젝트 파일 |
| hello_world | 16KB | 실행 파일 |

![](<과제 4.png>)
