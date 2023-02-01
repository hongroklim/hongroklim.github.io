---
date: 2021-03-29 23:45:00 +0900
title: Oracle PIVOT 함수 이해하기
excerpt: 매번 구글에 문법 찾지말고 이 기회에 씹어먹자
categories: bigdata
---

## PIVOT 함수가 필요한 순간

개발을 하다보면 저장된 자료의 모양과 보여줄 자료의 모양이 일치하지 않기도 한다. 수상자 목록을 저장하고 있는 아래의 테이블을 예시로 들어보겠다.

| 연도 | 분기 | 순위 | 대상자 |
|:---:|:---:|:---:|:---:|
| 2020 | 1 | 1 | A |
| 2020 | 1 | 2 | B |
| 2020 | 1 | 3 | C |

이런 식으로 저장되어 있는 데이터를

| 연도 | 분기 | 1등 | 2등 | 3등 |
|:---:|:---:|:---:|:---:|:---:|
| 2020 | 1 | A | B | C |

이렇게 한 눈에 보여줄 떄가 있다. 말하는 건 쉬워보이지만 실제로는 간단한 일이 아니다. 3줄로 나오던 걸 1줄로 압축해야 하고, 그러려면
테이블에도 없는 열을 만들어서 보여줘야 한다.

### `PIVOT` 등판
 
위의 모습대로 그리고자 일단 조건을 붙이면서 쿼리를 만들려는데, 새로만들 column에 일일이 손을 봐야 하니 난감할 따름이다.
놀랍게도 Oracle DBMS는 이러한 일들을 할 수 있는 함수가 있는데, 그게 바로 `PIVOT` 함수이다. 그러니 어떻게 구현할 지는 `PIVOT`이 알아서 할테니
신경 안 써도 된다. 이제 남은 건 내가 하고싶은 일이 무엇인지 쿼리로 정확히 표현하기만하면 된다.

## `PIVOT`에 대한 이해

`PIVOT`이라는 것은 *행을 열로 바꾸는 것*이다. 하지만 이 말 하나로 `PIVOT`을 자유자재로 쓰긴 어렵다. `PIVOT`에 어떤 인자를 넣어야 하는지,
또 정확히 SELECT문에 어떤 영향을 미치는지 쉽게 떠올려지지 않기 때문이다. `PIVOT`으로 달라진 조회결과를 보면서 `PIVOT`에게 어떤 정보가 필요한지
알아보자.

### 행을 열로 바꾼다는 건

`PIVOT`으로 조회결과의 column에 변화가 생기는데, 이 column들을 3가지로 구분할 수 있다. 이들을 각각 @, $, #이라는 임의의 기호로 정리했다.

- **@** : 새롭게 생성되는 *PIVOT*의 `column 이름`이 된다
- **$** : 새롭게 생성되는 *PIVOT*의 `데이터`가 된다
- **#** : 위치는 `변화없다`

그림으로 나타내면 다음과 같다.

| | | | | \> | | | @ | @ | @ |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| #1 | #2 | @ | $ | | #1 | #2 | $ | $ | $ |
|#1 | #2 | @ | $ | | | | | | |
|#1 | #2 | @ | $ | | | | | | |

세로 모양에서 가로 모양으로 바뀌는 column은 **@** 와 **$** 이고, 자연스럽게 나머지 column들은 #이 된다.
**@** column 데이터는 데이터가 아니라 column 이름 역할을 맡아 새로운 column들을 만들어내고,
**$** column 데이터는 새로 만들어진 column들의 데이터 자리로 이동하게 된다.
`PIVOT`에게 알려줄 정보로 행이 열로 바뀌는 두 가지의 column, 즉 **column 이름** 과 **데이터** 역할을 맡을 column에 관한 정보가 필요하다.

> `PIVOT`에게 필요한 정보
>> 1 . *`PIVOT` 테이블에서 **column 이름** 과 **데이터** 가 될 column*


### column값도 먼저 정한다

column 이름(위에서 보면 **@** 역할)만 정하면 될 줄 알았는데, 그 column에 해당하는 값도 정해야 한다. 처음엔 나는 이를 귀찮게 여겼다.
*그냥 @ column 이름만 가지고 알아서 값을 찾아가면 안 되나?* 하는 생각이었지만, PIVOT으로 만들 column을 먼저 만드는 이유를 깨달았다.
DBMS는 쿼리 실행 전 SQL Parser를 거치는데, 이 과정에서 문법(Syntax)상의 확인과 의미(Semantic)상의 확인을 한다.

<figure>
  <img src="https://i.imgur.com/b9YTuS5.gif"
       alt="content_01">
  <figcaption>SQL Parser 진행과정&emsp;/&emsp;<a href="https://docs.oracle.com/database/121/TGSQL/tgsql_sqlproc.htm">출처 : docs.oracle.com</a></figcaption>
</figure>

의미 상의 확인에서 column이 실제로 존재하는지 여부를 검증한 뒤에서야 쿼리가 실행되므로, 쿼리결과 데이터로 column을 정한다는 것은 순서에 맞지 않다.

> `PIVOT`에게 필요한 정보
>> 2 . ***column 이름**에서 명시적으로 생성될 column값*

### 데이터는 그냥 들어갈 수 없다

데이터(위에서 보면 **$** 역할)는 무조건 집계함수와 같이 있어야한다. 굳이 왜 그래야 하나 싶겠지만, 다시 수상자 목록을 예시를 들어보겠다.
만약 아래와 같은 데이터가 들어있는 테이블에 `PIVOT`을 사용한다면 어떻게 되겠는가?

| 연도 | 분기 | 순위 | 대상자 |
|:---:|:---:|:---:|:---:|
| 2020 | 1 | 1 | A |
| 2020 | 1 | 2 | B |
| 2020 | 1 | 3 | C |
| 2020 | 1 | 3 | D |

3등이 C와 D 2개의 데이터가 있어서 PIVOT은 스스로 결정하지 못 한다. Oracle DBMS 입장에서는 `PIVOT`은 일종의 집계함수이다.
`PIVOT`의 세 종류의 열 중에서 데이터가 될 **$** 빼고 나머지는 전부 `GROUP BY`절에 들어가게 된다. 그러므로 위의 예시에서는 
연도, 분기, 순위에 따른 대상자는 오로지 1명만 존재해야 한다는 것이다. 그러니 집계함수를 사용한다 치고 `COUNT()`, `MAX()`, `MIN()`, 'AVG()' 등을
함께 사용해야 한다.

> `PIVOT`에게 필요한 정보
>> 3 . ***데이터**를 표현할 집계함수*

## PIVOT 사용법

### 필요한 정보와 문법

지금까지 알아보았듯 `PIVOT`에게 필요한 정보를 정리하자면 다음과 같다.

1. `PIVOT` 테이블에서 **column 이름** 과 **데이터** 가 될 column
1. **column 이름**에서 명시적으로 생성될 column값
1. **데이터**를 표현할 집계함수

그리고 `PIVOT` 문법은 다음과 같다.
```sql
-- FROM ...
PIVOT (
    <집계함수>(**데이터**)
    FOR **column이름**
    IN (<column값 ...>)
)
-- WHERE...
```

필요한 정보와 `PIVOT`문법을 조합해 위의 수상자목록을 가지고 예시를 들어보겠다.

### 예시

위에서 알아본 내용을 바탕으로 `WINNER_LIST`라는 테이블을 `PIVOT`을 활용하여 조회하는 예시이다.

| YEAR | QUARTER | RANK | WINNER |
|:---:|:---:|:---:|:---:|
| 2020 | 1 | 1 | A |
| 2020 | 1 | 2 | B |
| 2020 | 1 | 3 | C |

| YEAR | QUARTER | 1등! | 2등! | 3등! |
|:---:|:---:|:---:|:---:|:---:|
| 2020 | 1 | A | B | C |

1. `PIVOT` 테이블에서 **column 이름** 과 **데이터** 가 될 column
   - 모든 column들을 생각할 필요가 없다. `PIVOT` 으로 변화된 부분만 살펴본다면 **column 이름**은
     `RANK`, **데이터**는 `WINNER`가 된다.
1. **column 이름**에서 명시적으로 생성될 column값
   - `RANK`로 생성될 값들은 총 3가지(1, 2, 3)이다.
1. **데이터**를 표현할 집계함수
   - 어차피 1건만 가져올건데 `MAX()` 함수를 사용할 것이다.
    
이를 `PIVOT` 문법에 대응시키면 다음과 같은 SQL을 만들 수 있다.

```sql
SELECT *
  FROM WINNER_LIST
 PIVOT (
     MAX(WINNER)
     FOR RANK
     IN (1, 2, 3)
)
```

`PIVOT`을 사용할 때는 항상 찾아봤었는데, 이제는 이해했으니 더이상 찾지 않아도 될 것 같다.

## 심화

기본적인 `PIVOT` 문법만 알고 있다면 구현에는 아무런 문제가 없을 것이다. 그걸 넘어 `PIVOT`을 알아가는 과정은 좀 더 능숙하게
`PIVOT`을 사용하고자 하는 데 목적이 있다.

### 여러 column을 데이터로 활용하기

행을 열로 만들고 싶은 건 여러개일 때가 있다. 그렇다면 **column 데이터**를 필요한 만큼 선언하면 된다.

```sql
SELECT *
  FROM WINNER_LIST
 PIVOT (
    MAX(WINNER), MAX(SCORE) /* 집계함수와 함께 여러 column을 지정할 수 있다 */
    FOR RANK
    IN (1, 2, 3)
)
```

### 별칭(Alias) 사용하기

별칭(Alias)은 총 세 곳에 사용할 수 있다. column 데이터(집계함수 쓰는 곳), column 값(IN 절) 그리고 PIVOT 전체.

```sql
SELECT 
    /* B.YEAR, B.QUARTER, */ -- 사용불가
       A.YEAR, B.QUARTER,    -- 사용가능
       B.FST_WINNER, B.FST_SCR,
       B.SEC_WINNER, B.SEC_SCR,
       B.TRD_WINNER, B.TRD_SCR
  FROM WINNER_LIST A -- 사용불가
 PIVOT (
    MAX(WINNER) AS WINNER, MAX(SCORE) AS SCR /* 1. column 데이터 */
    FOR RANK
    IN (1 AS FST, 2 AS SEC, 3 AS TRD) /* 2. column값 */
) B /* 3. PIVOT */
/* WHERE A.YEAR = '2021' */ -- 사용불가
WHERE B.YEAR = '2021'       -- 사용가능
```

`SELECT` 부분을 보면 column값의 alias와 column 데이터의 alias가 snake case(_ 붙이기)로 이어지는 것을 볼 수 있다.
그리고 주의할 점으로 `FROM`에 있는 A라는 alias는 사용이 불가능한데, PIVOT을 거치면서 그 위에 선언되었던 모든 column들은
`PIVOT`으로 흡수되는 모양이기 때문에 B라는 alias만 사용가능하다. 마찬가지로 `WHERE`에서도 A 대신 B를 사용해야 한다.

### 동적으로 column값 설정하기

column값(`IN` 부분)은 쿼리에 직접 정의할 필요가 있지만, `PIVOT XML`은 그럴 필요 없이 동적으로 정의 가능하다.
`PIVOT` 옆에 `XML`을 붙이고 `IN` 안에는 subQuery를 작성하면 된다.

```sql
SELECT YEAR, QUARTER, RANK_XML
  FROM WINNER_LIST
 PIVOT XML ( -- XML 붙이기
    MAX(WINNER) AS WINNER, MAX(SCORE) AS SCR
    FOR RANK
    IN (SELECT distinct RANK FROM WINNER_LIST) -- SUB_QUERY 사용가능
)
```

이를 통해 `PIVOT`으로 생성되는 값들은 전부 `CLOB` 타입의 `RANK_XML`으로 들어가게 되므로 쿼리 입장에서도 유효한 column인지
판단할 수 있게된다. `RANK_XML`의 모양은 다음과 같다.
```xml
<PivotSet>
    <items>
        <item>
            <column name="RANK">1</column>
            <column name="WINNER">A</column>
            <column name="SCR">98</column>
        </item>
        <item>
            <column name="RANK">2</column>
            <column name="WINNER">B</column>
            <column name="SCR">92</column>
        </item>
        <item>
            <column name="RANK">3</column>
            <column name="WINNER">C</column>
            <column name="SCR">90</column>
        </item>
    </items>
</PivotSet>
```

이를 잘 parsing 한다면 유용하게 사용할 수 있을 것 같다. 그 방법은 다음 포스팅에서 소개할 예정이다.
