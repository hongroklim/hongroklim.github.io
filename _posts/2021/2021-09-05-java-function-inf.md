---
date: 2021-09-05 23:28:00 +0900
title: 어쩌다보니 함수형 프로그래밍 비슷한걸 하는 중
excerpt: Java의 Lambda와 Functional Interface를 이해하는 의식의 흐름
categories: java
---

## Stream을 막 쓰던 중에

작년 초에 내가 유지보수하던 Spring Framework의 버전을 JDK 1.6에서 Java 8로 올릴거라는 소식이 들려왔다.
그 소식을 들은 나는 "그런가보다..." 하고 있었지만, 다른 동료들은 "드디어 Java 8을 쓸 수 있게 되는구나!"하는 기대감에 부풀어 올라 차 있는 모습이었다.
Java 8에서 도대체 어떤 변화가 있는지 궁금했던 나는 찾아보았고, 여러 변화들 중에서 가장 먼저 **Stream**이 생겼다는 것을 알게 되었다.

### Stream의 단순 활용법

JDK 1.6에서 Java 8로 Spring 소스코드를 컴파일하고 오류가 있는지 확인하던 중, 이왕 업그레이드 하는 김에 Java 8을 코드에도 적용해보면 어떨까하는
생각이 들었다. 그리하여 Stream을 한번 써보기로 결정했고 젹용한 부분은 다음과 같다.

```java
// JDK 1.6
private List<User> getAdminUsers(List<User> userList){
    // 사용자 목록을 필터링하여 가져온다
    List<User> list = new ArrayList<>();

    for(User user : userList){
        // 관리자로 지정된 사용자만 추가
        if(user.isAdmin()){
            list.add(user);
        }
    }
    
    // 목록을 반환한다
    return list;
}
```

```java
// Java 8
private List<User> getAdminUsers(List<User> userList){
    return userList.stream()                // Stream을 활용
        .filter(user -> user.isAdmin())     // 관리자인 사용자만 필터링
        .collect(Collectors.asList());      // 목록을 반환
}
```

잘 보면 Java 8에서는 코드 한 줄로 해결이 된다! 뭔가 되게 세련된 느낌이면서도 훨씬 간결해진? 그런 인상을 받았다.
소스코드에 목록을 다루는 부분이 많이 등장하는데, 이들을 훨씬 정갈된 방법으로 구현할 수 있을 것 같다는 기대가 들었다.
Stream이 `#filter()` 메서드 이외에 또 다른 기능이 존재하는지 찾아보았다.

### filtering 말고도 다양한 활용

`#forEach`도 Stream에서 활용할 수 있다.

```java
public void trimNames(){
    userList.stream()
        .forEach(user -> user.setName(user.getName().trim()));
}
```

모든 User 객체에서 `#getName`을 통해 이름 목록을 구하는 것 또한 Stream으로 간단하게 할 수 있다.

```java
private List<String> getNames(){
    return userList.stream()
        .map(user -> user.getName())
        .collect(Collectors.toList());
}
```

## Functional Interface의 등장

이런저런 Stream이 지원하는 메서들을 찾아보던 중, 문득 이들의 파라미터가 독특하다는 것을 느꼈다.
일반적으로 파라미터로 전달하는 것은 `field`이지만, 이들은 `method`를 전달하고 있었다.

| method | parameter | action |
|:---:| --- | --- |
| map | `Function<? super T, ? extends R> mapper` | T를 받고 R을 반환 |
| forEach | `Consumer<? super T> action` | T를 받음 |
| filter | `Predicate<? super T> predicate` | T를 받고 boolean을 반환 |

이들의 파라미터는 어떤 **정보** 그 자체가 아니라, 정해진 파라미터 또는 리턴 타입을 따르는 **행위**라는 것이다.
과연 저 타입은 어떻게 생겨먹었길래 메서드를 전달할 수 있는지 궁금해졌다.

### 메서드 하나만 구현하면 되는 Interface

저 파라미터 타입 중에서 [Function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)
에 대해 찾아보았다. 이는 단순히 `interface`로, 일반적인 방식으로 구현할 수 있었다. `Function`을 활용하여 `#getNames`를 구현해보면 다음과 같다.

```java
public class GetNameFunc implements Function<User, String> {
    @Override
    public String apply(User user) {
        return user.getName();
    }
}
```

이처럼 하나의 메서드만 `@Override`하는 Interface들에게 [@FunctionalInterface](https://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html)
를 붙여준다고 한다. 이러한 FunctionalInterface들은 **하나의 메서드**만 갖게 되므로 Stream에서는 이것을 invoke하게 된다.

**하나의 메서드?** : `Function` 인터페이스를 보면 `#apply` 이외에도 `#compose`, `#andThen`, `#identity`가 있다. 이들은 모두
인터페이스 수준에서 이미 구현이 된
[**Default Methods**](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html) 이다. 
그러니 "구현해야 할 메서드가 하나뿐인" 이라고 하는 것이 좀 더 정확하다.
{: .notice}

## Lambda는 또 뭐지

여기서 또 흥미로운 점은 그동안 FunctionalInterface를 쓸 때 `new` 와 같은 생성자를 쓰지 않았다는 것이다.
대신에 `->` 라는 기호로 간단히 매서드를 선언했을 뿐이다. 여러 줄에 걸쳐서 할 일을 이렇게 한 줄로 끝낼 수 있었던 것은
바로 `Lambda Expression` 덕분이었다.

### 이름없는 함수 만들기

위의 `GetNameFunc`을 참고해보면, `#apply`는 User를 파라미터로 받고, String을 반환한다. 이 메서드를 Lambda Expression으로
표현하면 다음과 같다.

```java
// 일반적인 형태의 메서드
public String getUserName(User user) {
    return user.getName();
}

// Lambda Expression
// (파라미터) -> { body }
(user) -> { return user.getName(); }
// map 에서는
user -> user.getName()
```

이런 식으로 한 줄에 함수 선언이 끝나게 되었다. 기존 @Override로 메서드를 정의할 떄와 완전히 다르다.
보다시피 class, 생성자, 메서드 등 여러 줄의 코드로 함수를 정의하고 사용하던 것을, 몇 개의 기호와 단어만으로 똑같은 역할을 할 수 있었다.
특히 Stream 같이 chain 형식으로 여러 함수들을 정의하는 상황이라면 이 차이는 극대화 될 것이다.

## 함수형 프로그래밍에 다가가는 중

그렇다면 왜 Java는 Lambda Expression을 만들었을까? 단순히 코드가 줄었다는 사실을 넘어, _이러한 형태로 코드를 작성하려 한다_
는 점을 주목해보자.

### 요즘 뜨고 있는 패러다임

프로그래밍을 하는 방법은 계속 발전하는 중이다. 기존 프로그래밍에 존재하는 문제들을 해결하기 위해 다양한 개념들이 연구되고,
유용성을 인정받은 기법들은 자주 쓰이게 된다. 이러한 활용의 수준을 넘어서 프로그래밍 자체에 대한 관점의 변화가 나타나기도 하는데, 이는
프로그래밍 패러다임에 대한 변화를 의미한다.

최근에는 **함수형 프로그래밍**이 대세다. 이는 수 많은 함수들을 가지고 문제를 해결하는 기법이다.
이러한 폭발적인 수요에 대응하고자 손쉽게 함수를 정의하고 사용하기 위해 Lambda Expression을 만들었다고 봐도 무방하다.
앞서 볼 수 있듯, 물론 객체지향 프로그래밍에서도 함수를 사용할 수 있다. 하지만 함수를 사용했다고 해서 함수형 프로그래밍이라고 말하지 않는다.

### 함수형 프로그래밍처럼 생각해보기

함수형 프로그래밍을 한다는 것은 기존 프로그래밍을 할 때와 다른 관점이 필요하다.
만약 모든 admin의 name을 구하는 메서드를 작성한다고 할 때, 평소 하던 대로 생각한다면 다음과 같이 작성할 것이다.

```java
private List<String> getAdminNames(){
    List<String> nameList = new ArrayList<>();
    
    // 모든 사용자에 대해
    for(User user : userList){
        // 사용자가 admin일 때
        if(user.isAdmin()){
            // 사용자의 name을 구한다
            nameList.add(user.getName());
        }
    }
    
    return nameList;
}
```

절차적으로 작성되어 있는 코드를 함수형 스타일로 설명하자면 다음과 같다.
* `#getAdminNames`라는 함수에게 두 가지 방법을 알려주어야 한다.
  1. 누가 admin인지 구별하는 방법
  2. 어떻게 name을 가져오는 방법
* 각각의 방법은 `#isAdmin`과 `#getName`이라는 **순수함수**로 설명할 수 있다.
* 이러한 함수들을 모든 사용자에 적용할 때 나오는 결과를 구하면 된다.

**순수함수?** : 어떤 함수에 같은 파라미터를 전달했다면 항상 같은 값을 반환하면서, 외부에 변화를 일으키지 않는 함수이다.
그러므로 언제 어떻게 이 함수를 다루던 지 간에, 같은 결과를 얻을 수 있다.
{: .notice}

이처럼 하나의 문제를 여러 함수로 나누어서 해결하는 사고가 함수형 프로그래밍이라고 할 수 있다.
시작은 Stream 메서드였지만, 어느새 다른 차원의 생각을 하고 있는 중이다.

## 더 읽을거리

* [java 8 람다식 소개](https://www.slideshare.net/gyumee/java-8-lambda-35352385) - slideshare
* [Java 8: Lambdas, Part 1](https://www.oracle.com/technical-resources/articles/java/architect-lambdas-part1.html) - Oracle
* [함수형 프로그래밍 언어에 대한 고찰](https://engineering.linecorp.com/ko/blog/functional-programing-language-and-line-game-cloud/) - Line Engineering
