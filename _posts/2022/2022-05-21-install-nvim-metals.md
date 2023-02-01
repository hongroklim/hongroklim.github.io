---
date: 2022-05-21 15:00:00 +0900
title: Vim으로 Scala 3 개발환경 구축하기
excerpt: 스칼라 2.x 로 개발할 때가 참 편했었지
categories: dev
---

<figure>
  <img src="https://i.imgur.com/ZxQ6HvB.png"
       alt="scala_logo">
  <figcaption>
    <a href="https://www.scala-lang.org">
      출처: Scala
    </a>
  </figcaption>
</figure>

## Scala 3를 인식 못하는 문제

Scala에 발을 들인지 4달 정도 지났다. 지금까지 Scala 2.x 로 소스코드를 작성하고
있었고 VSCode나 IntelliJ 같은 IDE는 따로 써보지는 않았고 Vim으로만 해왔다.
Scala의 세계에서는 [Scala Build Tool (SBT)](https://www.scala-sbt.org)라고
컴파일, 테스트, 실행 그리고 REPL까지 할 수 있는 아주 훌륭한 개발 도구가 있어서
Text Edit만 있으면 충분했다. 그렇다고 메모장에서 할 수는 없으니 Syntax Highlighting
가 있으면 좋겠다 싶어 [vim-scala](https://github.com/derekwyatt/vim-scala)
플러그인을 따로 설치했다.

어느 날 Scala 2.x 소스코드가 아닌 3.x로 작성된 파일을 열었는데, Vim 플러그인에서
오류가 있다고 나왔다. 정말로 오류가 있나 싶어서 봤는데 그냥 3.x 문법을 인식하지
못 하고 있었다. 찾아본 결과 해당 플러그인은 Scala 2.x 기준으로 작성된 것이라고
한다. 3.x을 지원하는 다른 플러그인은 없나 찾아봤는데... 그게 좀 복잡해 보였다.

<figure>
  <img src="https://imgur.com/q8jKKJo.png"
       alt="scala-2x-error">
  <figcaption>그냥 무시하고 수정하면 되지만 아주 심히 거슬린다.</figcaption>
</figure>

## Scala Metals

> Metals: Scala metadata + Language Server

<figure>
  <img src="https://imgur.com/gqTyC2L.png"
       alt="metals-hover">
  <figcaption>Rich IDE Features라. 아주 기대가 되는군.</figcaption>
</figure>

다행히도 [nvim-metals](https://github.com/scalameta/nvim-metals)
라고 Scala 3.x를 지원하는 플러그인이 있는데, 불행(?)하게도 그 설치과정이 좀
복잡하다. 우선 Vim이 아닌 NeoVim을 써야하고, 플러그인 설치 후 설정도 따로 해아
한다.

`README.md`를 읽어보면 위의 플러그인에 대한 설명은 다음과 같다.

> nvim-metals is a Lua plugin built to provide a better experience
> while using Metals, the Scala Language Server, with
> Neovim's built-in LSP support.

평범한 Vim 플러그인이 아님을 알 수 있다. 저 설명을 어느정도 이해하려면 다음과 같은
배경지식이 필요하다.

1. **Lua**: 프로그래밍 언어 중 하나.
'Lua plugin'이라는 말은 Lua 언어를 사용해서 만들었다는 뜻.
[위키백과 링크](https://en.wikipedia.org/wiki/Lua_(programming_language))
1. **Language Server (LS), Language Server Protocol (LSP)**:
개발 도구와 언어 서버 간의 커뮤니케이션에 사용되는 프로토콜.
[MS Docs 링크](https://docs.microsoft.com/ko-kr/visualstudio/extensibility/language-server-protocol?view=vs-2022)
1. **Metals**: 스칼라의 LSP. [링크](https://scalameta.org/metals/)

세 가지 모두 처음 듣는 단어라 감이 잘 안왔다. 이번에는 기능을 살펴보았다.
[Official Reference](https://github.com/scalameta/nvim-metals/discussions/279)

1. Goto definition
1. Find references
1. Formatting
1. Hovers
1. Signature help
1. Etc ...

IDE에서 볼 법한 기능들을 지원하는 것으로 보아, 비유하자면 Cscope의 Scala 버전이라고
이해를 하였다.

## 설치 과정

nvim-metals의 [README](https://github.com/scalameta/nvim-metals/blob/main/README.md)
에 있는 Prerequisites를 먼저 충족해야 한다.

### Install neovim

```sh
$ brew install neovim
# 출처: https://formulae.brew.sh/formula/neovim
```

### Install Coursier

#### JDK (Optional)

나는 Apple Silicon을 사용 중이라서 그냥 homebrew로 Coursier를 설치하면 안 된다.
이럴 때는 java를 따로 설치해야 한다. 나는 이미 설치되어 있어서 생략.
[AdoptOpenJDK 링크](https://adoptopenjdk.net/releases.html)

#### Coursier

```sh
$ cd /usr/local/bin
$ curl -fLo coursier https://github.com/coursier/launchers/raw/master/coursier &&
    chmod +x coursier &&
    ./coursier
# 출처: https://get-coursier.io/docs/cli-installation#jar-based-launcher
```

JDK와 Coursier가 정상적으로 설치가 완료 되었다면 아래와 같은 명령어가 잘
실행되어야 한다.

```sh
$ coursier java -version
# 출처: https://get-coursier.io/docs/cli-java
```

### Install nvim-metals

#### Install vim-plug

nvim-metals는 직접 다운로드 받지 않고 [vim-plug](https://github.com/junegunn/vim-plug)
를 통해 설치한다. 이건 neovim 쓰는 사람이들이라면 대부분 쓰는 것이다.
*(개발자 분께서 한국인이신데 전 세계 사람들이 애용하는 거 보고 살짝 국뽕)*

```sh
$ sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
# 출처: https://github.com/junegunn/vim-plug
```

#### Edit init.vim

neovim의 설정 파일은 통상적으로 `~/.config/nvim/init.vim`에 위치한다.
없다면 `touch` 명령어로 만들어주고 다음과 같이 수정한다.

```vim
call plug#begin()

Plug 'scalameta/nvim-metals'

call plug#end()
```

그리고 `nvim`에서 다음과 같은 명령어를 실행한다.

```
:PlugInstall
```

그러면 알아서 설치가 안 된 플러그인들이 설치된다. *(세상 참 편하다.)*

### Configure nvim-metals

#### Install Packer

```sh
$ git clone --depth 1 https://github.com/wbthomason/packer.nvim\
 ~/.local/share/nvim/site/pack/packer/start/packer.nvim
# 출처: https://github.com/wbthomason/packer.nvim
```

#### Create

nvim-metals 플러그인 설치와는 별도로 설정파일이 따로 필요하다. 다행히도
공식 문서에 아주 훌륭한 예시가 소개되어 있다.

neovim은 설정 파일 언어로 Lua를 지원하는데, 통상적으로 `~/.config/nvim/lua`
디렉터리 안에 위치한다. `touch ~/.config/nvim/lua/nvim-metals.lua`로
파일을 생성하고 여기에 [공식 링크](https://github.com/scalameta/nvim-metals/discussions/39)에 올라온 소스코드를 복붙한다.

#### Edit init.vim

다시 `init.vim` 파일을 열어서 아래의 설정을 추가한다.

```vim
lua require('nvim-metals')
```

그리고 `nvim`에서 다음과 같은 명령어를 실행한다.

```
:PackerUpdate
```

방금 생성한 `nvim-metals.lua` 파일을 읽어들이고 설정을 완료하게 된다.

### Test

Scala Project File 아무거나 열어서 *.scala 파일을 하나 열어본다.
딱히 달라진 것이 없기에 이게 잘 설치가 되었나 갸우뚱하지만 기다리면 다음과 같은
화면이 나온다.

<figure>
  <img src="https://imgur.com/yyAcKGc.png"
       alt="metals-welcome">
  <figcaption>오. 된건가?</figcaption>
</figure>

Welcome이라고 하는 것을 보니 잘 설치가 된 것 같다. nvim에서 `:MetalInstall`을
입력하고 기능들을 테스트 해 보았다. 정상적으로 Hover가 동작하였다....! 성공!

<figure>
  <img src="https://imgur.com/DsfCLmb.png"
       alt="metals-hover">
  <figcaption>오. 아주 잘 되었군. IDE 부럽지가 않아.</figcaption>
</figure>
