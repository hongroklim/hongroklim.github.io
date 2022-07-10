---
date: 2021-04-19 22:54:00 +0900
title: 나만의 웹페이지 UI/UX 개선하기(1/2)
excerpt: backend가 아닌 퍼블리싱부터 시작해보는 개발 경험담
header:
  overlay_image: https://user-images.githubusercontent.com/59322692/115428336-4177b100-a23d-11eb-96e8-285df511450e.png
categories: web
---

## DBR Viwer (1세대)
[동아 비즈니스 리뷰](https://dbr.donga.com/), DBR을 처음 알게 된 건 대학교 1학년 경영학원론 시간이었다. 단지 경영학 이론 중 하나에 관한
글이었음에도 이 매거진이 너무 좋았다. 입대 후에 DBR을 구독해 읽으려고 찾아보았는데, 1달에 2만원 정도 되는 금액이 당시 군인인 나에겐 좀 부담스러웠다.
다행인건 DBR 홈페이지에서 하루 2편의 글을 제공하고 있었는데, 이를 잘만 이용하면 무한으로 즐길 수 있겠다고 생각했다. 그래서 탄생한 것이 바로 DBR Viewer(1세대)이다.

<figure>
  <img src="https://user-images.githubusercontent.com/59322692/115415400-d8d70700-a231-11eb-8273-201a37d18f52.png"
       alt="content_01">
  <figcaption>DBR Viewer 1세대. (좌) 발행정보, (우) 아티클</figcaption>
</figure>

### 사양
하나의 파일 안에 html, css 그리고 javascript까지 모두 들어있다. 발행호수 별로 아티클을 목록을 볼 수 있는 화면과 아티클을 읽는 화면 두 개로 구성되어 있다.
호스팅은 github에서 제공하는 기능을 이용하였고, DBR에서 해당 정보를 가져올 때 [cors-anywhere](https://github.com/Rob--W/cors-anywhere) 이라는 프록시를 거쳐서 가져온다.

### 개선할 점
1년 남짓한 기간동안 거의 안 바꾸고 잘 사용하던 중 아티클 목록이 로딩이 안 되기 시작했다. 원인을 알아보니 cors-anywhere 사이트가 사용을 일부
제한하기 시작한 것이었다. DBR Viewer를 아예 사용할 수 없게 되버리니 이참에 대대적으로 수정하기로 마음먹었다.
CORS proxy를 [herokuapp](https://www.heroku.com)으로 직접 운영하려 하는데, 시간당 request 제한이 있으니 [IndexedDB](https://developer.mozilla.org/ko/docs/Web/API/IndexedDB_API)
를 활용해 캐싱을 해보려고 한다. 또한 원래 독립적으로 호스팅되던 웹페이지를 이 블로그와 합치려는데 그렇다면 이 블로그에 맞는 디자인으로 수정하려고 한다.
미지막으로 최근에 본 글과 다음에 읽을 글을 보여주는 편의기능도 추가하면 좋을 것 같다.

> - rokong.github.io 블로그에 맞는 UI로 바꾸기
> - IndexedDB를 활용한 CORS proxy 요청 수 줄이기
> - 최근에 본 글, 다음에 읽을 글 보여주기

## 	UI 개선과정

DBR Viewer 1세대 에서는 DBR 홈페이지에 있는 html에서 필요한 부분을 그대로 도려내서 보여주는 방식이었다.
2세대에선 유연한 퍼블리싱을 위해 html에서 필요한 정보만 뽑아내 json 형태로 가공한 뒤 이를 html 템플릿에 보여주는 형식으로 바꿀 생각이다.

<figure>
  <img src="https://user-images.githubusercontent.com/59322692/115421669-1b4f1280-a237-11eb-9e12-5acf4cfe8b02.png"
       alt="content_02">
  <figcaption>1세대(상)와 2세대(하)의 작동방식 차이</figcaption>
</figure>

### 퍼블리싱 먼저 하기

일단 퍼블리싱부터 할 생각이다. 웹 개발에서는 백엔드 - 프론트엔드 - 퍼블리싱 순서대로 하는게 순조로운 것 같긴 한데, 일단 보기가 좋아야 개발할 맛이
나는건 어쩔 수 없다. 기존 1세대의 화면을 snapshot 찍듯이 html로 저장하고 이 블로그 테마의 layout을 참고하여 기본적인 html부터 만들었다.

<figure>
  <img src="https://user-images.githubusercontent.com/59322692/115423432-c6ac9700-a238-11eb-956d-556e7eedc899.png"
       alt="content_03">
  <figcaption>1세대(좌) 화면에서 html 요소만 추려낸 결과(우)</figcaption>
</figure>

얼추 골격이 갖추어졌으니 그 위에 css를 꾸밀 차례이다. 블로그 퍼블리싱은 [sass](https://sass-lang.com/) 로 개발되었는데 한 번도 sass를 다뤄보지 않아서 걱정이 앞섰다.
막상 개발을 시작하니 css만 알고 있다면 sass를 적용하는데는 큰 문제는 없었고, 블로그 테마에 이미 폰트 사이즈와 컬러에 대한 상수가 있으니
손쉽게 개발할 수 있었다. 제법 그럴듯하게 만들어진 화면을 과녁으로 삼아서 본격적인 개발을 시작하겠다.

<figure>
  <img src="https://user-images.githubusercontent.com/59322692/115424006-3cb0fe00-a239-11eb-8219-40bb26406f0f.png"
       alt="content_04"
       style="width: 70%">
  <figcaption>블로그 레이아웃을 참고하기 위한 퍼블리싱 전용 페이지</figcaption>
</figure>

### json 데이터 만들기

frontend에서 봤을 때 1세대에서 2세대로의 가장 큰 변화는 IndexedDB이다. 아티클 목록이나 본문에 관한 정보를 html 뭉텅이가 아니라
json형태로 들고 있어야 이들을 잘 활용할 수 있을 것이다. 그래서 DBR 홈페이지에서 json을 추출하려고 하는데, 만약 그렇게 하게 된다면 frontend에서
backend로의 개발과정을 벗어나버린다. 이 문제를 해결하는 방법으로 떠올린 것이 일단 json 파일을 만드는 것이다. json으로 파싱을 했던, IndexedDB에다
캐싱을 했던 간에 javascript 입장에서는 결국엔 json 형식의 데이터를 전달받을 뿐이다. backend의 세부 로직은 건너뛰고 발행정보와 아티클 정보를
담고 있는 *.json 파일을 만들었다. 그리고 일단은 1세대에서 사용하던 방식인 ajax로 json 데이터를 가져오도록 한다.

<figure>
  <img src="https://user-images.githubusercontent.com/59322692/115425304-79c9c000-a23a-11eb-9a50-255bef6b23aa.png"
       alt="content_05">
  <figcaption>기존 backend 설계(상)와 frontend 개발에 쓰일 방식(하)</figcaption>
</figure>

### html template 추출

화면 렌더링을 할 때 가장 기초적인 방법은 모든 html을 만들어 둔 채 json에서 뽑은 정보를 가지고 text를 집어넣는 것이다. 하지만 아티클 목록 개수나
다음에 읽을 글과 같이 json 데이터에 따라 html 자체가 달라지는 상황이 생기므로 데이터를 넣기 전에 html을 그린다는 것은 불가능하다.
고정된 요소들은 미리 html로 그려놓는다 치고, 변할 수 있는 요소에 대해서는 [html template](https://developer.mozilla.org/ko/docs/Web/HTML/Element/template)
을 활용하여 동적으로 생성할 계획이다.

```html
<!-- table of contents(TOC)에 쓰일 요소 -->
<template id="toc">
    <nav class="toc">
        <header>
            <h4 class="nav__title">
                <i class="fas fa-file-alt"></i>
                최근 본 글들
            </h4>
        </header>
        <ul class="toc__menu">
        </ul>
    </nav>
</template>

<!-- TOC 안에 있는 항목 -->
<template id="tocItem">
    <li>
        <a href=""></a>
    </li>
</template>

<!-- 발행정보 내 아티클 목록 -->
<template id="articleEl">
    <li>
        <a class="article_title">
            <span class="category"></span>
            <span class="title"></span>
            <span class="name"></span>
        </a>
    </li>
</template>
```
### loadPage 함수와 eventListener

어느정도 페이지 골격과 template까지 만들었으니 본격적으로 javascript를 통해 화면에 그릴 차례이다. 일단 이미 페이지에 그려진 html에 대해서는 
innerText나 setAttribute 등으로 데이터를 뿌려놓았다. 동적으로 만들 부분은 아래와 같이 template를 복제하여 화면에 그려넣는다.

```javascript
/* 발행정보에 있는 아티클 목록 생성하기 */
let ulWrapper = document.createElement('ul');           //항목들의 parent 요소
let template = document.getElementById('articleEl');    //template 요소
pub.articleList.forEach(function (article, index) {
    //template 요소 복사
    let articleEl = document.importNode(template.content, true);
    
    //요소 안에 정보 집어넣기
    articleEl.querySelector('a').href = single.getUrlById(article.id);
    articleEl.querySelector('.category').innerText = article.category;
    articleEl.querySelector('.title').innerText = article.title;
    articleEl.querySelector('.name').innerText = article.author;
    
    //각 항목들을 parent에 append
    ulWrapper.append(articleEl);
});
```

화면을 그릴 함수를 다 만들었으면 html의 eventListner에 바인드 할 차례이다. 발행정보를 보는 화면에서 페이지를 그릴 상황은 다음과 같다.
첫번째는 처음 페이지가 로딩될 때, 두번째는 다른 발행정보를 조회할 때 이다. 나는 어떤 발행정보를 조회하는 지를 url의 hash에 넣어서 표현할 예정이기
때문에 [window.onload](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onload) 와
[window.onhashchange](https://developer.mozilla.org/en-US/docs/Web/API/WindowEventHandlers/onhashchange) 에 다음과 같은 이벤트를 걸었다.

```javascript
window.onload = function(){
    //onload event는 onhashchange를 trigger
    window.onhashchange(undefined);
}

window.onhashchange = function(){
    //현재 location을 가지고 발행번호 가져오기
    const pubNumber = getPubNumberByHash(window.location.hash);
    
    //화면 그리기
    loadPage(pubNumber);
}
```
