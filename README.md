# 스프링 부트와 내장 톰캣
## 스프링 부트와 웹 서버 - 실행 과정
```java
@SpringBootApplication
public class BootApplication { 
    public static void main(String[] args) {
        SpringApplication.run(BootApplication.class, args);
    }
}
```
* 스프링 부트를 실행할 때는 자바 `main()` 메서드에서 `SpringApplication.run()`을 호출해주면 된다.
* 여기에 메인 설정 정보를 넘겨주는, 보통 `@SpringBootApplication` 애노테이션이 있는 현재 클래스를 지정해주면 된다.

이 단순해 보이는 코드 한줄 안에서는 수 많은 일들이 발생하지만 핵심은 2가지다.
1. 스프링 컨테이너를 생성한다.
2. WAS(내장 톰캣)를 생성한다.

## 스프링 부트와 웹 서버 - 빌드와 배포
스프링 부트 jar 분석
`boot-0.0.1-SNAPSHOT.jar` 압축 해제
* `boot-0.0.1-SNAPSHOT.jar`
  * META-INF
    * MANIFEST.MF
  * org/springframework/boot/loader
    * `JarLauncher.class`: 스프링 부트 `main()` 실행 클래스
  * BOOT-INF
    * `classes`: 개발자가 개발한 class 파일과 리소스 파일
    * `lib`: 외부 라이브러리
    * `classpath.idx`: 외부라이브러리 경로
    * `layers.idx`: 스프링 부트 구조 경로

## 스프링 부트 실행가능 Jar
Fat Jar는 하나의 Jar 파일에 라이브러리의 클래스 리소스를 모두 포함했다.
그래서 실행에 필요한 모든 내용을 하나의 Jar로 만들어 배포하는 것이 가능했다.
하지만 Fat Jar는 다음과 같은 문제를 가지고 있다.

Fat Jar의 단점
* 어떤 라이브러리가 포함되어 있는지 확인하기 어렵다.
  * 모두 `class`로 풀려있으니 어떤 라이브러리가 사용되는지 추적하기 어렵다.
* 파일명 중복을 해결할 수 없다.
  * 클래스나 리소스 명이 같은 경우 하나를 포기해야 한다.

실행 가능 Jar
스프링 부트는 이런 문제를 해결하기 위해 jar 내부에 jar를 포함할 수 있는 특별한 구조의 jar를 만들고 동시에 만든 jar를 내부 jar를 포함해서 실행할 수 있게 했다.
이것을 실행 가능 Jar (executable Jar)라 한다.
이 실행 가능 Jar를 사용하면 다음 문제들을 깔끔하게 해결할 수 있다.

* 문제: 어떤 라이브러리가 포함되어 있는지 확인하기 어렵다.
  * 해결: jar 내부에 jar를 포함하기 때문에 어떤 라이브러리가 포함되어있는지 쉽게 확인할 수 있다.
* 문제: 파일명 중복을 해결할 수 없다.
  * 해결: jar 내부에 jar를 포함하기 때문에 내부에 같은 경로의 파일이 있어도 둘다 인식할 수 있다.

Jar 실행 정보
`java -jar xxx.jar`를 실행하기 되면 우선 `META-INF/MANIFEST.MF` 파일을 찾는다. 그리고 여기에 있는 `Main-Class`를 읽어서 `Main-Class`를 읽어서 `main()` 메서드를 실행하게 된다.

`META-INF/MANIFEST.MF`
```MF
Manifest-Version: 1.0
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: hello.boot.BootApplication
Spring-Boot-Version: 3.0.2
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Spring-Boot-Classpath-Index: BOOT-INF/classpath.idx
Spring-Boot-Layers-Index: BOOT-INF/layers.idx
Build-Jdk-Spec: 17
```
* `Main-Class`
  * 우리가 기대한 `main()`이 있는 `hello.boot.BootApplication`이 아니라 `JarLauncher`라는 전혀 다른 클래스를 실행하고 있다.
  * `JarLauncher`는 스프링 부트가 빌드시에 넣어 준다. `org.springframework/boot/loader/JarLauncher`에 실제 포함되어 있다.
  * 스프링 부트는 jar 내부에 jar를 읽어들이는 기능이 필요하다. 또 특별한 구조에 맞게 클래스 정보도 읽어들여야 한다. 바로 `JarLauncher`가 이런 일을 처리해준다. 이런 작업을 먼저 처리한 다음 `Start-Class`에 지정된 `main()`을 호출한다.
* `Start-Class`: 우리가 기대한 `main()`이 있는 `hello.boot.BootApplication`이 적혀있다.

스프링 부트 로더
`org/springframework/boot/loader` 하위에 있는 클래스들이다.
`JarLauncher`를 포함한 스프링 부트가 제공하는 실행 가능 Jar 실제로 구동시키는 클래스들이 포함되어 있다. 스프링 부트는 빌드시에 이 클래스들을 포함해서 만들어준다.

BOOT-INF
* `classes`: 우리가 개발한 class 파일과 리소스 파일
* `lib`: 외부 라이브러리
* `classpath.idx`: 외부 라이브러리 모음
* `layers.idx`: 스프링 부트 구조 정보

* WAR구조는 `WEB-INF`라는 내부 폴더에 사용자 클래스와 라이브러리를 포함하고 있는데, 실행 가능 Jar도 그 구조를 본따서 만들었다.
