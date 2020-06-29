---
title: "[42 Seoul] Get Next Line 과제를 수행하며"
# excerpt: "정말 10분이면 만들 수 있다"
date: 2020-06-29 15:00:00 +0900
categories: study
tags: GNL 42 C
header:
  teaser: # /assets/blog2.png
  image: # /assets/mozilla_logo.png 

toc: true  
toc_sticky: true 
---

42의 Get Next Line 과제를 수행하며 정리하는 내용이다. 코로나로 인해 장기간 닫혀있던 42 클러스터의 문이 열려서 모처럼 공부를 하게 되었는데, 코드 자체를 너무 오랜만에 보니 적응이 하나도 안 되어 하나하나 정리하며 공부하기로 했다.

### 과제를 수행하기 전 알아야 할 내용
#### 파일 디스크립터(fd)
 * 운영체제가 파일 또는 하드웨어와 통신을 편히 하기 위해 부여한 숫자라고 한다.
 * 파일 디스크립터는 0,1,2 순으로 숫자를 부여하며, 0,1,2는 이미 사용중이어서 3부터 파일 디스크립터를 부여한다.
   * 0 : 표준 입력
   * 1 : 표준 출력
   * 2 : 표준 에러

#### read() 함수
```
size_t read(int fd, void *buf, size_t bytes)
```
 * 인자로 받은 bytes의 수 만큼 fd를 읽어 buf에 저장하는 함수다.
 * 읽어 온 바이트 수를 반환하며, 실패시 -1을 반환한다.
 * 파일을 끝까지 읽었다면, 다음 번에는 더 이상 읽을 바이트가 없기 때문에 0을 반환한다고 한다.

#### static 변수
  * 메모리의 데이터 영역에 저장되어 프로그램이 종료될 때까지 남아있는 변수다. 다음 line을 읽을 시작 주소를 계속 저장할 수 있도록 백업 버퍼를 만들어 static 변수로 선언해야 한다.
  * [위키피디아의 정적 변수 설명](https://ko.wikipedia.org/wiki/%EC%A0%95%EC%A0%81_%EB%B3%80%EC%88%98)

#### gcc의 -d 플래그
 * 프로그램 외부에서 `#define`을 정의할 수 있다.
 * 이 과제에서, 채점 시 컴파일은 다음과 같이 진행한다.
```
  $ gcc -Wall -Wextra -Werror -D BUFFER_SIZE=32 get_next_line.c get_next_line_utils.c
```
 * 즉 컴파일할 때 BUFFER_SIZE를 정하고 이 버퍼의 크기만큼 파일을 한 번에 읽게 된다.

### 과제의 목표
 * GNL 함수를 반복문 안에서 호출하면, fd의 텍스트를 EOF가 올 때까지 한 번에 한 줄씩 읽을 수 있다.
 * GNL 함수를 처음 호출했을 때 지정한 버퍼의 크기가 매우 커서 한 번에 파일을 끝까지 읽었다 하더라도, 두 번째 호출했을 때는 두번째 줄부터 읽기를 시작해야 한다.
 * file, redirection, stdin 으로부터 읽었을 때 함수가 제대로 작동해야 한다.
 * gcc -d 플래그로 받은 BUFFER_SIZE가 1일 때도, 9999일 때도, 10000000일 때도 함수가 제대로 작동해야 한다.

 ### 작동 구조 아이디어
  1. 파일을 read할 임시 버퍼를 만든다.
```
char buf[BUFFER_SIZE];
```
  2.  read한 버퍼를 백업할 static 버퍼를 만든다.