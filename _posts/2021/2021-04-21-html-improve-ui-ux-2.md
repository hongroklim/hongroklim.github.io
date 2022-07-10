---
date: 2021-04-21 22:57:00 +0900
title: 나만의 웹페이지 UI/UX 개선하기(2/2)
excerpt: IndexedDB로 서버없이 html에 날개를 달아보아요 
header:
  overlay_image: https://user-images.githubusercontent.com/59322692/115569862-93314180-a2f8-11eb-9fd4-6f7c2f7227bd.png
categories: web 
---

## ajax만 써봐서 그런지

나는 ES6 문법이나 W3C 최신 표준이 어색하다. InternetExplorer 8에서도 작동하는 페이지를 개발해왔기에, DBR Viewer 1세대에서는
데이터를 불러올 때 ajax (jQuery 없이 쓸 때는 [XMLHttpRequest](https://developer.mozilla.org/ko/docs/Web/API/XMLHttpRequest)) 
만을 사용하여 매번 요청을 보냈다. 하지만 이젠 CORS proxy에 제한이 생겨버린 request를 줄이고자 캐싱을 해야 하는데, 다행이도 [IndexedDB](https://developer.mozilla.org/ko/docs/Web/API/IndexedDB_API)
라는 게 존재하여 이를 활용해보기로 했다. 다만 비동기 프로그래밍이 익숙하지 않은 나란 사람을 배려하고자 라이브러리 처럼 최대한 활용하기 쉽게 구현해보기로 했다.

## repo.js

모던한 javaScript가 익숙하지 않은 개발자(특히 나)를 위해 최대한 활용하기 쉽게 IndexedDB를 지원하는 `repo.js`를 만들 것이다. 기존 1세대에서
사용하던 ajax를 쓰는 것 같으면서도 IndexedDB가 제공하는 DBMS스러운 기능들도 누릴 수 있는 방식으로 개발할 생각이다. 주요 메서드를 정리하자면
다음과 같다.

- `#ajax(String url)` : CORS proxy를 통한 XMLHttpRequest를 수행하게 된다
- `#tx(String mode)` : IndexedDB에 readwrite 또는 readonly로 transaction을 연다
- `#insert(String storeName, JSON Value)` : DML의 INSERT 역할
- `#put(String storeName, JSON value)` : DML의 UPDATE 역할
- `#getValueByKey(String storeName, String key)` : DML의 SELECT 역할. key로 한 건만 조회한다

### 비동기적인 사용법

절차적인 프로그래밍 언어로 개발할 때는 위의 코드가 모든 실행을 끝낸 다음에 아래의 코드가 실행된다. 하지만 비동기 프로그래밍은 다르다.
비동기적인 구문을 만나면 일단 동작을 시작하고 다음줄에 있는 코드 또한 동작한다. 그렇다면 비동기 구문의 실행결과를 받은 뒤에 해야 할 코드는 어떻게 만들어야 할까?
바로 `callback` 매서드를 작성하면 된다.

IndexedDB에서 `open()`이라는 매서드가 있는데, 데이터베이스와의 connection을 만드는 역할로 비동기적으로 동작한다. 먼저 open이 되어야
INSERT나 SELECT를 할 수 있으므로 [Promise](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise)
를 사용하여 callback 함수를 활용한다.

```javascript
function db(){
    return new Promise((resolve, reject) => {
        //connection 만들기
        let request = indexedDB.open('dbr', 1);
        
        //새로 생성되거나 version이 바뀌었을 때
        request.onupgradeneeded = function(ev){
            let db = ev.target.result;
            //create objectStores
            // ...
        };
        
        //connection 만들기 성공
        request.onsuccess = function(ev){
            resolve(ev.target.result);
        };
        
        //connection 만들기 실패
        request.onerror = function(ev){
            console.error('failed to open indexedDB', ev);
            reject(ev);
        };
    });
}

//매서드 활용 예시
db().then(function(conn){
    // ...
});
```

`indexedDB.open()`을 전달받은 `request`는 connection을 가진 것이 아니라 단지 connection을 만들도록 요청한 것이다. 만약 connection을 만드는 데
실패했다면 `request.onerror`, 성공했다면 `request.onsuccess` 라는 매서드가 실행될 것이다. 성공한 매서드의 파라미터에는
connection(`ev.target.result`)이 담겨있고 그걸 Promise에 있는 `resolve`에 전달하면 된다. 위에서 `db()`라는 함수를 사용할 때는 `.then(...)` 으로
`resolve`로 전달한 값을 받을 수 있다. open이라는 매서드 자체는 비동기이지만, 그걸 감싸고 있는 Promise와 이를 잇는 then 매서드는 동기적으로 작동한다.

### INSERT와 UPDATE

connection을 열어 transaction을 만들었다면 그 다음은 쉽다. 수정하고자 하는 `objectStore`를 가져와서 다음과 같이 사용하면 된다.
```javascript
//transaction 만들기
function tx(){
  // db()로 connection을 만든다.
  return db().then((db)=> {
    //transaction의 첫번째 parameter에는 필요한 objectStore들을 넣으면 된다.
    let tx = db.transaction(['archive', 'single', 'recent'], 'readwrite');
    tx.onerror = function (ev) {
      console.error('failed to add into indexedDB', ev);
    };
    return tx;
  });
}

//storeName으로 가져온다.
let objStore = tx().objectStore('archive');

let value = {"pubNumber": 319, "title": 'New Wave of Logistics'};
objStore.add(value);    //INSERT
objStore.put(value);    //UPDATE

//주의 : 특정 keyPath나 index key 값이 없다면 error를 뱉어낸다.
```

### SELECT 모음집

IndexedDB는 값을 조회할 수 있는 여러 방법들을 제공한다. 우선 transaction을 만들어 `storeName`으로 `objectStore`를 가져온다.
```javascript
//조회를 할 때는 readOnly 모드도 가능하다.
function tx() {
  //...
  let tx = db.transaction(['archive', 'single', 'recent'], 'readOnly');
  //...
}

//먼저 storeName으로 objStore를 가져온다.
let objStore = tx().objectStore('archive');
```

이제 Promise 구문과 함께 원하는 방식대로 조회하면 된다.

```javascript
new Promise(((resolve, reject) => {
  /** 1. key로 1건 조회할 때 */
  let request = objStore.get(key);
  
  request.onsuccess = function(ev){
    //정상적으로 조회한 경우
    resolve(request.result);
  };
  request.onerror = function(ev){
    // 오류가 발생했어요 ...
  };
}));


/** 2. index key로 가져올 때 */
// Promise 안에서 ...
let index = objStore.index('pubDate');
let request = index.get('20210421');
// onsuccess, onerror 생략 ...


/** 3. key의 최댓값 또는 최솟값 가져올 때 */
// Promise 안에서 ...
let request = objStore.openCursor(null, 'prev');    //prev : 최대 -> 최소
//let request = objStore.openCursor(null, 'next');  //next : 최소 -> 최대

request.onsuccess = function(ev){
  let cursor = ev.target.result;
  if(cursor){
    resolve(cursor.value);  //조회된 결과가 있다면
  }else{
    resolve(null);          //조회된 결과가 없다면
  }
};


/** 4. 상위 n개를 가져올 때 */
return new Promise(((resolve, reject) => {
  let objectList = [];
  let request = objStore.openCursor(null, 'prev');  //최댓값부터 조회하기 시작한다.
  request.onsuccess = function(ev){
    let cursor = ev.target.result;

    if(cursor){
      // 조회된 결과가 있다면 array에 넣기
      objectList.push(cursor.value);
      
      if(objectList.length < n){
        //아직 n개 다 못 채웠다
        cursor.continue();
      }else{
        // n개 반환
        resolve(objectList);
      }
      
    }else{
      // 더이상 조회할 게 남아있지 않다면
      resolve(objectList);
    }
  };
  // onerror 생략 ...
}));
```

## 웹페이지에서 활용하기

### IndexedDB 들키지 않기

`repo.js`를 만든건 ajax로만 데이터를 조회하던 것을 중간에 IndexedDB를 껴서 활용하고자 하는 목적이었다. 나의 구상은 다음과 같다.

<figure>
  <img src="https://user-images.githubusercontent.com/59322692/115735950-d73e4800-a3c5-11eb-9d5a-6462fa2d0bd5.png"
       alt="content_01">
  <figcaption>loadPage 부터 setPage 까지 발행정보를 불러오는 과정. 회색부분이 IndexedDB를 활용한 부분이다.</figcaption>
</figure>

`loadPage` 입장에서는 발행정보를 가져오고(`getArchive`) 그 정보를 화면에 보여주기(`setPage`)만 하면 된다. 이건 ajax로만 했던 방식과는
차이가 없지만, `getArchive`에서는 IndexedDB를 활용한다는 점에서 구체적인 작동방식이 달라진다. 먼저 IndexedDB에서 해당 발행정보가 존재하는지 확인
(`getFromIDB`)한 다음 만약 존재하지 않다면 발행정보를 추가(`addArchive`)하게 된다. 이 때 ajax로 조회한 내용을 IndexedDB에 담은 뒤(`addToIDB`)
발행정보를 돌려준다.

### 실제로 개발

이를 javaScript에서는 다음과 같이 구현했다.

```javascript
function loadPage(pubNumber){
  // 페이지를 불러오는 함수. IndexedDB가 쓰인 줄도 모른다.
  return getArchive(pubNumber).then(setPage);
}

function getArchive(pubNumber){
  //먼저 IndexedDB에서 찾는다
  return getArchiveFromIDB(pubNumber).then((pub)=>{
    if(pub !== undefined){
      //있다면 바로 가져오고
      return new Promise(resolve => {resolve(pub);});
    }else{
      //없으면 IndexedDB에 추가
      return addArchive(pubNumber);
    }
  });
}

function addArchive(pubNumber){
    const archiveUrl = `https://dbr.donga.com/magazine/mcontents/pub_number/${pubNumber || ''}`;
    
    //ajax로 정보를 가져오기
    return repo.ajax(archiveUrl).then((response) => {
        let archive = parseArchive(response);   // html에서 json을 뽑은 다음
        repo.add('archive', archive);           // IndexedDB에 추가를 하고
        return archive;                         // 데이터를 돌려준다
    });
}
```

## 끝이 아니다

그동안 javaScript를 다른 절차적 프로그래밍 언어처럼 다루어 그런지 `repo.js`를 만들 때 낯선 느낌이 들었다. 어서 ES6 문법에 익숙해져서
javaScript가 추구하는 방향대로 개발해야겠다. 앞으로 DBR Viewer 기능으로 추가하고 싶은 건 **로딩화면** 구현이다. 핑그르르 돌아가는 애니메이션을
달거나 skeleton만 세워두고 대체하는 식으로 하던가 여러 방법은 차차 고민해볼 생각이다. 그리고 `repo.js`에서 **예외상황**을 좀 다루고 싶은데,
지금은 일단 무조건 성공했을 때만을 가정하여 진행하고 있다. 오류가 나면 console에만 보여주지 말고 그에 따른 적절한 action들을 사용자에게
제공해야 할 것 같다.

> **앞으로의 DBR Viewer의 계획**
> 1. 로딩화면(또는 애니메이션)이 보여집니다.
> 1. 오류가 발생한다면 적절한 조치를 취합니다.

### 아티클을 읽을 때

1세대에서 2세대로 넘어가면서 빠진 것이 하나 있는데, 바로 **얼마나 읽었는지 표시** 해주는 기능이다. 그런 요소에 대한 퍼블리싱 같은건 없거니와,
이 블로그와의 테마와도 맞추어야 하기 때문에 좀 고민을 해봐야 겠다. 그래도 javaScript로 구현은 되어있으니 디자인만 결정하기만 하면 된다.
그리고 또 2세대에서 욕심을 내려고 하는 것은 **자동생성 목차**이다. 단순히 몇몇 html 규칙을 찾아서 적용해도 되긴 하는데, 더 정교하게 할 수도
있을 것 같다. 예를 들어 여러 형식으로 아티클을 조회하는데 그 중에서 두드러지는 요소들을 고른다거나 딥러닝(...뇌절이다)을 통해 뽑아낸다거나.

> **아티클을 더 편하게 읽어요**
> 1. 지금 얼마나 읽었는지 실시간으로 알려줍니다.
> 1. 아티클 목차를 정리해주고, 목차를 누르면 해당 위치로 이동합니다.
