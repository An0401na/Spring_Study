
![spring](https://github.com/An0401na/Spring_Study/assets/99172832/72d12fc8-c3fa-4d54-9001-8aa44c33a5ca)
<br><br>
### ➡️ 프로젝트 생성

**0\. 사전 준비 : ~Java 11 설치,~ Java 17 설치, IntelliJ 설치**

Spring Boot 2.3.x 를 사용할 경우는 Java 11을

Spring Boot 3.0.x 를 사용할 경우에는 Java 17을 설치해야 한다.

글에서는 11로 설치했다가 다시 17로 재설치 하였다

<br><br>

**<Java8에서 Java11(Java17)로 변경하기>**

**1\. Java 버전 확인하기**

cmd 창에 java -version 명령어를 입력하면 자바 버전을 확인 할 수 있다. 


현재 자바 8을 사용하고 있음을 확인 했다.

Java 11을 설치해보도록 하자.

**2\. Java 11 설치하기**

[www.oracle.com/java/technologies/javase-jdk11-downloads.html](http://www.oracle.com/java/technologies/javase-jdk11-downloads.html)

위 링크에서 **jdk-11.0.9**  를 설치 하면 된다.

다운로드 후

C:\\Program Files\\Java

로컬 파일 위치에서 jdk 파일과 jre 파일들을 확인 할 수 있었다. 

자바 11 위치   :   C:\\Program Files\\Java\\jdk-11

나중에 환경변수 설정할 때 경로가 필요하니 위치를 잘 찾아두고 복사해두자

\------
글 작성 후 spring 3.0버전을 사용하기 위해서 자바 17을 다시 재설치 해주었다.

[https://www.oracle.com/java/technologies/downloads/#java17](https://www.oracle.com/java/technologies/downloads/#java17)
<br><br>
**3\. 환경변수 설정하기**


윈도우 검색 창에서 "환경 변수"를 검색하면 제어판 \[시스템 속성\] 을 열수 있다. 환경 (띄고) 변수 로 입력해야한다.

\[환경 변수\] - \[시스템 변수\]-\[JAVA\_HOME\]의 path 값을 변경해주어야 한다.


위에서 복사해둔 값으로 변경 

다시 자바 버전을 확인해두면 11로 변경됨을 확인할 수 있다.

---
<br><br><br><br>
**1\. 스프링 부트 스타터 사이트로 이동해서 스프링 프로젝트 생성**

스프링 부트 스타터 : 스프링 프로젝트를 편리하게 만들어주는 사이트 

[https://start.spring.io/](https://start.spring.io/)


위 링크 접속 후 프로젝트 초기 설정을 위와 같이 설정해주었다.

Maven, Gradle 은 라이브러리를 땡겨오고 스프링의 라이프 사이클을 관리해주는 툴이라고 생각하자

Dependencies는 웹 프로젝트를 하기 위해서 ****Spring Web****과 html 만들어주는 템플릿 엔진을 위한 **Thymeleaf** 를 추가해주었다. 


하단 GENERATE 눌러서 다운로드 
<br><br><br><br>
**2\. 인텔리제이에서 스프링 프로젝트 생성**

인텔리제이 워크스페이스를 만들어서 다운로드 받은 파일을 압축을 풀어 넣는다.

나는 spring\_study 라는 폴더를 만들어 압축을 풀어 넣었다.

인텔리제이를 켠 후 Open file or project 를 선택하여 만들어둔 워크스페이스를 찾아 압축을 푼 파일을 펼친다.

펼치면 build.gradle 이라는 파일을 찾을 수 있다.

선택 후 open 한다.

라이브러리 다운로드에 시간이 걸릴 수 있다.

마친 후 build.gradle을 열어보면


이렇게 스타터 를 통해서 설정한 프로젝트 환경이 제대로 작성됨을 확인 할 수 있다. 
<br><br><br><br>
**3\. 스프링 부트 실행**

위 파일을 찾아 실행해보면 정상적으로 확인 할 수 있다.

웹서버가 8080 포트로 실행됐음을 확인했으니 창을 띄워보면


이런 에러 페이지가 뜨면 성공이다 !
<br><br>
**4\. 추가 설정**

\[file\] - \[settings\] 을 누르고 위창에서

Build, Execution, Deployment의 Gradle 탭에서 Build and run using과 Run tests using을 Gradle 이 아닌 인텔리제이로 변경하면 gradle을 통하지 않고 바로 바로 실행할 수 있기 때문에 보다 빠르게 실행할 수 있다.
<br><br><br><br><br><br>
### ➡️ 라이브러리 살펴보기

### ➡️ View 환경설정


### ➡️ 빌드하고 실행하기
