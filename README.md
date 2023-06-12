## 프로그램 이름:sp2076036_proj3
  이번 프로젝트는 프로젝트 #2에서 구현한 어셈블러에 linking과 loading 기능을 추가하는 프로그램이다. 프로젝트 #2 에서 구현된 assemble 명령을 통해서 생성된 object 파일을 link시켜 메모리에 올리는 일을 수행하게 만든다.



## 프로그램 구현과정
1. loadProgram 함수
  loadProgram 함수는 파일에서 프로그램을 읽어와 메모리에 로드하는 역할을 한다.  FILE* file = fopen(filename, "r")를 사용하여 파일을 열고, 레코드 타입에 따라 H, T, M, E를 처리한다. H(Hader) 레코드는 섹션의 시작 주소와 길이를 저장한다. 파일에서 시작 주소와 길이를 읽어와 controlSections 배열에 저장한다. T(Text) 레코드는 메모리에 프로그램 코드를 로드한다. 파일에서 주소와 길이를 읽어와 해당하는 메모리 위치에 값을 저장한다. M(Modification) 레코드는 메모리에 주소 값을 더하는 연산을 수행합니다. 파일에서 주소와 길이를 읽어와 해당하는 메모리 위치의 값을 주어진 값만큼 증가시킨다. E(End) 레코드는 프로그램 로드를 종료한다.

2. printLoadMap 함수
  printLoadMap 함수는 로드한 프로그램의 맵을 출력하는 역할을 한다. 우선 ("control symbol address length\n")와 ("section name\n")를 출력하고 controlSections 배열에 저장된 섹션 정보를 순회하며, 각 섹션의 이름, 주소, 길이를 출력한다. 모든 섹션 정보를 출력한 후, 총 길이를 계산하여 출력합니다.

3. dumpMemory 함수
  dumpMemory 함수는 메모리를 덤프하는 역할을 한다. 시작 주소와 길이를 입력받아 해당 범위의 메모리 값을 출력한다. 시작 주소부터 시작하여 길이만큼의 메모리 값을 출력하는데, 16바이트마다 줄바꿈을 하여 가독성을 높인다.

4. main 함수
  main 함수는 프로그램의 진입점으로, 사용자의 명령어를 입력받고 실행하는 역할을 한다. 우선 sicsim> progaddr [address]:를 출력하여 progaddr 명령을 입력받아 초기 주소(progAddr)를 설정한다. 다음으로 sicsim> loader [object filename1] [object filename2] [...]를 출력하여 loader 명령을 입력받아 프로그램 파일을 로드한다. 이때 원하는 OBJ파일을 입력한다. 각각의 로드된 섹션의 정보를 출력한다. 마지막으로 sicsim> dump [start address], [length]를 출력하여 dump 명령을 입력받아 메모리를 덤프한다. 이 코드는 프로그램 파일을 메모리에 로드하고, 프로그램의 맵을 출력하며, 메모리를 덤프하는 기능을 제공한다. 사용자는 progaddr, loader, dump 등의 명령을 통해 원하는 작업을 수행할 수 있다.



## 소스코드
```
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#pragma comment(linker, "/STACK:100000000")
#pragma comment(linker, "/HEAP:100000000")



#define MAX_SIZE 1000
#define MEMORY_SIZE 1048576  // 1M

typedef struct {
     char name[MAX_SIZE];
     int address;
     int length;
} ControlSection;

void loadProgram(char* filename, ControlSection* controlSections, int* controlSectionCount, char* memory) {//프로그램 로드하는 함수
     FILE* file = fopen(filename, "r");
     if (file == NULL) {
          printf("파일을 열 수 없습니다.\n");//파일을 열 수 없을때
          exit(1);
     }

     char record_type;
     while (fscanf(file, "%c", &record_type) != EOF) {
          if (record_type == 'H') {//H코드 
               int start_address;
               int length;
               fscanf(file, "%X %X", &start_address, &length);
               controlSections[*controlSectionCount].address = start_address;
               controlSections[*controlSectionCount].length = length;
          }
          else if (record_type == 'T') {//T코드
               int address;
               int length;
               fscanf(file, "%X %X", &address, &length);

               int i;
               for (i = 0; i < length; i++) {
                    int value;
                    fscanf(file, "%2X", &value);
                    memory[address + i] = (char)value;
               }
          }
          else if (record_type == 'M') {//M코드
               int address;
               int length;
               fscanf(file, "%X %X", &address, &length);

               int i;
               for (i = 0; i < length; i++) {
                    int value;
                    fscanf(file, "%2X", &value);
                    memory[address + i] += (char)value;
               }
          }
          else if (record_type == 'E') {//E코드 일때
               break;
          }
     }

     fclose(file);
}

void printLoadMap(ControlSection* controlSections, int controlSectionCount) {//로드맵 출력하는 함수
     printf("control symbol address length\n");
     printf("section name\n");
     printf("------------------------------------------\n");

     int i;
     for (i = 0; i < controlSectionCount; i++) {
          printf("%-7s %04X %04X\n", controlSections[i].name, controlSections[i].address, controlSections[i].length);
     }

     printf("------------------------------------------\n");
     printf("total length %04X\n", controlSections[controlSectionCount - 1].address + controlSections[controlSectionCount - 1].length);//총길이
}

void dumpMemory(char* memory, int startAddress, int length) {//메모리 덤프하는 함수
     int i;
     for (i = startAddress; i < startAddress + length; i++) {
          if (i % 16 == 0) {
               printf("%04X ", i);
          }

          printf("%02X ", (unsigned char)memory[i]);

          if ((i + 1) % 16 == 0) {
               printf("\n");
          }
     }

     if (startAddress % 16 != 0) {
          printf("\n");
     }
}

int main() {
     _setmaxstdio(2048);
     char filenames[MAX_SIZE][MAX_SIZE];
     ControlSection controlSections[MAX_SIZE];
     char memory[MEMORY_SIZE];
     int controlSectionCount = 0;
     int progAddr = 0x00;
     
     printf("sicsim> progaddr [address]: ");
     char command[MAX_SIZE];
     fgets(command, MAX_SIZE, stdin);

     char* token = strtok(command, " ");
     if (strcmp(token, "progaddr") == 0) {
          token = strtok(NULL, " ");
          if (token != NULL) {
               progAddr = strtol(token, NULL, 16);
          }
     }

     printf("sicsim> loader [object filename1] [object filename2] [...]: ");//입력된 OBJ파일로 메모리에 결과기록
     fgets(command, MAX_SIZE, stdin);

     token = strtok(command, " ");
     if (strcmp(token, "loader") == 0) {
          token = strtok(NULL, " ");
          while (token != NULL) {
               token[strcspn(token, "\n")] = '\0';  // 개행 문자 제거

               strcpy(filenames[controlSectionCount], token);
               loadProgram(token, controlSections, &controlSectionCount, memory);

               token = strtok(NULL, " ");
               controlSectionCount++;
          }
     }

     printf("\n");

     if (controlSectionCount > 0) {
          printf("Load Map:\n");//성공하면 로드맵 출력
          printLoadMap(controlSections, controlSectionCount);
     }

     printf("sicsim> dump [start address], [length]: ");
     fgets(command, MAX_SIZE, stdin);

     token = strtok(command, ",");
     if (strcmp(token, "dump") == 0) {
          token = strtok(NULL, ",");
          int startAddress = strtol(token, NULL, 16);

          token = strtok(NULL, ",");
          int length = strtol(token, NULL, 16);

          printf("\n");
          dumpMemory(memory, startAddress, length);
     }

     return 0;
}
```
