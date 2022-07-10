---
date: 2021-04-03 17:26:00 +0900
title: Java에서 PIVOT XML 파싱하기 (2/2)
excerpt: 그런데 이제 객체지항적인 리팩터링을 곁들인
categories: java
---

## 개선하고 싶은 것들

이전 포스트에서 `PIVOT XML`을 일반적인 `PIVOT`으로 조회했을 때와 같은 형태로 만들었다. 그렇지만 아직 더 하고싶은 일이 남아있다.

- `DAO`와 `PivotXMLReader`를 약하게 결합시키고 싶다.
- selectOne(한 행)으로 조회할 때 말고도 selectList(여러 행)으로 조회했을 때도 다루고 싶다.
- XML을 VARCHAR2 형식 말고도 CLOB와 BLOB로 조회했을 때도 다루고 싶다.

이러한 것들을 객체지향적으로 접근해 개선해볼 것이다.

## PivotParser

현재 DAO 안에는 `PivotXMLReader`를 쓰기 위해 수많은 코드들이 존재한다.

```java
// DAO
public Map<String, Object> getWinnerAndScore(){
    // VARCHAR2 형식으로 XML을 직렬화한 쿼리
    Map<String, Object> row = sqlSession.selectOne("getWinnerAndScoreAsVarchar2");
    
    // pivot column 가져와서 InputSource 만들기
    String xml = row.get("PRIZE_XML");
    InputSource inputSource = new InputSource(new ByteArrayInputStream(xml.getBytes(StandardCharsets.UTF_8)));
    
    // pivotRow 파싱하기
    PivotXMLReader pivotXMLReader = new PivotXMLReader();
    Map<String, Object> pivotRow = pivotXMLReader.parsePivotColumn(inputSource);
    
    // pivotRow 추가
    row.putAll(pivotRow);
    return row;
}
```
코드가 많다는 뜻은 `DAO`가 `PivotXMLReader`에 대해 알고있는 정보들이 많다는 것이고, 이러한 것들은 `DAO`의 주된 관심사와는 거리가 멀다.
두 객체 사이의 높은 결합도를 낮출 필요가 있다.

### 메서드 추가?

결합도를 낮추는 데 가장 간단한 방법은 `DAO`에서 `PivotXMLReader`와 관련된 모든 로직들을 `PivotXMLReader` 안의 메서드로
넣는 뒤에 DAO는 최소한의 메서드만 호출하는 방법이 있다. 그러나 *`PivotXMLReader`가 꼭 `DAO`에게만 쓰이는 건 아니지 않는가?*
`PivotXMLReader의` 역할은 XML을 Map 형태로 바꾸는 거지, 항상 DAO에서만 쓰일 필요는 없다. ~~다른 Service에서 XML을 전달받아 사용될 수도 있다~~
관련 메서드를 추가한다면 `PivotXMLReader` 입장에서도 DAO와 높게 결합되버린다.

### 샹속?

메서드를 추가하는 건 안되니 `PivotParser`라는 객체를 새롭게 만들 필요성을 느꼈다.
PivotXMLReader를 상속받아서 아래와 같은 parsePivotColumn을 사용해 보았다.

```java
public class PivotParser extends PivotXMLReader {   // PivotXMLReader를 상속받는다
    // ...
    private Map<String, Object> parsePivotColumn(String xml){
        // ...
        return parsePivotColumn(inputSource);   // 부모 객체의 public 메서드를 자유롭게 쓸 수 있다
    }체
}
```

막상 이렇게 하고 나니 *상속은 맹목적 사용에 따른 부작용이 너무 크다* 라는 [객체지향에 관한 책](https://wikibook.co.kr/object/)
의 말이 떠올랐다. 그 책에서 상속(is a) 말고 추천한 방법은 **합성(has a)** 이었다.

### 합성!

`PivotParser`라는 새로운 객체가 앞으로 해야 할 역할들을 생각해보면 여러 행 또는 단일 행 처리라던가, VARCHAR2, CLOB나 BLOB 등을
`PivotXMLReader`의 입맛이 맞게 바꾸기 등 나름 독립적인 객체로 봐도 될 정도였다. `PivotXMLReader`의 자식으로 남을 수준 까지는 아니었다.
`PivotParser` 안에 `PivotXMLReader`를 인스턴스 변수로 선언한다면 parsePivotColumn 메서드를 자유롭게 사용하는 건 마찬가지다.

```java
public class PivotParser {
    // 상속이 아닌 인스턴스 변수로 사용한다
    private PivotXMLReader pivotXMLReader = new PivotParser();
    // ...
    private Map<String, Object> parsePivotColumn(String xml){
        // ...
        return pivotXMLReader.parsePivotColumn(inputSource);    // 인스턴스 변수이니 자유롭게 메서드를 사용할 수 있다.
    }
}
```

### DAO와 결합도 낮추기

다시 `DAO`와의 관계로 돌아와서 관련 로직들을 `PivotParser`의 메서드로 뺄 차례다.

```java
public class PivotParser {

    private final String pivotColumn;       // row에서 PIVOT XML이 담긴 key
    private PivotXMLReader pivotXMLReader;
    
    public AbstractPivotParser(String pivotColumn) {
        this.pivotColumn = pivotColumn;
        this.pivotXMLReader = new PivotXMLReader();
    }
    
    // List 형식(여러 행)일 때
    public List<Map<String, Object>> parsePivotRow(List<Map<String, Object>> rows) {
        List<Map<String, Object>> result = new ArrayList<>();
        for(Map<String, Object> row : rows){
            result.add(parsePivotRow(row));
        }
        return result;
    }
    
    // Map 형식(단일 행)일 때
    public Map<String, Object> parsePivotRow(Map<String, Object> row) {
        String document = (String) row.get(pivotColumn);
        Map<String, Object> pivotRow = parsePivotColumn(document);
        row.putAll(pivotRow);
        return row;
    }

    private Map<String, Object> parsePivotColumn(String xml){
        InputSource is = new InputSource(new ByteArrayInputStream(xml.getBytes(StandardCharsets.UTF_8)));
        return pivotXMLReader.parsePivotColumn(is);
    }
}
```
이제 DAO에서는 여러 행 또는 단일 행을 조회하던 간에 parsePivotRow 라는 메서드를 호출하기만 하면 된다.

```java
// DAO에서 PivotParser 활용
public Map<String, Object> multiplePivotRows(){         // 단일 행을 조회할 때
    Map<String, Object> row = sqlSession.selectList("sqlId");
    PivotParser pivotParser = new PivotParser("XML");
    return pivotParser.parsePivotRow(row);  // Map
}

public List<Map<String, Object>> multiplePivotRows(){   // 여러 행을 조회할 때
    List<Map<String, Object>> rows = sqlSession.selectList("sqlId");
    PivotParser pivotParser = new PivotParser("XML");
    return pivotParser.parsePivotRow(rows); // List
}
```

## Abstract, Concrete, Interface

### AbstractPivotParser

지금까지 `PIVOT XML`이 VARCHAR2(Java에서는 String)으로 조회되었지만, BLOB나 CLOB로도 조회되는 경우까지 다루고 싶다.
전달받는 XML의 타입이 다르다는 것 말고 나머지는 모두 동일하므로 캡슐화를 통해 변하는 부분만 분리하려고 한다.

```java
/**
 * 가져오게 되는 XML의 class를 Generic으로 선언했다.
 * <E>로는 String이나 Blob, Clob 등이 될 수 있다.
 */
public abstract class AbstractPivotParser<E> {
    
    // PivotParser와 동일하므로 중략 ...
    public Map<String, Object> parsePivotRow(Map<String, Object> row) {
        E document = getPivotXML(row);
        Map<String, Object> pivotRow = parsePivot(document);
        row.putAll(pivotRow);
        return row;
    }

    /**
     * Map에서 XML을 꺼낼 때 전처리가 필요한 경우가 있다.
     * 그럴 때는 이 Hook Method를 @Override하면 된다.
     */
    protected E getPivotXML(Map<String, Object> row){
        if(pivotColumn == null){
            throw new IllegalArgumentException("pivotColumn is not defined");
        }
        return (E) row.get(pivotColumn);
    }

    /**
     * XML을 InputStream으로 변환하는 부분만 자식 객체에서 @Override하면 된다.
     */
    protected abstract InputStream convert(E xml);

    /**
     * 자식 객체가 InputStream을 가져오면 InputSource로 바꿔서 parsePivotColumn를 수행한다.
     */
    private Map<String, Object> parsePivot(E xml){
        InputStream inputStream = convert(xml);
        return pivotXMLReader.parsePivotColumn(new InputSource(inputStream));
    }
}
```

**convert가 InputSource를 반환하지 않는 이유** : InputSource는 SAX 패키지에만 국한되므로 좀 더 보편적인
InputStream으로 변환할 때가 쉬울 거라 생각했다. 그리고 InputSource 생성자에 InputStream을 받을 수 있으니
부모 객체 입장에서도 상관없다.
{: .notice--info}

### PivotStringParser 외

위의 추상 클래스를 상속받아서 구체 클래스를 만들려고 한다. `@Override`를 해야 하는 메서드는 convert 밖에 없다.
먼저 제너릭(Generic, 타입 E)이 String인 `PivotStringParser`는 다음과 같다.

```java
public class PivotStringParser extends AbstractPivotParser<String> {
    
    public PivotStringParser(String pivotColumn) {
        super(pivotColumn);
    }
    
    @Override
    protected InputStream convert(String xml) {
        InputStream is = new ByteArrayInputStream(xml.getBytes(StandardCharsets.UTF_8));
        return is;
    }
}
```

마찬가지로 PivotBlobParser와 PivotClobParser는 다음과 같다.

```java
public class PivotBlobParser extends AbstractPivotParser<Blob> {

    public PivotBlobParser(String pivotColumn) {
        super(pivotColumn);
    }

    @Override
    protected InputStream convert(Blob xml) {
        InputStream is;

        try {
            is = xml.getBinaryStream();
        } catch (SQLException e) {
            throw new IllegalArgumentException("Can not read binary stream", e);
        }

        return is;
    }

}
```

```java
public class PivotClobParser extends AbstractPivotParser<Clob> {

    public PivotClobParser(String pivotColumn){
        super(pivotColumn);
    }

    @Override
    protected InputStream convert(Clob xml) {
        Reader reader = null;
        InputStream is = null;

        try {
            reader = xml.getCharacterStream();
            is = IOUtils.toInputStream(IOUtils.toString(reader), StandardCharsets.UTF_8);

        } catch (SQLException | IOException e) {
            throw new IllegalArgumentException("Can not read character stream", e);
        } finally {
            if(reader != null) { try { reader.close(); } catch (IOException e) { } }
            if(is != null) { try { is.close(); } catch (IOException e) { } }
        }

        return is;
    }
}
```

abstract 메서드와 제너릭 타입을 통해 매우 적절하게 확장을 유도할 수 있었다.

### (Interface) PivotRowParser

AbstractPivotParser가 여러 타입을 가지다보니 이를 포괄할 수 있도록 Interface를 세울 생각이다. 인터페이스만 알고 있으면 되니
XML 타입을 가리키는 특정 구체 클래스와 느슨한 결합을 할 수 있게 된다.

```java
public interface PivotRowParser {
    public Map<String, Object> parsePivotRow(Map<String, Object> row);
    public List<Map<String, Object>> parsePivotRow(List<Map<String, Object>> rows);
}
```

## 기존 코드와 PivotRowParser 비교

위의 내용대로 개선된 `PivotRowParser`가 얼마나 나아졌는지 `DAO`에서 활용되는 모습으로 비교해보자.

```java
// AS-IS (기존)
public Map<String, Object> getWinnerAndScore(){
    // VARCHAR2 형식으로 XML을 직렬화한 쿼리
    Map<String, Object> row = sqlSession.selectOne("getWinnerAndScoreAsVarchar2");
    
    // pivot column 가져와서 InputSource 만들기
    String xml = row.get("PRIZE_XML");
    InputSource inputSource = new InputSource(new ByteArrayInputStream(xml.getBytes(StandardCharsets.UTF_8)));
    
    // pivotRow 파싱하기
    PivotXMLReader pivotXMLReader = new PivotXMLReader();
    Map<String, Object> pivotRow = pivotXMLReader.parsePivotColumn(inputSource);
    
    // pivotRow 추가
    row.putAll(pivotRow);
    return row;
}

// Refactored (개선)
public Map<String, Object> getWinnerAndScore(){
    Map<String, Object> row = sqlSession.selectOne("getWinnerAndScoreAsVarchar2");

    // BLOB 라면 PivotBlobParser(), CLOB 라면 PivotClobParser()를 생성
    PivotRowParser pivotParser = new PivotStringParser("PRIZE_XML");
    return pivotParser.parsePivotRow(row);
}
```

좋다. 간결하기도 하고 유연하기도 한게 그냥 좋다. 지금까지 만든 코드의 테스트는 [Github](https://github.com/rokong/hello-spring/blob/master/src/test/java/com/rokong/pivot/PivotRowTest.java)
에서 확인할 수 있다.
