---
date: 2022-09-15 16:00:00 +0900
title: Scala로 '같은 값이면 싼걸 사죠' 구현하기
excerpt: 리스트 자료형을 효과적으로 다루는 방법을 생각해보자
categories: java
---

## 계기

얼마 전에 예능을 보고 있었는데 ‘이어서 말해요’ 게임을 하고 있었다. 여러 속담들이
나오고 있던 중 안유진 차례에 다음 문제가 나왔다.

<figure>
  <img src="https://i.imgur.com/b05Enxa.jpg"
       alt="yoojin_all_time_legends">
  <figcaption>나PD를 당황시킨 안유진의 대답&emsp;/&emsp;<a href="http://program.tving.com/tvn/tvneartharcade">출처 : Tving</a></figcaption>
</figure>

그걸 들은 멤버들도, 나PD도, 나도 순간 멍- 해졌다. 너무나도 신박한 저 말을
곱씹던 중 "프로그래밍언어로 표현해보면 어떨까"하는 생각이 들었다.

## 명제 분석하기

본격적으로 구현을 시작하기 전, 저 말을 분해해보자.

* 같은 값이면 싼걸 사죠
* **같은 값**(Equality)이면 **싼걸 사죠**(Comparison)

저 논리가 적용되는 상황을 그려보면 다음과 같다.

* 한 명의 **구매자**와 여러 개의 **물건**
* 모든 물건의 가격이 같은 경우 구매자는 가장 낮은 가격의 물건을 선택

저 논리를 구현하는 데 필요한 객체는 구매자와 물건이고, 메서드는 '모든 물건의
가격이 같은 지 확인하기'와 '가장 낮은 가격의 물건을 선택하기'가 필요하다는
것을 알 수 있다.

## Class와 Trait 만들기

구매자와 물건을 정의해보면 다음과 같다.

```scala
trait Buyer {
  /* Return true when all the prices are same. */
  def areEquals(goods: List[Good]): Boolean

  /* Get a good which is the lowest price. */
  def cheapest(goods: List[Good]): Good
}

case class Good(val name: String, val price: Int) {
  override def toString(): String = s"$name ($price)"
}
```

`Buyer`는 모든 물건의 값이 구분하는 `areEquals`와 가장 낮은 가격의 물건을
고르는 `cheapest` 메서드가 있다. `Good`는 `name`과 `price` 필드 만을 갖고
있다.

이를 활용하여 '같은 값이면 싼걸 사죠'는 다음과 같은 함수로 나타낼 수 있다.

```scala
final def purchase(goods: List[Good]): Good = {
  if (goods.isEmpty) throw new UnsupportedOperationException("empty.purchase")
  else if (areEquals(goods)) cheapest(goods)
  else goods.head
}
```

## Object 구현하기

Signature는 만들었으니 저 메서드를 구현해보자.

### 1. 함수형 패러다임 적용하기

`@tailrec`한 helper-function을 활용하여 구현해보았다.

```scala
object FunctionalBuyer extends Buyer {

  def areEquals(goods: List[Good]): Boolean = {
    @tailrec
    def iter(aPrice: Int, goods: List[Good]): Boolean = {
      if (goods.isEmpty) true
      else if (aPrice != goods.head.price) false
      else iter(aPrice, goods.tail)
    }

    iter(goods.head.price, goods.tail)
  }

  def cheapest(goods: List[Good]): Good = {
    @tailrec
    def iter(min: Good, goods: List[Good]): Good = {
      if (goods.isEmpty) min
      else {
        val lower = if(min.price > goods.head.price) goods.head else min
        iter(lower, goods.tail)
      }
    }

    iter(goods.head, goods.tail)
  }
}
```

`areEquals`가 return하게 되는 상황은 'goods가 비어있는 경우'와 'aPrice와 head의
price가 다른 경우'가 있다. `cheapest`는 호출될 때마다 min 값을 업데이트 여부를
판단하고 저장한다.

### 2. List 내장함수 적용하기

일일이 모든 매서드를 구현하기 보다 이미 만들어진 것들을 활용하는 게 더 낫다.
그러기 전에 `Good`를 가격 기준으로 comparable하게 만들 필요가 있다.

```scala
case class Good(val name: String, val price: Int) extends Ordered[Good] {
  def compare(that: Good): Int = this.price.compare(that.price)
  override def toString(): String = s"$name ($price)"
}
```

`Ordered[A]` trait를 상속 받고 `compare` 함수를 price 필드를 이용하여 구현해 준다.
이를 바탕으로 `Buyer`를 구현하면 다음과 같다.

```scala
object ListBuyer extends Buyer {
  def areEquals(goods: List[Good]): Boolean =
    goods.tail.count(_.compare(goods.head) != 0) == 0

  def cheapest(goods: List[Good]): Good = goods.min
}
```

저 `count`와 `min`의 세부 구현을 살펴보면 var와 while-loop가 쓰이고 있다는
것을 알 수 있다. Scala는 처음부터 끝까지 함수형으로 만들어진 줄 알았는데 그건
아닌가 보다.

### 3. Parallelism 적용하기

Scala를 사용하고 있는 만큼 병렬처리가 가능한 형태로도 구현할 수 있다.

```scala
object ParallelBuyer extends Buyer {
  def areEquals(goods: List[Good]): Boolean =
    goods.tail.par.forall(_.compare(goods.head) == 0)

  def cheapest(goods: List[Good]): Good = {
    def getMin(x: Good, y: Good): Good = if (x.compare(y) > 0) x else y
    goods.par.reduce(getMin)
  }
}
```

`forall` 메서드의 중간에 false가 나와도 안 멈출까봐 걱정했는데, 세부 구현을
살펴보니 중간 결과값이 false인 경우 abort signaling을 하게 되어 있었다.
`reduce`를 할 때 사용하는 splitter의 경우 compare 함수를 활용하여 구현하였다.

## 결론

지금까지 세 가지의 방법으로 '같은 값이면 싼걸 사죠'를 Scala로 표현해 보았다.
함수형 프로그래밍 언어의 장점으로 '논리 구조를 간결하게 표현할 수 있다'를 꼽을
수 있었는데, 그렇다고 해서 모든 메서드를 처음부터 만들면 코드가 다소
지저분해질 수 있다.

`min`이라는 리스트형 내장함수가 있음에도 `reduce`를 활용하기 위해 `getMin`을
직접 정의했다. 나중에 안 사실이지만 `ParSeq`에도 `min`이 정의되어 있다는 것을
발견하였다. 저 메서드에서는 내부적으로 `lteq`를 사용하고 있으니 처음에
`Ordering`만 잘 정의했으면 수월하겠구나라는 생각이 들었다.

<figure>
  <img src="https://i.imgur.com/yVVosT7.jpg"
       alt="yoojin_dm">
  <figcaption>특별한 미앤유~&emsp;/&emsp;<a href="http://program.tving.com/tvn/tvneartharcade">출처 : Tving</a></figcaption>
</figure>
