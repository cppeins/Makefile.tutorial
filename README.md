# Makefile Tutorial
[A Simple Makefile Tutorial](http://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/) 을 정리한 내용입니다.

Makefiles are a simple way to organize code compilation. This tutorial does not even scratch the surface of what is possible using make, but is intended as a starters guide so that you can quickly and easily create your own makefiles for small to medium-sized projects.

## A Simple Example
### hellomake.c 작성
```c
#include <hellomake.h>

int main() {
  // call a function in another file
  myPrintHelloMake();
  return(0);
}
```

### hellofunc.c 작성
```c
#include <stdio.h>
#include <hellomake.h>

void myPrintHelloMake(void) {
  printf("Hello makefiles!\n");
  return;
}
```

### hellomake.h 작성
```c
/*
example include file
*/

void myPrintHelloMake(void);
```

### gcc 컴파일
```shell
$ gcc -o hellomake hellomake.c hellofunc.c -I.
```
이 명령을 hellomake.c, hellofunc.c 파일을 컴파일하고 hellomake 실행파일을 생성합니다.  
-I. 옵션은 gcc에 현재 디렉토리(.)의 header 파일을 포함하기 위해 추가되었습니다.  
Makefile 이 없다면, 테스트/수정/디버그 주기동안 터미널을 매번 타이핑하지 않기 위해서 지난 명령들을 찾기 위해 위 아래 화살표를 계속 눌려아 합니다.  
특히 .c 파일을 더 추가할 때마다 더욱 그렇습니다.

불행하게도 이 방식은 큰 단점이 2가지가 있습니다. 첫 번째는, 가장 비효율적으로 컴파일 명령을 잊어버리거나 컴퓨터를 바꾸게 되면 처음부터 다시 입력해야합니다. 두 번째는 만약 하나의 .c 파일을 수정할 때 모두 다시 컴파일하게 된다면, 시간도 많이 소모되고 비효율적입니다. 이제 makefile 로 무얼할 수 있을 지 알아보겠습니다.
<br />
<br />

## [Makefile1](http://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/makefile.1)
```cmake
hellomake: hellomake.c hellofunc.c
    gcc -o hellomake hellomake.c hellofunc.c -I.
```
위의 내용을 **Makefile** 이나 **makefile** 에 추가하고 **make** 명령을 실행하면 작성된 컴파일 명령을 실행합니다.  
**make** 를 인자 없이 실행하면 이 파일을 실행합니다.  
첫 번째 라인의 hellomake 의 **:** 다음에 파일 리스트를 지정하여 파일 중 하나라도 변경되면 실행해야한다는 것을 알 수 있습니다.
바로 첫 번째 문제를 해결했습니다. 그러나 시스템은 최신 변경 사항만 컴파일하기 때문에 여전히 효율적이지 않습니다.
더 효율적
<br />
<br />

## [Makefile2](http://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/makefile.2)
```cmake
CC=gcc
CFLAGS=-I.

hellomake: hellomake.o hellofunc.o
     $(CC) -o hellomake hellomake.o hellofunc.o -I.
```
이제 **CC** 와 **CFLAGS** 라는 상수를 정의했습니다.  
이 들은 hellomake.c 와 hellofunc.c 파일을 컴파일하는 방법을 **make** 에 알려주는 특별한 상수니다.  
특히 **CC** 매크로는 사용할 C 컴파일러고 **CFLAGS** 는 컴파일 명령에 전달할 플래그 리스트입니다.  
hellomake.o 와 hellofunc.o object 파일을 리스트에 추가하면, **make** 는 **.c** 버전은 개별로 컴파일해야한다는 것을 알고, 그 다음에 **hellomake** 실행파일을 생성합니다.  
<br />
이 형태의 makefile 은 대부분의 작은 프로젝트에서는 충분합니다. 그러나, 한 가지 빠진 점이 있습니다:  
include 파일들에 대한 종속성. 만약, hellomake.h를 변경했을 때, **make** 는 **.c** 파일을 재컴파일하지 않아야 합니다.  
이 문제를 해결하기 위해서, **make** 에게 모든 **.c** 파일들이 특정한 **.h** 파일에 의존하고 있다는 것을 알려줘야 합니다.  
우리는 간단한 규칙을 작성하고 이를 makefile에 추가함으로써 이것을 할 수 있습니다.
<br />
<br />

## [Makefile3](http://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/makefile.3)
```cmake
CC=gcc
CFLAGS=-I.
DEPS = hellomake.h

%.o: %.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

hellomake: hellomake.o hellofunc.o
	gcc -o hellomake hellomake.o hellofunc.o -I.
```
이번에는 먼저 **.c** 파일이 의존하고 있는 **.h** 파일들의 집합인 DEPS 매크로를 작성했습니다.  
그 다음에는 **.o** 로 끝나는 모든 파일에 적용되는 규칙을 정의합니다.  
이 규칙은 **.o** 파일이 **.c** 파일과 DEPS 매크로에 포함된 **.h** 파일에 의존한다고 말합니다.  
이 규칙은 또 **.o** 파일을 생성하기 위해 **make** 가 **CC** 로 정의된 컴파일러를 사용하여 **.c** 파일을 컴파일해야한다고 말합니다.  
**-c** 플래그는 object 파일을 생성하는 것을 나타내고, **-o $@** 는 컴파일 출력물을 **:** 왼쪽의 파일에 추가하라고 알려줍니다.  
**$<** 는 종속 모델의 첫 번째 항목이며 **CFLAGS** 매크로는 위와 같이 정의되었습니다.
<br />
마지막으로, **:** 의 왼쪽과 오른쪽을 나타내는 특별한 매크로 **$@** 와 **$^** 사용하여 컴파일 규칙을 일반적으로 만듭니다.  
아래 예제에서 모든 include 파일은 DEPS 매크로의 일부로 나열되어야 하며 모든 object 파일은 OBJ 매크로의 일부로 나열되어야합니다.
<br />
<br />

## [Makefile4](http://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/makefile.4)
```cmake
CC=gcc
CFLAGS=-I.
DEPS = hellomake.h
OBJ = hellomake.o hellofunc.o

%.o: %.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

hellomake: $(OBJ)
	gcc -o $@ $^ $(CFLAGS)
```
그렇다면, **.h** 파일을 include 디렉토리에 소스 코드 를 src 디렉토리에, 일부 라이브러리들을 lib 디렉토리에 넣으려면 어떻게 해야할까요? 또, 어떻게 성가신 **.o** 파일을 숨길 수 있을까요?  
다음 makefile은 include 및 lib 디렉토리에 대한 경로를 정의하고 src 디렉토리 내의 obj 서브 디렉토리에 object 파일을 저장합니다.  
또한 수락 라이브러리 **-lm** 과 같이 포함하려는 라이브러리에 대해 이미 정의된 매크로가 있습니다.  
이 makefile은 src 디렉토리에 있어야 합니다.
**make clean** 을 입력하면 소스 및 object 디렉토리를 정리하는 규칙도 포함되어 있습니다.  
**.PHONY** 규칙은 **make** 가 **clean** 이라는 이름의 파일로 무엇인가를 하지 못하게 합니다.
<br />
<br />

## [Makefile5](http://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/makefile.5)
```cmake
IDIR =../include
CC=gcc
CFLAGS=-I$(IDIR)

ODIR=obj
LDIR =../lib

LIBS=-lm

_DEPS = hellomake.h
DEPS = $(patsubst %,$(IDIR)/%,$(_DEPS))

_OBJ = hellomake.o hellofunc.o
OBJ = $(patsubst %,$(ODIR)/%,$(_OBJ))


$(ODIR)/%.o: %.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

hellomake: $(OBJ)
	gcc -o $@ $^ $(CFLAGS) $(LIBS)

.PHONY: clean

clean:
	rm -f $(ODIR)/*.o *~ core $(INCDIR)/*~
```
이제 중소 규모의 프로젝트를 관리하기 위해 수정할 수 있는 **makefile** 이 생겼습니다.  
**makefile** 에 여러 규칙을 추가할 수도 있습니다. 다른 규칙을 호출할 수 있는 규칙을 만들 수도 있습니다.  
**makefile 과 **make** 함수에 대한 자세한 정보는 **GNU Make Manual** 을 참조하세요.
<br />
<br />

## References
  ### [A Simple Makefile Tutorial](http://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/)
