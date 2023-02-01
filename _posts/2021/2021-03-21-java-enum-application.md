---
date: 2021-03-21 17:47:00 +0900
title: Java enum 실전 활용
excerpt: 조건문과 반복문 대신 나름 객체지향적인 개발을 해볼까 한다
categories: java
---

<figure>
  <img src="https://i.imgur.com/rJRpJ6A.png"
       alt="java">
  <figcaption>
  </figcaption>
</figure>

대형 SI 프로젝트에서 나는 평소대로 노가다스럽게 복사 붙여넣기를 하고 있었다.
그러다 기발한 아이디어가 떠올랐다. 어디 한번 색다르게 코드를 짜봐야겠다.

## 간략한 배경지식

예전에 `enum`을 활용한 코드를 보고 궁금해서 찾아본 뒤 '아 이런 문법도 있구나'하고 넘어갔다. 내가 `enum` 사용법을 설명하는 것 보다 훨씬
좋은 글들이 많이 있으니깐 그 때 찾아봤던 사이트 중 인상깊었던 글을 소개할까 한다.

- [Java enum 활용기](https://woowabros.github.io/tools/2017/07/10/java-enum-uses.html)
  * 이 글을 쓰는 데 가장 많은 영향을 끼친 글이다. 어쩜 그리 흥미롭게 글을 쓰시는지 부러울 지경이다. 단순한 사용법 차원을 넘어 서비스 관점에서
    개발을 바라보는 글이다. `enum`을 사용하는데 개발자 나름대로 합리적인 견해를 제시하고 의도한 바와 이로인한 장점을 명쾌히 설명하고 있다.

- [Java: enum의 뿌리를 찾아서](https://www.nextree.co.kr/p11686/)
  * Java 8에나 나온 문법인줄 알았는데, JDK 1.5부터 있었다고? 이 글을 통해 `enum`이 나름 근본있는 문법이란 것을 알게되어 좀 더 호감이 생겼다. `final`
    상수로 선언되었지만 코드 내에서 난잡하게 쓰이던 걸 `enum`이 깔끔하게 해결하는 나름 멋진 장면들을 보여준다.

`enum`에 관한 설명은 링크로 마무리하고 여기서부터는 나의 경험담으로 채워보겠다.

**일러두기** : 프로젝트에 있는 소스코드 그대로 안 긁어왔다. 나름대로 상황도 덧붙이고 꾸미고 비틀었으니 소설이라고 봐도 될 정도다.
그러니 현업 느낌만 ~~결코 현업 그대로가 아닌~~ 내고 가면 된다.
{: .notice}

## 처음엔 enum을 쓸 생각은 없었다

나는 기간에 따라 통계자료를 조회하는 기능을 만들고 있었다. 검색조건이라면 월, 분기 등을 고른 뒤 6월인지 혹은 1분기인지를 정하고 나서 
그에 따른 범위의 자료를 가져오는 기능이었다. 마침 프로젝트에서 사용되는 표준코드모음에 '기간코드'라는 것이 있었고, 이를 이용할 생각이었다. 
찍어내기 식의 개발을 하던 나는 머리보다 손이 먼저 움직이고 있었고, 내가 만들던 검색조건 처리방법은 다음과 같았다.

```java
// ...
List<String> monthList = new ArrayList<>();
if("09".equals(statCond)){              // 통계기준이 '월'일 때
    monthList.add(statDtlCond);
}else if("11".equals(statCond)){        // '분기'일 때
    if("01".equals(statDtlCond)){           // 1분기
        monthList.addAll(Arrays.asList("01", "02", "03"));
    }else if("02".equals(statDtlCond)){     //2분기
        monthList.addAll(Arrays.asList("04", "05", "06"));
    }
    // ...
}
// ...

//해당하는 월 목록에 따른 통계정보 반환
return getStatInfo(monthList);
```

어느정도 조건문들을 완성시키고 있었을 때쯤 정신을 차리게 되었다. 누가봐도 중복이 지나친 코드였고, 개발자라면 가장 먼저 지적할 부분이다 싶었다.
*그럼 어디서부터 손을 대지?* 라고 생각할 때 가장먼저 떠오른 건 *상수*였다.

## 먼저 상수로 표현해보기

### 코드잖아? 안 변하면 상수!

검색조건으로 사용하는 '기간코드'는 값과 이름으로 이루어진 코드이다. 그러니 아래와 같이 *코드값에 이름을 붙여주면 좋겠다* 라는 생각을 했다.

```java
private final String MONTH = "09";      // 월
private final String QUARTER = "11";    // 분기
private final String SEMIANNUAL = "12"; // 반기
private final String ANNUAL = "13";     // 연도
```

이렇게 상수로 코드값들을 표현하면 다음과 같은 장점이 있다.
- 다른 개발자 입장에서 `09`의 뜻을 해석하기 보다 `MONTH`로 읽는 것이 이해하기 더 편하다.
- 개발하면서 모든 코드값을 외울 필요 없이 IDE의 auto complete로 개발할 수 있다.
- 표준코드모음에서 코드값이 바뀌었을 때 `ctrl + F`로 모든 값을 찾으면서 바꿀 필요 없이 상수만 바꾸면 된다.

그럼 이제 위의 코드에 상수를 첨가해 보겠다.

```java
// ...
List<String> monthList = new ArrayList<>();
if(MONTH.equals(statCond)){             // 통계기준이 '월'일 때
  monthList.add(statDtlCond);
}else if(QUARTER.equals(statCond)){     // '분기'일 때
  if("01".equals(statDtlCond)){           // 1분기
    monthList.addAll(Arrays.asList("01", "02", "03"));
  }else if("02".equals(statDtlCond)){     //2분기
    monthList.addAll(Arrays.asList("04", "05", "06"));
  }
  // ...
}
// ...

//해당하는 월 목록에 따른 통계정보 반환
return getStatInfo(monthList);
```

음. 별로 달라진게 없어 보인다. 상수선언까지 했으니 4줄이 더 늘어난 게 가장 큰 변화같다. 다행히도 개선의 여지가 더 남아있다. 이건
통계기준(`statCond`)만 손을 본 거다.

### 일관성을 도려내보자

이젠 통계세부기준(`statDtlCond`)에 관심을 가질 차례이다. 세부기준에 따라 일일이 월을 나열하는 것 보다 반복문으로 묶어버렸으면 좋겠다.
통계기준별로 갖는 개월 수는 다음과 같다.

 통계기준 | 월 | 분기 | 반기 | 연도 
:---:|:---:|:---:|:---:|:---:
 개월 | 1개월 | 3개월 | 6개월 | 12개월 

통계기준별로 일관성을 찾았으니 이또한 마찬가지로 상수로 표현해보자.
```java
private final int MONTH_COUNT = 1;          // 월 : 1개월
private final int QUARTER_COUNT = 3;        // 분기 : 3개월
private final int SEMIANNUAL_COUNT = 6;     // 반기 : 6개월
private final int ANNUAL_COUNT = 12;        // 연도 : 12개월
```
분기(`QUARTER`) 개월 수는 무조건 3개월(`QUARTER_COUNT`)이라는 사실을 안다면 *통계세부기준에 따른 월 목록* 을 반복문으로 나타낼 수 있다.
```java
// 통계기준이 '분기'일 때
int month = Integer.parseInt(statDtlCond);  // 시작 월
int endMonth = month + QUARTER_COUNT -1;    // 종료 월 (개월 수 만큼 더함)
while(month <= endMonth)
  monthList.add(StringUtils.leftPad(month, 2, "0"));  //4월을 '04'이런 식으로 표현
  month++;
}
// ...
```

### 상수의 한계

그래도 고쳐지지 않는 문제는 여전하다. 바로 통계기준에 따라 개월 수를 직접 알려줘야 한다는 것이다. 사람 입장에선 통계기준 `MONTH`와
대응되는 개월 수는 `MONTH_COUNT`이다. 하지만 이건 코드 명명규칙에 따른 거지, 관습과 직관이 없는 컴파일러는 자연스럽게 이 둘을 연결시키지
못 한다. 촘촘하게 반복되는 조건문을 고치고 싶다는 본능이 앞서지만 아쉽게도 상수만으론 못 할 노릇이다.

## enum을 쓰면 어떨까?

### 맥락을 담을 수 있는 enum

enum은 조금 특별한 상수이다. 상수만으론 코드값과 개월 수가 분리되었지만, enum은 그러지 않아도 된다. 8개의 상수 *(4개의 통계기준, 4개의
통계기준에 따른 개월 수)* 를 하나의 `enum`으로 구현해 보았다.
```java
public enum PERIOD {
  MONTH("09", 1),       // 월
  QUARTER("11", 3),     // 분기
  SEMIANNUAL("12", 6),  // 반기
  ANNUAL("13", 12);     // 연도

  private String codeValue; // 코드값
  private int monthCnt;     // 개월 수
  
  // 생성자
  PERIOD(String codeValue, int monthCnt) {
    this.codeValue = codeValue;
    this.monthCnt = monthCnt;
  }
  
  // ...
}
```
생성자로 알 수 있듯 `MONTH`는 `codeValue`라는 속성으로 `09`, `monthCnt`라는 속성에는 `1`을 갖고 있다.
상수만으로 표현할 때는 `MONTH`와 `MONTH_COUNT` 각각으로 값을 가지고 있었지만, 이제는 하나의 `enum` 안에서 속성으로 나타낼 수 있게 되었다.
또한 어느 `PERIOD`인지 간에 해당하는 개월 수를 알고 싶다면 다른 통계기준들과 비교할 필요 없이 자신의 `monthCnt`만 확인하면 된다.
'MONTH'라는 이름과 '09'라는 코드값 그리고 1이라는 숫자를 `enum`이라는 맥락에 담아 표현할 수 있었다.

### 객체답게 사용해보자

객체는 상태 말고도 행동을 가질 수 있다. 통계세부기준(`statDtlCond`)에 따른 월 목록을 구하는 일을 `PERIOD`한테 시켜보자.
```java
public enum PERIOD {
  // ...
  public List<String> getMonthList(Object statDtlCond){
    List<String> monthList = new ArrayList<>();
    
    // 통계기준이 '연도'라면 세부기준은 필요없다
    if(this.equals(PERIOD.ANNUAL)){
      statDtlCond = null;
    }
    
    // 시작 월 구하기
    int month = Integer.parseInt(StringUtils.defaultString(statDtlCond, "1"));
    month = ((month-1) * this.monthCnt) + 1;
    
    // 종료 월 구하기
    int endMonth = month + this.monthCnt - 1;

    while(month <= endMonth){
      monthList.add(StringUtils.leftPad(month, 2, "0"));
      month++;
    }

    return monthList;
  }
  // ...
}
```
`PERIOD`가 자기 개월 수를 꺼내서 혼자 계산하는 모습, 얼마나 보기 좋은가? *객체의 상태에 따라 다르게 행동할 수 있다* 라는 객체지향의 특징을
잘 살릴 수 있었다. 또한 어쩌면 '기간코드'에서 가장 핵심이 되는 `시작 월 구하기` 계산식을 `PERIOD` 안에 묶어두었다는 점에서 *높은 응집도* 를
따를 수 있었다.

### enum = 상수 + 객체

`enum`은 조금 특별한 상수이자 객체이다. 일반적인 상수는 값만 담을 수 있지, 맥락을 담진 못 한다. 일반적인 객체는 인스턴스를 생성해야만
사용할 수 있다. `eum`을 통해 상수를 객체지향적으로 사용할 수 있다. 상수 입장에서 보면 속성과 행동을 표현할 수 있게 되었고, 객체 입장에서
보면 `Compile Time`에 결정된 값들을 인스턴스 중복생성 없이 효과적으로 사용할 수 있다. 상수와 객체 둘 다 필요한 접점에서 비로소 `enum`의
존재이유가 나타난다.

### enum PERIOD 소스코드

현업에서 사용한 `PERIOD` 전체 소스코드는 [GitHub Repository](https://github.com/rokong/hello-spring/blob/master/src/main/java/com/rokong/period/PERIOD.java) 에서
확인할 수 있다. 참고로 나는 `getMonthList()` 대신에 `startMonthOf()`와 `endMonthOf()`로 범위탐색으로 통계정보를 조회했다.

## (여담) 현업에 있다보니

막상 `enum`을 구현하고 나니 여러 생각이 떠오른다.

### 그냥 하던대로 해도 된다
처음 코드를 만든 것 처럼 꼭 enum을 활용해야 하는 건 아니다. 만약 상수로만 구현했다면 어땠을지 상상해본다. 타이핑 보다는 복사와
붙여넣기로 logic을 완성시켰을 것이다. 새로운 통계기준이 추가된다면? 귀찮게 여기며 연속된 조건문 끝자락에 새로 추가된 통계기준 하나
더 달아주고 끝났을 것이다.

### 굳이 그렇게 해야만 하나
기존 코드에 새로운 시도를 할 때 '멀쩡하게 돌아가는 걸 왜?' 라는 묻는다면 나를 할 말이 없게 만든다. 이 물음은 개발 막바지의 최적화 단계
뿐만 아니라 구현하는 중에도 떠오르고, 다른 사람이 아니더라도 나 스스로 하게되는 말이다. *다른 서비스들은 모두 상수로 표현했다면?* *다른 개발자들이
`enum`에 익숙하지 않다면?* 지금 내가 고안한 방식을 마냥 고집할 수는 없을 것이다.

### 그래도 해야지 뭐
개발이라는 것도 사람이 하는 일이다. 다들 저마다의 전문지식을 갖고 있고, 본인이 직접 실패하거나 만족스럽다고 느낀 경험들이 쌓여 *서로다른
배경의 사람들* 이 완성되는 것이다. 그래도 다들 공감하는 생각은 있다. *좋은 코드를 작성하자* . '좋다' 라는 말이 뭔지는 지금은 잘 모르지만
그래도 나는 그 방향으로 향해야겠다라는 생각은 든다.
