## 프로그램 이름:2076036
  이 프로그램은 파일로부터 assembly 소스 프로그램을 입력받아 프로그래밍 언어의 기본적인 단위인 Token 을 생성하여 화면과 출력파일(output.txt)에 출력하는 프로그램이다.

## 프로그램 실행 전 과정
1. visual studio에 접속하여 입력받은 파일을 토큰으로 분리하여 파일에 출력하는 코드를 구성한다. 

2. 소스파일에 Main.c와 Control.c를 생성하고 헤더파일에 Control.h를 생성한다.

3. 입력 받을 "example.asm"파일을 작성한 뒤 C:\Users\jenny\source\repos\2076036\2076036 경로에 저장한다. 이 경로에는 위에 정의된 소스파일과 헤더파일도 저장되어있다.


## 프로그램 실행 과정
1. 이 프로그램은 C언어로 구현되었기때문에 visual studio에서 실행할 수 있다.
2. 처음 프로그램을 실행하면 첫번째 줄에 프로그램이 하는 일이 출력된다.-> "원하는 assembly 파일을 입력 받아 token으로 분리하기."
3. 두번째 줄에는 "1. 파일 불러오기" 가 출력되며 세번쨰 줄에는 "2. 프로그램 종료"가 출력된다. 1을 선택하면 프로그램이 실행되고 2를 선택하면 프로그램이 종료된다.
4. 네번째 줄에는 "1과 2 중에 선택하여 실행하고싶은 기능을 입력하세요:"가 출력된다. 이때 프로그램을 실행하고싶다면 1을 입력하고 프로그램을 종료하려면 2를 입력한다.
5. 만약 1을 입력한다면 프로그램이 실행되어 "입력 받을 파일 이름을 입력하세요:"가 출력되고 2를 입력하면 프로그램이 종료된다. 
6. 내용을 토큰으로 분리하여 "output.txt" 파일에 출력하고싶은 "example.asm"파일을 입력한다.
7. 프로그램이 정상적으로 작동하였다면 "파일 처리를 완료하였습니다."라고 출력된다.
8. 이후 "output.txt"파일을 확인하면 "example.asm"파일의 내용이 토큰으로 분리되어 저장되어 있다.

## 소스코드
* **Main.c**
```
#define _CRT_SECURE_NO_WARNINGS
#include <stdlib.h>
#include "Control.h"

int main(void)
{
	int input;
	while (1)//프로그램 종료인 2를 입력할 때까지 반복한다
	{
		printf("원하는 assembly 파일을 입력 받아 token으로 분리하기\n");
		printf("1.파일 불러오기\n");
		printf("2.프로그램 종료\n");
		printf("1과 2 중에 선택하여 실행하고싶은 기능을 입력하세요:");
		scanf("%d",&input);
		getchar();
		switch (input)
		{
		case 1:
			if (readFile() == 0)
				continue;
			if (writeFile() == 0)
				continue;
			fileControl();
				break;
		case 2:
					printf("프로그램을 종료합니다.\n");
					system("pause");
					return 0;
		default:
					printf("잘못된 입력입니다. \n");
					break;
		}
	}
}
```
* **Control.c**
```
#include "Control.h"

int writeFile()//파일을 out.txt에 쓰기 위해 파일을 여는 함수
{

	wf = fopen("output.txt", "w");

	if (wf == NULL) {

		printf("파일 쓰기에 실패하였습니다.\n");
		return 0;
	}

	return 1;
}

int readFile() {// 입력한 파일을 읽기 위해 파일을 여는 함수


	printf("입력받을 파일 이름을 입력하세요:");
	gets_s(fileName, sizeof(fileName));
	rf = fopen(fileName, "r");
	if (rf == NULL)
	{
		printf("파일 읽기에 실패하였습니다. \n");
		return 0;
	}
	return 1;
}

int fileControl()//파일을 처리하기 위한 함수
{
	char delimit[] = " \t\n,.";
	//공백 문자와 '-' 으로 단어를 구분
	char* token = NULL;
	

	while (!feof(rf))//파일이 끝날때까지 읽어온다
	{
		
		fgets(line, sizeof(line), rf);
		token = strtok(line, delimit);
		

		while (token != NULL)// 라인의 모든 단어를 토큰으로 인식
		{
			fprintf(wf, "%s\n", token);
			token = strtok(NULL, delimit);
			

		}
		

	}
	//파일을 다 처리한 후 파일을 닫음
	fclose(rf);
	fclose(wf);

	printf("파일 처리를 완료하였습니다.\n");
}
```
* **Control.h**
```
#pragma once
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <string.h>
	
FILE *rf;//파일 입력 변수 선언
FILE *wf;//파일 출력 변수 선언


char fileName[256];//파일 이름 최대 길이 선언
char line[80];//파일 한 줄 최대 길이 선언
char word[256];// 파일 단어 최대 길이 선언


//파일을 입출력하는 함수 선언
int readFile();
int writeFile();
int fileControl();
```
