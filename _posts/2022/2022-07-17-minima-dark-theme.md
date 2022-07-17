---
date: 2022-07-17 12:00:00 +0900
title: Jekyll Minima에 다크모드 적용하기
excerpt: 요즘 블로그처럼 테마를 적용해볼까나
categories: dev
---

## 기존 테마의 한계점

지금 나의 블로그는 Jekyll의 기본 테마인
[Minima](https://github.com/jekyll/minima) Theme을 사용하고 있다. 별다른 설정
없이 `_config.yml`에 `theme: minima` 한 줄만 적으면 되니 간편하니 좋았다.
그리고 Minima theme에는 기본적으로 classic과 dark 테마를 적용할 수 있는 기능이
있다. 하지만 문제는 `_config.yml`에서 `skin: dark` 이런 식으로 **하나의 테마가
고정된다는 것**이다. 내가 하고 싶은 것은 사용자가 버튼을 눌러 자신이 원하는
테마로 바꿀 수 있도록 구현하는 것이다.

**참고** : 아래의 설정들은 `Gemfile`에 `github-pages`가 에 등록되어 있음을
전제로 한다.
{: .notice}

## `_config.yml` 수정하기

### `theme` 말고 `remote_theme`

먼저 `_config.yml`에서 `theme` 대신에 `remote_theme`을 사용해야 한다.
`theme`을 사용하게 되면 Minima의 가장 최신 패키지를 사용할 수 있지만,
마지막 업데이트가 2019년 (v2.5.1 기준)이라 상당히 오래되었다. `theme:
minima`를 지우고 다음과 같이 설정하자.

```yml
remote_theme: jekyll/minima
```

### `skin` 설정하기

Minima theme은 `minima.skin` 설정 값으로 어떤 스킨을 적용할 지 정한다. 여러
옵션들이 존재하지만, 이들을 재조합하여 만든 새 스킨을 사용할 예정임으로 다음과
같이 바꿔준다.

```yml
# classic, dark, solarized-light, solarized-dark, solarized
minima:
  skin: classic-dark 
```

## `scss` 수정하기

### `style.scss`

`assets/css/style.scss`는 Minima theme에서의 `scss` 메인 함수라고 봐도 된다.
기존 설정은 다음과 같다.

```scss
@import
  "minima/skins/{{ site.minima.skin | default: 'classic' }}",
  "minima/initialize";
```

위의 코드를 보면 `_config.yml`에 있는 `minima.skin`을 읽어서 스킨이 적용된
파일을 `initialize` 전에 불러온다는 것이다. 이 부분을 다음과 같이 바꿔주자.

{% raw %}
```scss
{% assign site_skin = site.minima.skin | default: 'classic' %}
{% case site_skin %}
  {% when "classic-dark" %}
    @import "minima/skins/classic";

    html {
      // Define CSS variables
      --brand-color :           #{$brand-color};
      --brand-color-light :     #{$brand-color-light};
      // ...
    }
  
    // Set SCSS variables based on CSS variables
    $brand-color:           var(--brand-color,$brand-color); 
    $brand-color-light:     var(--brand-color-light,$brand-color-light); 
    // ...

    // Initialize here
    @import "minima/initialize";

    html[data-theme='dark'] {
      @import "minima/skins/dark-toggle";

      // Under the condition, overwrite CSS variables
      // Then, SCSS variables are also changed
      --brand-color :           #{$brand-color};
      --brand-color-light :     #{$brand-color-light};
      // ...
    }

  {% else %}
    @import
      "minima/skins/{{ site.minima.skin | default: 'classic' }}",
      "minima/initialize";
{% endcase %}
```
{% endraw %}

`classic-dark`를 구현한 곳을 살펴보면 `css`와 `scss`의 Variable을 섞어둔 것을
알 수 있다. 한 종류의 Variable로 했다면 깔끔했겠지만, 이렇게 할 수 밖에 없던
이유는 다음과 같다.

1. `css` Variable만? `minima/initialize`에서 `scss` Variable을 필요로 함으로
   안 됨.
1. `scss` Variable만? Variable이 하나로 고정되므로 사용자가 변경 불가.

관건은 *`scss` Variable을 정의하되, 이 값을 사용자 수준에서 변경할 수 있는가*
이다. 해결 방법은 다음과 같다.

1. `css` Variable로 초기값을 설정한다.
1. 위의 값에 기반하여 `scss` Variable을 정의한다.
1. `html[data-theme='dark']`인 상황에서 `css` Variable을 Overwrite 한다.
1. 위의 값이 변함에 따라 `scss` Variable도 바뀌게 된다.

### `dark-toggle.scss`

Minima theme의 `dark.scss`를 그대로 사용하면 좋으련만, 몇 가지 수정이
필요하다. 먼저 [기존 scss 파일](https://github.com/jekyll/minima/blob/master/_sass/minima/skins/dark.scss)
을 다운로드 받고 `_sass/minima/skins/dark-toggle.scss`라는 위치와 이름으로
이동한다.

기존 파일은 다음과 같을 것이다.

```scss
@charset "utf-8";

$brand-color:           #999999 !default;
$brand-color-light:     lighten($brand-color, 5%) !default;
// ...
```

여기서 맨 첫줄의 `@charset ...`과 `!default` 키워드를 모두 제거한다.

```scss
$brand-color:           #999999;
$brand-color-light:     lighten($brand-color, 5%);
// ...
```

이와 같이 수정해야 정상적으로 Variable이 Overwrite 된다. 지금까지 CSS/SCSS 를
손 보았다. 이제 Frontend를 손 볼 차례이다.

## Frontend 추가하기

### 우선은 Toggle 기능 부터

내가 구현하고자 하는 것은 버튼을 눌렀을 때 `Light`, `Dark` 모드를 바꿀 수
있는 기능이다. Toggle은 다음과 같이 만들어 볼 수 있다.

```js
let isDarkTheme = false;

document.getElementById('btn_theme')
    .addEventListener('click', () => setTheme(!isDarkTheme));
```

여기서 `setTheme`의 구체적인 동작을 생각해보면 다음과 같다.

1. `scss` variable 변경을 위한 `<html>` dataset 속성 변경 (light or dark)
1. Toggle 버튼의 label 변경 (Light or Dark)
1. Meta Tag `theme-color` 변경
1. 마지막으로 isDarkTheme 변경

이는 다음과 같이 구현할 수 있다.

```js
const setTheme = (isDark) => {
  // Dataset and Label
  document.documentElement.dataset.theme = isDark ? 'dark' : 'light';
  document.getElementById('btn_theme').textContent = isDark ? 'Light ☼' : 'Dark ◘';

  // Meta tag
  const themeColor = getComputedStyle(document.documentElement)
      .getPropertyValue('--background-color');
  document.querySelector('meta[name="msapplication-TileColor"]')
      .setAttribute("content", themeColor);
  document.querySelector('meta[name="theme-color"]')
      .setAttribute("content", themeColor);
  
  isDarkTheme = isDark;
}
```

### Cookie에 설정값 저장하기

지금까지의 구현으로 Light와 Dark 테마를 자유자재로 변경할 수 있게 되었다.
개선해 볼만한 점으로는 사용자가 특정한 테마를 선택했을 때 다른 페이지로
넘어가더라도 그 설정이 유지되게끔 만드는 것이다.

복잡할 것 없이 사용자의 설정값을 Cookie에서 읽고 쓸 예정이다. 간단하게 Util
함수를 만들었다.

```js
const getCookie = (name) => {
  const cookie = decodeURIComponent(document.cookie)
      .split(';').map(e => e.trim())
      .find(e => e.startsWith(`${name}=`)) || '=';

  return cookie.split('=')[1];
}

const setCookie = (name, value) => {
  document.cookie = `${name}=${value}; path=/`;
}
```

이를 바탕으로 최초 렌더링 때 쿠키에 저장된 값을 읽고 (없다면 Light가 기본값),
`setTheme` 함수에서 쿠키에 값을 쓰는 동작을 추가했다.

```js
// Initialization
(() => {
  const cookieTheme = getCookie('isDarkTheme');
  setTheme(cookieTheme === 'true');
})();

const setTheme = (isDark) => {
  // ...
  setCookie('isDarkTheme', isDark);
  // ...
}
```

### 설정에서 초깃값 불러오기

지금까지는 초기화할 때 기본값으로 Light를 설정했지만, 여기서 조금 더 개선을 해
보자면 시스템 상 다크모드로 설정 되어있는가를 읽어서 설정하는 방법이 있다.
아직 모든 브라우저에서 지원하는 기능이 아니긴 하지만 `prefers-color-scheme`
값을 읽어오도록 해 보겠다.

```js
// Initialization
(() => {
  const cookieTheme = getCookie('isDarkTheme');
  let theme = false;

  if(cookieTheme === ''){
    // Get system preference
    theme = window.matchMedia('(prefers-color-scheme: dark)').matches;
  }else{
    // Get saved value
    theme = cookieTheme === 'true';
  }

  setTheme(theme);
})();
```

## References

구글 덕분에 내가 원하는 다크모드 기능을 구현할 수 있었다. 특히 아래 3개의 글이
많은 도움이 되었다.

* [How to make your Jekyll blog respect users theme (dark or light)](https://alexander-taran.github.io/2022/06/08/adopting-dark-theme-in-jekyll-blog.html)
* [벨로그에 다크 모드 적용하기](https://velog.io/@velopert/velog-dark-mode)
* [W3Schools - JavaScript Cookies](https://www.w3schools.com/js/js_cookies.asp)
