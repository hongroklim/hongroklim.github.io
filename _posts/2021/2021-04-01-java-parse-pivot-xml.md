---
date: 2021-04-01 00:17:00 +0900
title: Java에서 PIVOT XML 파싱하기 (1/2)
excerpt: Oracle의 PIVOT XML을 Java에서 PIVOT처럼 다루자
categories: java
---

## 동적으로 PIVOT을 쓰려면

### PIVOT에게 아쉬웠던 점

여러 행에 걸쳐 조회되는 결과를 `PIVOT`을 사용하여 한 행으로 표현할 수 있었다. 이런 대단한 기능을 자체 내장함수로
지원한다는 점에서 마음에 들었다. 하지만 아쉬웠던 점은 **일일이 생성할 열의 값을 선언해 주어야 한다**는 것이다.

### PIVOT XML

동적으로 `PIVOT`을 쓰는 방법을 알아보던 중 `PIVOT XML`을 알게 되었다. `PIVOT` 결과를 XML 타입의 열로 조회한다면
`IN` 절에 서브쿼리를 사용할 수 있다는 것이다.

```sql
WITH TMP1 AS (
    SELECT '2021' AS YEAR, 1 AS QUARTER, 'GOLD' AS PRIZE, '가' AS WINNER, 98 AS SCORE UNION ALL
    SELECT '2021' AS YEAR, 1 AS QUARTER, 'SILV' AS PRIZE, '나' AS WINNER, 93 AS SCORE UNION ALL
    SELECT '2021' AS YEAR, 1 AS QUARTER, 'BRNZ' AS PRIZE, '다' AS WINNER, 89 AS SCORE UNION ALL
)
SELECT YEAR, QUARTER,
       PRIZE_XML /* 일반적인 PIVOT과 달리 하나의 열만 조회된다 */
  FROM TMP1
 PIVOT XML ( /* PIVOT 뒤에 XML을 넣어주면 된다 */
    MAX(WINNER) AS WINNER,
    MAX(SCORE) AS SCORE
    FOR PRIZE /* <FOR절 column>_XML 형식의 column으로 PIVOT 결과가 조회된다 */
    IN (SELECT distinct PRIZE FROM TMP1) /* IN 절에 서브쿼리를 쓸 수 있다 */
 )
```

PRIZE_XML은 다음과 같은 XML이 조회된다.

```xml
<PivotSet>
    <item> <!-- IN절에서 조회된 만큼 item이 조회된다 -->
        <column name="PRIZE">GOLD</column>  <!-- 첫번째 column은 column 이름이다 -->
        <column name="WINNER">가</column>   <!-- 그 뒤의 column은 데이터이다 -->
        <column name="SCORE">98</column>
    </item>
    <item>
        <column name="PRIZE">SILV</column>
        <column name="WINNER">나</column>
        <column name="SCORE">93</column>
    </item>
    <item>
        <column name="PRIZE">BRNZ</column>
        <column name="WINNER">다</column>
        <column name="SCORE">89</column>
    </item>
</PivotSet>
```

쿼리에서 위와 같은 XML로 조회한 뒤 이를 Java에서 `Map<String, Object>` 안에 일반적인 `PIVOT` 결과로 바꿔보려고 한다.

## 쿼리에서 XMLType을 처리하자

일반적으로 Mybatis에서 XML 타입의 결과를 조회한다면 다음과 같은 오류가 발생한다.
```text
java.lang.NoClassDefFoundError: oracle/xdb/XMLType
```
일반적으로 잘 알려진 JDBC Type이 아니다보니 아무래도 별도의 라이브러리가 필요한 상황이다. XMLType이 아닌 다른 타입으로 바꿔보자.

**참고** : XMLType으로 다루고 싶다면 [Oracle 홈페이지](https://www.oracle.com/database/technologies/jdbc-upc-downloads.html)
에서 `xdb6.jar`를 다운로드 받아 라이브러리에 추가하면 위의 문제는 해결된다.
{: .notice}

### CLOB, BLOB 타입

`XMLSERIALIZE`이라는 Oracle 함수를 사용한다면 XML을 직렬화(Serialize)할 수 있다. 변환할 타입으로 CLOB인지, BLOB인지 선택할 수 있다.

```sql
WITH TMP1 AS (
    SELECT '2021' AS YEAR, 1 AS QUARTER, 'GOLD' AS PRIZE, '가' AS WINNER, 98 AS SCORE UNION ALL
    SELECT '2021' AS YEAR, 1 AS QUARTER, 'SILV' AS PRIZE, '나' AS WINNER, 93 AS SCORE UNION ALL
    SELECT '2021' AS YEAR, 1 AS QUARTER, 'BRNZ' AS PRIZE, '다' AS WINNER, 89 AS SCORE UNION ALL
)
SELECT PRIZE_XML,
       XMLSERIALIZE(CONTENT PRIZE_XML AS CLOB) AS PRIZE_XML_CLOB,
       XMLSERIALIZE(CONTENT PRIZE_XML AS BLOB) AS PRIZE_XML_CLOB
  FROM TMP1
 PIVOT XML (
    MAX(WINNER) AS WINNER, MAX(SCORE) AS SCORE
    FOR PRIZE
    IN (SELECT distinct PRIZE FROM TMP1)
 )
```

### VARCHAR2 타입

직렬화 한 CLOB나 BLOB를 VARCHAR2로도 형 변환을 할 수 있다. 물론 VARCHAR2 타입 답게 **4000 bytes** 까지만 조회된다.

```sql
WITH TMP1 AS (
    SELECT '2021' AS YEAR, 1 AS QUARTER, 'GOLD' AS PRIZE, '가' AS WINNER, 98 AS SCORE UNION ALL
    SELECT '2021' AS YEAR, 1 AS QUARTER, 'SILV' AS PRIZE, '나' AS WINNER, 93 AS SCORE UNION ALL
    SELECT '2021' AS YEAR, 1 AS QUARTER, 'BRNZ' AS PRIZE, '다' AS WINNER, 89 AS SCORE UNION ALL
)
SELECT PRIZE_XML,
       CAST(XMLSERIALIZE(CONTENT PRIZE_XML AS CLOB) AS VARCHAR2(4000)) AS PRIZE_XML_VARCHAR2
       /* XMLSERIALIZE 할 떄 BLOB 설정해도 결과는 같다 */
       /* VARCHAR2는 최대 4000 bytes 까지 설정 가능하다 */
  FROM TMP1
 PIVOT XML (
    MAX(WINNER) AS WINNER, MAX(SCORE) AS SCORE
    FOR PRIZE
    IN (SELECT distinct PRIZE FROM TMP1)
 )
```

데이터베이스에서 무사히 XML을 가져왔으니 이제 Java가 나설 차례이다.

## Java에서 XML을 읽어보자

### SAX 패키지를 사용하려면

XML을 다루는 여러 라이브러리들이 있는데, 그 중 JDK에 포함되어 있는 [org.xml.sax](https://docs.oracle.com/javase/8/docs/api/org/xml/sax/package-summary.html)
 패키지를 활용할 예정이다. 참고로 sax는 *Simple API for XML* 이라는 뜻이다. sax를 활용하기 위해선 기본적으로 두 개의 인스턴스가 필요하다.

- **XMLReader** : XML을 읽는 동작을 수행한다.
- **DefaultHandler** : XML을 읽는데 필요한 정보들을 담고있다.

정리하자면 어떻게 XML을 읽어야 하는지 `DefaultHandler`에다 설명한 다음 그걸 `XMLReader`에 전달해 XML을 읽도록 시키는 방식이다.

**DefaultHandler의 역할** : 사실 XMLReader에겐 `EntityResolver`, `DTDHandler`, `ContentHandler`, `ErrorHandler` 총 4가지의
Handler가 필요하다. `DefaultHandler`는 이 4가지 인터페이스를 모두 구현하는 객체이며, 모든 메서드는 아무 동작을 수행하지 않는다. 모든 Handler
인스턴스를 만들 필요 없이 `DefaultHandler`를 상속받아 필요한 메서드만 `@Override`해서 XMLReader의 4가지 Handler에 담으면 된다.
{: .notice}

### PivotRowHandler

XML을 가지고 `Map<String, Object>` 에다가 `PIVOT` 테이블 형태로 바꾸는 방법을 알려주는 객체이다.
XML 타입의 `PIVOT XML` 형식과 Map 타입의 `PIVOT` 형식을 이어주는 아주 핵심적인 역할이다.
`DefaultHandler`를 상속받았는데, 어떤 메서드를 어떻게 `@Override`할 지는 다음과 같다.

* \#startDocument
  - 문서를 읽기 시작할 때 가장 먼저 실행된다.
  - `PIVOT` 테이블의 결과를 담을 속성(pivotRow)을 초기화한다.
* \#characters
  - element 안(<...>와 </...>)에 있는 문자열을 읽는 역할이다.
  - 일단 읽어들인 문자열을 `StringBuffer`에 저장한다.
* \#startElement
  - element가 시작(<...>)될 때 실행되며 element의 name(태그 이름)과 attributes(##='@@' 이런 거)를 담고 있다.
  - \<item\>을 만날 때 마다 새로운 column group임을 인식하고 column 데이터의 이름(name attribute에 있음)을 가지고 있는다.
* \#endElement
  - elemente가 종료(</...>)될 때 실행된다.
  - 새로운 column group이라면 column 데이터의 이름에 붙일 접두어를 저장하고, column 데이터를 만났다면 테이블의 결과에 추가한다.

```java
public class PivotRowHandler extends DefaultHandler {
    private Map<String, Object> pivotRow;
    private StringBuilder content = new StringBuilder();
    private String columnPrefix = null;
    private String dataKey;
    
    @Override
    public void startDocument(){
        pivotRow = new HashMap<>();
    }
    
    @Override
    public void characters(char[] ch, int start, int length){
        content.setLength(0);
        content.append(ch, start, length);
    }

    @Override
    public void startElement(String url, String localName, String qName, Attributes attributes){
        if("item".equals(qName)){
            //start bundle of column group
            columnPrefix = null;
        }else if("column".equals(qName) && columnPrefix != null){
            //belongs to column group
            dataKey = columnPrefix + attributes.getValue("name");
        }
    }

    private String getContent(){
        return content.toString().trim();
    }

    @Override
    public void endElement(String url, String localName, String qName){
        if("column".equals(qName)){
            if(columnPrefix == null){
                //this element becomes header of column group
                columnPrefix = getContent() + "_";
            }else{
                //the others belong to header
                pivotRow.put(dataKey, getContent());
            }
        }
    }

    public Map<String, Object> getPivotRow(){
        return pivotRow;
    }
}
```

사람이 XML을 읽는 경우보다는 훨씬 더 복잡해 보인다. 왜냐하면 XML을 바탕으로 pivotRow를 만들기 위해서는 맥락이 필요하지만
Handler의 메서드는 자신이 호출된 시점에서 `XMLReader`가 읽은 부분만 알고있기 때문이다. 그러므로 메서드들이 맥락있게 동작하기 위해선
자신이 알고 있는 정보를 다른 메서들과 공유, 즉 객체 속성에 저장을 해야하고 다른 메서드들은 기존에 저장된 속성과 자신이 아는 정보를
비교하면서 특정한 동작을 수행하게 된다.

### PivotXMLReader

위에서 생성한 `PivotRowHandler`를 `XMLReader`에다가 담은 뒤 parse 동작을 수행한다.
\#parsePivotColumn 메서드를 손쉽게 사용할 수 있도록 `XMLReader` 생성과 예외처리를 돕는 역할이다.

```java
public class PivotXMLReader {

    protected XMLReader pivotReader;
    protected PivotRowHandler pivotRowHandler;

    public PivotXMLReader(){
        initXMLReader();
    }

    private void initXMLReader(){
        try {
            pivotReader = XMLReaderFactory.createXMLReader();
        } catch (SAXException e) {
            throw new IllegalStateException("XMLReaderFactory can not create XMLReader", e);
        }

        pivotRowHandler = new PivotRowHandler();

        pivotReader.setContentHandler(pivotRowHandler);
        pivotReader.setEntityResolver(pivotRowHandler);
        pivotReader.setErrorHandler(pivotRowHandler);
        pivotReader.setDTDHandler(pivotRowHandler);
    }

    public Map<String, Object> parsePivotColumn(InputSource inputSource){
        Map<String, Object> pivotColumns = null;

        try {
            pivotReader.parse(inputSource);
            pivotColumns = pivotRowHandler.getPivotRow();
        } catch (IOException | SAXException e) {
            throw new IllegalArgumentException("Can not read XML", e);
        }

        return pivotColumns;
    }
}
```

**XMLReader가 SingleTon이 아닌 이유** : Thread-Safe하지 않기 때문이다. XMLReader의 주석을 참고하자.\
`All SAX interfaces are assumed to be synchronous: the parse methods must not return until parsing is complete,
and readers must wait for an event-handler callback to return before reporting the next event.`
{: .notice}

## 한번 활용해보자

### DAO에서 파싱하기

`PivotXMLReader`를 활용한다면 `PIVOT XML`을 사용한 쿼리더라도 일반적인 `PIVOT`만을 사용한 쿼리와 같은 형태로 만들 수 있다.
Service 계층 입장에서는 구체적인 쿼리의 구현방법을 모를테니 `PivotXMLReader`를 DAO에서 활용하는 게 적절해 보인다.

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

### 개선할 점

일반적인 `PIVOT`을 활용할 때와 같은 결과를 만들 수 있었다. 하지만 이건 VARCHAR2 형식으로 조회를 하고, 하나의 행만 조회한 경우에만
활용할 수 있는 방법이다. 지금도 문제없이 작동할 수 있긴 한데, 이러한 점들을 다음 글에서 개선해보겠다.
