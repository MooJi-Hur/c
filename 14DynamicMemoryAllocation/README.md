### Allocate and Free
```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {

    // 메모리 동적 할당
    int *allocated = malloc(sizeof(int));

    printf("%d \n", *allocated); // 0, 쓰레기 값일 수도 있음
    printf("%p \n", allocated); // 0x7fd418a04080 <- 힙에 할당된 메모리 주소
    printf("%p \n", &allocated); // 0x7ff7bfeff3a0 <- 스택에 있는 allocated 포인터 자체의 주소

    // 메모리 해제, 메모리 누수 방지를 위해 해야함
    // 스택에서는 해제를 할 필요 없으나 동적 할당되어 힙 영역에 할당된 메모리는 해제해야 함
    free(allocated);
    return 0;
}
```

### Memset
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void) {
    char *preparedA = malloc(100);

    // memset(포인터, 값, 크기);
    // 문자열을 의도한 값으로 미리 채우기
    memset(preparedA, 'A', 10);

    for (int i = 0; i < 50; i++) {
        printf("%c ", preparedA[i]); // A A A A A A A A A A x00 x00 x00 x00 ..
    }

    free(preparedA);

    return 0;
}
```

### Allocate Double Pointers
```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {

    // 행 공간 생성
    // doublePointer -> [공간 1], [공간 2], [공간 3]
    int **doublePointer = (int**)malloc(sizeof(int*) * 3);

    
    printf("sizeof(int*) : %lu \n", sizeof(int*));  // 8바이트, 주소의 크기

    // 열 공간 생성
    for (int i = 0; i < 3; i++ ){
        // int의 크기만큼 각 행마다 4개씩 공간 할당
        // doublePinter -> [주소 1], [공간 2], [공간 3]
        // [주소 1] -> [], [], [], []
        *(doublePointer + i) = (int*)malloc(sizeof(int) * 4);

        printf("sizeof(doublePointer) : %lu \n", sizeof(doublePointer[i])); // 주소를 담는 크기 8바이트
        printf("sizeof(&doublePointer) : %lu \n", sizeof(&doublePointer[i])); // 주소를 담는 크기 8바이트
        printf("sizeof(**doublePointer) : %lu \n", sizeof(**doublePointer)); // doublePointer는 int**, 역참조한 **doublePointer는 4바이트 int
    }

    printf("sizeof(int) : %lu\n", sizeof(int));    // 4바이트, int의 크기

    // 값 할당
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 4; j++) {
            // 0 ~ 11까지의 값을 할당
            // doublePinter -> [주소 1], [주소 2], [주소 3]
            // [주소 1] -> [값 1], [], [], []
            *( *(doublePointer + i) + j) = i * 4 + j;
        }
    }


    // 0 0x7...304080 1 0x7...304084 2 0x7...304088 3 0x7...30408c 
    // 4 0x7...2040a0 5 0x7...2040a4 6 0x7...2040a8 7 0x7...2040ac 
    // 8 0x7...404080 9 0x7...404084 10 0x7...404088 11 0x7...40408c 

    // 0x7...304080 − 0x7...2040a0 = fffe0 -> 행 간에는 넓은 간격
    // 열 간에는 4바이트씩 증가

    // 출력 결과
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 4; j++) {
            printf("%d ", *( *(doublePointer + i) + j));
            printf("%p ", ( *(doublePointer + i) + j));
        }

        printf("\n");
    }

    // 각 행도 메모리 해제를 해주어야 함
    // malloc을 사용한 만큼 free를 쓴다고 생각하기
    for (int i = 0; i < 3; i++) {
        free(doublePointer[i]);
    }

    free(doublePointer);

    return 0;
}
```