---
date: 2021-09-07 22:21:00 +0900
title: XPath로 XML 문서 수정하기
excerpt: Java 기본 라이브러리인데 한번 써봐야지
categories: java
---

<figure>
  <img src="https://i.imgur.com/AStFrPk.png"
       alt="java_codes">
  <figcaption>
  </figcaption>
</figure>

전에 개발자로 참여했던 웹 프로젝트가 있었는데, UI 개발할 때 상용 소프트웨어를 사용하고 있었다.
개발자는 Drag & Drop으로 Component를 위치시킨 후 속성들을 에디터에서 수정할 수 있고, 이러한 정보들이
내부적으로 XML Document 형태로 저장되는 방식이다. 그러던 어느날 일괄적으로 어느 Component의 기본 속성들을
바꿀 일이 생겼다. 이미 너무 많은 파일과 Component들을 만들어둔 상태에서 고민에 빠졌다. 노가다로 할 것인가? 아니면 다른 방법을 시도할 것인가?
나의 선택은 후자였다.

## Document 읽기

XML을 읽기 위헤서는 `DocumentBuilderFactory`로 `DocumentBuilder`를 만들고 이를 `#parse`하면 된다.

```java
private static final DocumentBuilderFactory documentBuilderFactory;

static{
    documentBuilderFactory = DocumentBuilderFactory.newInstance();
    documentBuilderFactory.setNamespaceAware(true); // true로 해야지만 namespace를 인식할 수 있다
}

public Document parseXML(String pathname) throws Exception {
    DocumentBuilder documentBuilder = documentBuilderFactory.newDocumentBuilder();
    // #newDocumentBuilder로 매번 인스턴스를 생성한다
    
    return documentBuilder.parse(new File(pathname));
}
```

**Static Attributes** :
`DocumentBuilderFactory`는 Thread-Safe하다. (test : [junit](https://github.com/rokong/hello-spring/blob/master/src/test/java/com/rokong/xpath/FileIOTest.java)).
하지만 `documentBuilder`는 Thread-Unsafe 하다. (출처 : [stackoverflow](https://stackoverflow.com/questions/12455602/is-documentbuilder-thread-safe))
둘 다 static으로 하고 싶었지만, `documentBuilder`는 인스턴스로 해야만 한다.
{: .notice}

## XPath로 원하는 element 찾기

### XPath Instance

`Document`를 불러들였으면 이를 `XPath`로 읽어들이면 된다.

```java
XPath xPath = XPathFactory.newInstance().newXPath();
```

만약 XML 파일에 namespace가 존재할 때, prefix에 bound된 namespaceURI를 찾기 위한 메서드를 `@Override` 하면 된다.

```java
XPath xPath = XPathFactory.newInstance().newXPath();
xPath.setNamespaceContext(new NamespaceContext() {
    @Override
    public String getNamespaceURI(String prefix) {
        if("ns".euqlas(prefix)) {
            return "http://www.w3.org/2000/xmlns/";
        } else {
            return "";
        }
    }
    
    @Override
    public String getPrefix(String namespaceURI) {
        return null;
    }
    
    @Override
    public Iterator<String> getPrefixes(String namespaceURI) {
        return null;
    }
});
```

XPath로 `Document`를 포함한 `Node`의 객체를 탐색할 수 있다.

```java
String expression = "";
Document document;
NodeList nodeList = null;
try {
    nodeList = (NodeList) xPath.evaluate(expression, document, XPathConstants.NODESET);
} catch (XPathExpressionException e) {
    e.printStackTrace();
}
```

`XPathConstants.NODESET`로 `Node` 타입의 항목만 탐색하게 되며, 조회된 결과가 없을 경우 `getLength()`가 0인 `NodeList`가
반환된다.

### expression 예제

태그명으로 `Node`를 찾는 방법은 다음과 같다.

| expression | output |
|:--- |:---|
| `/html` | root인 `<html>` |
| `//td` | 모든 `<td>` |
| `//tr/td` | parent가 `<tr>`인 `<td>` |
| `//table/*` | parent가 `<table>`인 모든 Node |
| `//table//td` | ancestor 중에 `<table>`이 있는 `<td>` |

특정 `Node`의 속성을 활용하여 필터링을 할 수도 있다.

| expression | description |
|:--- |:---|
| `//td[@style]` | style 속성이 존재 (공백포함) |
| `//td[@colspan=2]` | colspan 속성이 2 |
| `//td[@colspan>=3]` | colspan 속성이 3 이상 |
| `//table[@id='tb_main']/td` | (ancestor도 필터링을 할 수 있다) |

또한 다양한 함수를 지원한다.

| expression | description |
|:--- |:---|
| `//td[position() = 1]` | 첫번째 child |
| `//td[contains(@class, 'text-right')]` | `text-right`를 포함하는 class |

## element 수정하기

expression으로 특정 `Node` 또는 `NodeSet`을 구한 뒤 객체를 수정한다.

```java
for(int i=0; i<nodeList.getLength(); i++){
    Element e = (Element) nodeList.item(i);
    
    // nodeValue를 수정할 때
    e.setNodeValue("new value");
    
    // attribute를 수정할 때
    e.setAttribute("attribute name", "new value");
}
```

## Document 저장하기

읽을 때와 비슷하게 저장할 때는`TransformerFactory`를 활용한다.

```java
private static final TransformerFactory transformerFactory = TransformerFactory.newInstance();

public Document saveXML(File file, Document document) throws Exception {
    DOMSource domSource = new DOMSource(document);
    StreamResult streamResult = new StreamResult(file);

    Transformer transformer = transformerFactory.newTransformer();
    transformer.transform(domSource, streamResult);
}
```

이렇게 수정한 내용을 파일에 저장할 수 있다. 노가다로 하려던 수정작업을 XPath를 활용하여 쉽게 할 수 있게 되었다.
다만 이렇게 파일로 저장하게 되면 새로 XML 파일을 저장하는 과정에서 공백이나 줄바꿈 등이 바뀌어버렸다. commit할 때
whitespace에 의한 수정인지 아닌지를 확인해봐야 하는 번거로움이 있었지만, 그래도 수고로움은 아낄 수 있었다.

나는 XPath로 문서를 다룰 때 파일 읽기/쓰기를 담당하는 `FileIOHandler`와 XPath를 담당하는 `XmlFile`로 객체를 나누었다.
테스트 코드는
[GitHub](https://github.com/rokong/hello-spring/blob/master/src/test/java/com/rokong/xpath/XPathTest.java)
에서 확인할 수 있다.
{: .notice}
