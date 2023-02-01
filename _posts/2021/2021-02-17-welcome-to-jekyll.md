---
date:   2021-02-17 22:57:54 +0900
title:  나만 보는 Jekyll 연습장
excerpt: 자주 쓰는 코드들은 여기에 모아두었음
categories: dev
---

## Post 맨 위에

- Tech category: `blockchain`, `bigdata`, `web`, `java`, `dev`
- 나머지: `review`, `diary`

```yaml
---
date: 202-0-0 00:00:00 +0900
title: 
excerpt: 
categories:
---
```

```yaml
---
date: 2021-03-20 22:00:00 +0900
title: 데비안에 펌웨어 설치하기
excerpt: 네트워크도 안 되는데 펌웨어를 설치할 일이 생겼다
categories: dev
---
```

{% comment %}
### 사진 있을 때

```yaml
---
date: 2021-03-15 23:16:54 +0900
last_modified_at: 2021-03-18 23:12:17 +0900
title: "나의 첫번째 데스크톱"
excerpt: 부품 중에서 케이스가 가장 비싸다
header:
  overlay_image: https://i.imgur.com/XsxgBHy.jpg
categories: diary computer
---
```
{% endcomment %}

## notice block

**ProTip:** Be sure to remove `/docs` and `/test` if you forked Minimal Mistakes. These folders contain documentation and test pages for the theme and you probably don't want them littering up your repo.
{: .notice}

```text
{: .notice}
```

```text
**ProTip:** Be sure to remove `/docs` and `/test` if you forked Minimal Mistakes. These folders contain documentation and test pages for the theme and you probably don't want them littering up your repo.
{: .notice}
```

## 이미지와 캡션

<figure>
  <img src="https://i.imgur.com/6Yq5WUf.png"
       alt="content_01">
  <figcaption>6.4 없는 펌웨어 읽어들이기&emsp;/&emsp;<a href="https://www.debian.org/releases/stable/i386/install.pdf.ko">출처 : 데비안 GNU/리눅스 설치 안내서 69p</a></figcaption>
</figure>

```html
<figure>
  <img src=""
       alt="">
  <figcaption>
    &emsp;/&emsp;
    <a href="">
    </a>
  </figcaption>
</figure>
```

```html
<figure>
  <img src="https://i.imgur.com/6Yq5WUf.png"
       alt="content_01">
  <figcaption>
    6.4 없는 펌웨어 읽어들이기
    &emsp;/&emsp;
    <a href="https://www.debian.org/releases/stable/i386/install.pdf.ko">
      출처 : 데비안 GNU/리눅스 설치 안내서 69p
    </a>
  </figcaption>
</figure>
```

## 접히는 거

<details>
    <summary>항상 보이는 것</summary>
    눌렀을 때 보이는 것
</details>

```html
<details>
    <summary></summary>
</details>
```

```html
<details>
    <summary>항상 보이는 것</summary>
    눌렀을 때 보이는 것
</details>
```

## liquid 문법

[링크](https://shopify.github.io/liquid/tags/control-flow/)

```liquid{% raw %}
{% comment %}
{% endcomment %}

{% post_url %}
{% endraw %}
{% raw %}{%{% endraw %} raw %}
{% raw %}{%{% endraw %} endraw %}
```

```liquid{% raw %}
{% # 주석 %}
{% comment %}
{% assign verb = "converted" %}
{% endcomment %}

{% # 내부 URL %}
{% post_url 2022/2022-07-05-room-19 %}

{% # 문자 그대로 %}{% endraw %}
{% raw %}{%{% endraw %} raw %}
{% raw %}{%{% endraw %} endraw %}
```

## something

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

## Somethings222

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

### Somethings3333

Jekyll also offers powerful support for code snippets:

### Something444

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

Settings that affect your entire site can be changed in [Jekyll's configuration file](https://jekyllrb.com/docs/configuration/): `_config.yml`, found in the root of your project. If you don't have this file you'll need to copy or create one using the theme's [default `_config.yml`](https://github.com/mmistakes/minimal-mistakes/blob/master/_config.yml) as a base.

**Note:** for technical reasons, `_config.yml` is NOT reloaded automatically when used with `jekyll serve`. If you make any changes to this file, please restart the server process for them to be applied.
{: .notice}

Take a moment to look over the configuration file included with the theme. Comments have been added to provide examples and default values for most settings. Detailed explanations of each can be found below.
