---
title:  Jekyll 연습장
date:   2021-02-17 22:57:54 +0900
categories: dev
---

Jekyll을 탐구하는 곳 입니다.

## post's prefix skeleton

### 사진 있을 때
```yaml
---
date: 2021-03-15 23:16:54 +0900
last_modified_at: 2021-03-18 23:12:17 +0900
title: "나의 첫번째 데스크톱"
excerpt: 부품 중에서 케이스가 가장 비싸다
header:
  overlay_image: https://user-images.githubusercontent.com/59322692/111871299-873a2300-89cc-11eb-88a7-a7e3461803ad.jpeg
categories: diary computer
---
```

### 텍스트만
```yaml
---
date: 2021-03-20 22:56:00 +0900
title: "데비안에 펌웨어 설치하기"
excerpt: 네트워크도 안 되는데 펌웨어를 설치할 일이 생겼다
categories: linux debian
---
```

## notice block

**ProTip:** Be sure to remove `/docs` and `/test` if you forked Minimal Mistakes. These folders contain documentation and test pages for the theme and you probably don't want them littering up your repo.
{: .notice}

```text
**ProTip:** Be sure to remove `/docs` and `/test` if you forked Minimal Mistakes. These folders contain documentation and test pages for the theme and you probably don't want them littering up your repo.
{: .notice}
```

**Notice Primary** : This is notice--primary block
{: .notice--primary}

**Notice Info** : This is notice--info block
{: .notice--info}

**Notice Warning** : This is notice--warning block
{: .notice--warning}

**Notice Success** : This is notice--success block
{: .notice--success}

**Notice Danger** : This is notice--danger block
{: .notice--danger}

## 이미지 캡션 달기

<figure>
  <img src="https://user-images.githubusercontent.com/59322692/111873258-cc615380-89d2-11eb-9b65-dcbd6f6caf60.png"
       alt="content_01">
  <figcaption>6.4 없는 펌웨어 읽어들이기&emsp;/&emsp;<a href="https://www.debian.org/releases/stable/i386/install.pdf.ko">출처 : 데비안 GNU/리눅스 설치 안내서 69p</a></figcaption>
</figure>

```html
<figure>
  <img src="https://user-images.githubusercontent.com/59322692/111873258-cc615380-89d2-11eb-9b65-dcbd6f6caf60.png"
       alt="content_01">
  <figcaption>6.4 없는 펌웨어 읽어들이기&emsp;/&emsp;<a href="https://www.debian.org/releases/stable/i386/install.pdf.ko">출처 : 데비안 GNU/리눅스 설치 안내서 69p</a></figcaption>
</figure>
```

## collapsible

이거 스타일 추가해야겠네

<details>
    <summary>항상 보이는 것</summary>
    눌렀을 때 보이는 것
</details>

```html
<details>
    <summary>항상 보이는 것</summary>
    눌렀을 때 보이는 것
</details>
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
{: .notice--warning}

Take a moment to look over the configuration file included with the theme. Comments have been added to provide examples and default values for most settings. Detailed explanations of each can be found below.

## Site Settings

### Theme

If you're using the Ruby gem version of the theme you'll need this line to activate it:

```yaml
theme: minimal-mistakes-jekyll
```

### Skin
