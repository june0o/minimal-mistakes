---
title: "C/C++ 에서 Java 호출하기 - 기본편"
permalink: /C-Java-Call-page/
---

## C/C++ 에서 Java 호출하기 - 기본편 :seedling:

: C/C++ 에서 Java api 를 호출하는 가장 간단한 방법에 대해 알아봅니다.

### 0. 개요
-  Java 작성 및 빌드
-  C/C++ 에서 Java 호출 코드 작성
-  빌드 및 테스트 (window, linux)

### 1. Java 작성 및 빌드
: JDK 는 설치되어 있어야 함
#### 1) Hello world 를 출력하는 class 를 하나 작성합니다.

    public class Hello {
        public static void test() {
            System.out.println("Hello world");
        }
    }

#### 2) 빌드하여 Hello.class 가 생성되는 것을 확인 합니다.

    $ javac Hello.java
     
### 2. C/C++ 에서 Java 호출 코드 작성
#### 1) jvm 생성 및 java class 찾기
: jvm 을 생성하고, class.path 로 지정된 위치에서 java class 를 찾는다.

	printf("JVM Create Start!!\n");

	JavaVM *jvm;
	JNIEnv *env;
	jclass cls;
	JavaVMOption options[1];
	JavaVMInitArgs vm_args;

	options[0].optionString = "-Djava.class.path=.";  //<- set class path
	memset(&vm_args, 0, sizeof(vm_args));
	vm_args.version = JNI_VERSION_1_6;
	vm_args.nOptions = 1;
	vm_args.options = options;
  
	long jvm_status = JNI_CreateJavaVM(&jvm, (void**)&env, &vm_args);  //<- create jvm
	if (jvm_status != JNI_ERR)
	{
	    cls = env->FindClass("Hello");  //<- find class
	}

	if (cls != 0) 
	{
	    printf("Find class!\n");
	}
    
#### 2) java api 호출
: Hello class 의 test api를 호출한다.

	if (cls != 0)
	{
		jmethodID mid = env->GetStaticMethodID(cls, "test", "()V");
		if (mid != 0)
		{
		    env->CallStaticVoidMethod(cls, mid);
		}
		else
		{
		    printf("mid error\n");
		    return false;
		}
	}

#### 3) jvm 삭제

	g_jvm->DestroyJavaVM();

### 3. 빌드 및 테스트
#### 1) window 환경 (x64)
- window console app project 를 하나 생성

> 아래 설정들 project 파일에 추가
- include path 설정

Project Properties>C/C++>General>Additional include Directories 에 추가
ex)
%JAVA_HOME%\include;%JAVA_HOME%\include\win32

![b62d48e0](https://user-images.githubusercontent.com/38425370/44067002-459e9648-9fae-11e8-8f78-c65dbc83e03e.png)

- jvm library path 설정

Project Properties>Linker>General>Additional Library Directories 에 추가
ex)
%JAVA_HOME%\lib;%JAVA_HOME%\jre\bin\server

![70a252c7](https://user-images.githubusercontent.com/38425370/44067027-66001e48-9fae-11e8-92e2-168b32ecf6a0.png)

- jvm library dependency 설정

Project Properties>Linker>Input>Additional Dependencies 에 jvm.lib 추가
![2ebfc32f](https://user-images.githubusercontent.com/38425370/44067052-7a58a432-9fae-11e8-98b0-dc6f642dc86f.png)

- Exception setting 수정

Debug>Windows>Exception Settings
Access violation 체크 해제. 해제하지 않으면 실행 시, Access violation error 발생한다.
![8c1872f2](https://user-images.githubusercontent.com/38425370/44067065-865e8b7a-9fae-11e8-9e5b-bb23d89f11d0.png)

- 실행

Hello world 가 출력되는 것 확인.
![1973005f](https://user-images.githubusercontent.com/38425370/44067068-8d269d58-9fae-11e8-92b6-3cf2e80a5592.png)

#### 2) linux 환경
- /lib64/ 에 libjvm.so 심볼릭 링크 걸어줘야 함

ex)
ln -s $(JAVA_HOME)/jre/lib/amd64/server/libjvm.so /lib64/libjvm.so

> 아래 설정들 make 파일에 추가
- include path 설정

export JDK_INC := -I $(JAVA_HOME)/include -I $(JAVA_HOME)/include/linux

- jvm library path 설정

export JDK_LIB := -L $(JAVA_HOME)/lib/amd64 -L $(JAVA_HOME)/jre/lib/amd64/server/

- jvm library dependency 설정

LIBS := -ljvm

- 실행

Hello world 가 출력되는 것 확인.
![5902498e](https://user-images.githubusercontent.com/38425370/44067076-972ec03c-9fae-11e8-8469-d446c367cd5e.png)
