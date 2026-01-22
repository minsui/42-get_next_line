*This project has been created as part of the 42 curriculum by <mkim2>.*

# get_next_line

## Description (설명)
**get_next_line** 프로젝트의 목표는 파일 디스크립터로부터 한 줄씩 읽어오는 함수를 구현하는 것입니다. 정적 변수의 개념을 정확히 이해하고, 효율적인 메모리 관리와 BUFFER_SIZE의 크키에 따른 로직을 구축하는 것이 핵심입니다.

### 상세 라이브러리 설명
본 라이브러리는 다음과 같은 기능을 포함합니다:
- **함수 함수 구성**: 

get_next_line.c: 전체 흐름을 제어

read_file: 개행 문자가 나타날 때까지 반복적으로 파일을 읽어 backup에 저장

extract_line: 저장된 데이터에서 실제 반환할 '한 줄'을 잘라냄

update_backup: 반환한 줄을 제외한 나머지 데이터를 다음 호출을 위해 보관

get_next_line_utils.c: strlen, strchr, strjoin이 포함된 문자열 처리 도구

---

## Instructions (사용 방법)

### 컴파일 및 빌드
- 일반 컴파일: cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line.c get_next_line_utils.c

- 보너스 컴파일: cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line_bonus.c get_next_line_utils_bonus.c

### 사용 예시(main.c)

#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include "get_next_line.h"
#include "get_next_line_bonus.h"

int main(int argc, char **argv)
{
    int     fd;
    char    *line;

    if (argc == 2)
        fd = open(argv[1], O_RDONLY);
    else
        fd = 0; 

    if (fd == -1)
        return (1);

    while ((line = get_next_line(fd)) != NULL)
    {
        printf("[%s]", line); 
        free(line);
    }
}

// int main(void)
// {
//     int     fd[3];
//     char    *line;
//     int     i;
//     int     active;

//     fd[0] = open("test1.txt", O_RDONLY);
//     fd[1] = open("test2.txt", O_RDONLY);
//     fd[2] = open("test3.txt", O_RDONLY);

//     if (fd[0] < 0 || fd[1] < 0 || fd[2] < 0)
//     {
//         printf("Error: test1.txt, test2.txt, test3.txt를 확인하세요.\n");
//         return (1);
//     }

//     active = 1;
//     while (active)
//     {
//         active = 0;
//         i = 0;
//         while (i < 3)
//         {
//             if (fd[i] != -1)
//             {
//                 line = get_next_line(fd[i]);
//                 if (line)
//                 {
//                     printf("[FD %d] %s", fd[i], line);
//                     free(line);
//                     active = 1;
//                 }
//                 else
//                 {
//                     close(fd[i]);
//                     fd[i] = -1;
//                 }
//             }
//             i++;
//         }
//     }
//     return (0);
// }

### 알고리즘 및 데이터 구조
1. 정적 변수 기반 상태 보존 함수가 종료되어도 값이 유지되는 static 변수를 사용하여, BUFFER_SIZE만큼 읽어오고 남은 데이터를 다음 함수 호출 시에도 잃어버리지 않고 이어서 처리할 수 있도록 설계

2. 동적 버퍼링 로직 BUFFER_SIZE가 1이든 1,000,000이든 상관없이 동일하게 동작

개행 문자를 찾을 때까지 while 루프 내에서 read()를 반복 수행하며, 읽어온 데이터는 ft_strjoin을 통해 누적적으로 합쳐짐

3. Multi-fd 관리 static char *backup[OPEN_MAX] 배열 구조

각 파일 디스크립터의 인덱스에 고유한 backup 공간을 할당함으로써 여러 파일을 번갈아 가며 읽어도 데이터가 섞이지 않고 독립적인 읽기 상태를 유지

---

## Resources (참고 자료)
- [42 Seoul Subject PDF]
- [Linux Man Pages]: read(2), malloc(3), free(3)

## 기술적 선택 사항 (Technical Rationale)
### 메모리 할당 전략  : 왜 매 단계마다 free를 검사하는가?

GNL은 반복적인 malloc과 free가 일어나는 프로젝트이므로, 메모리 누수 방지가 최우선인 과제이다.

따라서 

Dangling Pointer 방지: free를 한 직후에는 반드시 해당 포인터를 NULL로 초기화하여, 이미 해제된 메모리에 접근하는 Segfault 에러를 차단

예외 발생 시 즉시 해제: read() 에러(-1)나 malloc 실패 시 현재까지 쌓아온 backup 데이터를 즉시 free 하고 NULL을 리턴하여 시스템 자원을 보호하도록 설계

---
### AI 활용 기술 기술
본 프로젝트의 완성도를 높이기 위해 AI(Gemini)를 다음과 같이 활용하였습니다:
- **strjoin함수 25줄 최적화 학습** : 25줓 제한에 어려움이 있어 코드 최적화 기법 학습
- **메모리 누수 학습** : 어느 로직에서 메모리 누수를 막아야하는지에 관한 개념이 잡히지 않아 메모리 흐름과 누수에 대한 학습

