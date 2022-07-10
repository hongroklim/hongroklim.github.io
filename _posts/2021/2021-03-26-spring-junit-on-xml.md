---
date: 2021-03-26 13:20:00 +0900
title: Spring JUnit으로 Test를 가볍고 유연하게
excerpt: 꽉 막힌 Spring Framework XML 설정환경에서 Java Configuration으로 테스트를 구사하는 전략
header:
  overlay_image: https://user-images.githubusercontent.com/59322692/112853559-efcd8200-90e7-11eb-9d66-1ec8e99b1c6a.png
  caption: "&copy; [**spring**](https://spring.io), [**junit**](https://github.com/junit-team/junit4)"
categories: java
---

## 테스트를 위한 Context의 필요성

### 테스트는 유용하다

테스트 기반 개발(TDD; Test Driven Development) 은 소프트웨어 산업 내에서 인정받는 개발 방법론이다. 그리고 나 또한 테스트 코드를 작성하는 것을 긍정적으로 생각한다. 내가 느낀 테스트의 장점 중 하나로 매번 서버를 내렸다 올릴 필요 없이 방금 작성한 코드를 빠르고 편리하게 확인할 수 있다는 것이다. 그렇다면 대형 프로젝트에서 테스트를 실행한다면 어떨까? 전체 코드 규모가 클 수록 조금씩 확인하는 테스트가 더욱 유용해 보일 수 있다. 하지만 막상 테스트를 실행하려면 그렇지 않다.

### 운영환경 그대로 설정한다면

프로젝트 규모가 크면 클 수록 테스트는 운영환경 설정 그대로 테스트를 실행하게 된다면 그 거대한 Application Context를 힘겹게 올릴 때 모든 Bean들을 등록했지만 실제 테스트가 수행될 때는 고작 몇 개의 Bean들만 사용될 것이다. 그렇다고 테스트 서비스 딱 한 개만 등록해버린다면 너무나도 작은 단위에서만 코드가 문제없음을 보장하게 된다. 테스트를 제대로 활용하기 위해서는 테스트에 맞는 Context 설정이 필요하다.

## 테스트는 Java로

### 테스트 하기에 부적합한 XML

프로젝트 설정이 XML이라면 테스트 설정은 더이상 XML일 수 없다. 그 이유는 테스트가 읽어들일 context 설정은 유연하고 편리하게 고칠 수 있어야 하기 때문이다. 가볍게 테스트를 실행하기 위해 매번 어렵게 설정을 바꿔야 한다면 수고스러운건 마찬가지가 된다. 더욱이 개발자 스스로 편리하게 테스트를 설계하고 실행할 수 있어야 한다는 것은 결코 타협할 수 없는 기본이다.

### Java Configuration으로 하려는 이유

테스트를 Java Configuration으로 하려는 이유는 XML 설정보다 더 좋아서 하는 게 아니고, ApplicationContext이 사용되는 상황 때문이다. 운영환경이라면 자주 바꿀 일이 없기도 하고, 급하게 바꿀 일이 있다면 build없이 수정할 수 있는 XML이 적합해보인다. 하지만 테스트에서는 다르다. IDE에서는 자유롭게 build가 되기도 하고, 테스트를 할 때마다 설정을 수정해야 하니 Java로 설정하는 것이 더 낫다.

## XML 위에서 Java로 설정하는 전략

XML 설정 통째로 Java Configuration 형태로 갈아넣진 않는다. 테스트 상황에 맞추어 내 입맛대로 수정할 부분만 골라 Java Configuration으로 올려보자.

### 수정할 필요가 없는 건 그냥 가져오기

특정 서비스에 관계없이 존재하는 Component들이 있다. Authorization이나 Transaction과 같은 설정은 건드릴 필요가 없다. 이러한 XML파일들을 먼저 설정하자.

```java
@ImportResource({“*.xml”, “*.xml”})
public class SpringMvcTest {
// ...
}
```

### 주요 설정사항은 별도로 관리하기

특정 서비스는 특정 설정에 의존한다. 예를 들면 `com.company.account`라는 패키지의 서비스들은 `component-scan`과 Mybatis의 `mapperLocations` 등에 해당 패키지가 포함되어야 Spring에서 사용할 수 있다. 이를 테스트 입장에서 본다면, 테스트 할 서비스만 설정에 올리고, 그렇지 않은 것은 설정에서 제외시킨다면 테스트를 가볍게 만들 수 있다는 것이다. 특정 서비스를 사용하기 위한 설정들은 XML 파일 곳곳에 흩어져있는데, 이를 하나로 합쳐서 관리하자.
```java
// 테스트 대상을 모은 Class
public class ConfigConstant {
    // 테스트 Service
    public static final Class<?>[] SERVICE_CLASSES = {*.class, *.class};
    // 테스트 Controller
    public static final Class<?>[] CONTROLLER_CLASSES = *.class;
}
```

### 필요한 Component만 등록하기

`component-scan` 설정은 기본적인 탐색단위가 package이다. `component-scan` 설정에서 `classes`로 *.class만 입력했다면 `ApplicationContext`는 해당 클래스 뿐만 아니라 그 클래스가 위치한 패키지 전체를 탐색하고 전부 등록하게 된다. 내가 필요한 `ConfigConstant.SERVICE_CLASSES`만 등록하려면 Component-scan에 CustomFilter를 설정해주자.

```java
@ComponentScan(
    customFilter=*.class
)
``` 

```java
public class CustomServiceFilter {

}
```

### 중복된 설정 줄이기

테스트할 서비스 목록은 여러 설정에서 쓰일 수 있다. 만약 서비스의 패키지 아래 `mapper/`라는 디렉토리로 Mybatis의 SqlMapper들이 위치한다면  `SERVICE_CLASSES`목록을 MapperLocations 설정에도 활용할 수 있다. 기존 XML 설정에서는 path를 별도로 입력해야 했다면, Java Configuration에서는 그럴 필요가 없다.

```xml
<!—- XML 설정 시 —->
<property key=“mapperLocations”>
    <list>
	    <value>classpath:/com/company/account/mapper/*.xml</value>
    </list>
</property>
```

```java
// 서비스 패키지를 통해 Mapper의 경로를 가져온다
private String getMapperPath (String servicePackage) {
    String path = “”;
    // TODO servicePackage 아래 mapper/*.xml 파일 설정
    return path;
}

// Mybatis의 MapperLocations를 가져온다 
private Resource[] getMapperLocations () {
    List<String> list = new ArrayList<>();
    for(String serviceClass : ConfigConstant.SERVICE_CLASSES) {
        list.add(getMapperPath(serviceClass.getPackageName()));
}
    return list.toArray(Resource::new);
}

// TODO bean 등록
public 
```

### 테스트를 위한 인증 수행하기

서비스 안에서 현재 접속한 사용자의 정보를 활용할 일이 있다면 테스트를 위한 인증과정을 별도로 수행해야 한다. 그럴 때는 Spring Security의 context를 통째로 가져오기 보단 `AuthenticationProvider` 내에서 사용할 `Authenticator` 하나만 `Bean`으로 등록하고 사용하면 된다.

```java
public Authenticator companyAuthenticator() {
    return new CompanyAuthenticator();
}
```

이를 Test 안에 있는 @Before 에서 활용하자.

```java
@Before
public void authentication() {
    // TODO Auth
}
```

## Constant의 한계

Java Configuration이 XML보다 대체적으로 유연하게 설정할 수 있었지만, 그래도 Annotation 기반이다 보니 설정하면서 몇가지 아쉬웠던 순간은 있었다.

### 정적인 basePackage

basePackage 설정의 구상은 다음과 같았다. `SERVICE_CLASSES` 목록을 받아 이들을 모두 포함할 수 있는 가장 작은 단위의 패키지 목록을 만드는 것이었다. 하지만 basePackage는 상수형태의 값만 받기 떄문에 동적으로 선언할 수 없었다. 만약 그게 가능했다면 더 적은 범위의 패키지만 탐색하였을텐데 말이다.

### 하나의 Configuration

이게 가장 큰 단점인 것 같다. 여러 Test Class별로 `ConfigConstant`를 사용했다면 편리했을텐데, 지금 설정으로는 한 번에 하나의 테스트설정만 저장이 가능하다. 다른 테스트를 실행할 때는 필수적으로 그에 해당하는 `ConfigConstant`로 바꿔야 한다는 약간의 불편함이 있다.
