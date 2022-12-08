---
layout: project
title: Compiler Lab Project
spec: PL/Java
excerpt: JVM 기반 3 Address Code 생성
github: https://github.com/hongroklim/compiler-labs
image: https://logo.github.com
order: 7
---

### 개요

* AY22/23-1 Compiler Techniques 랩 프로젝트
* 지도 교수: Huang Shell Ying 교수님

### 구현

* Implement Lexical Analyser
  * 문자열을 입력받아 Token을 반환하는 Lexer 구현
  * 각 Token에 상응하는 Regular Expression 작성
  * JFlex 라이브러리 활용
* Implement Syntax Analyser
  * Token을 입력 받아 Context Free Grammar를 반환하는 Parser 구현
  * Non-terminal과 Terminal을 활용해 Grammar 작성
  * Beaver 라이브러리 활용
* Understand Semantic Analyser
  * Parsing 이후 Name 및 Type Check 과정을 이해
  * Abstract Syntax Tree와 *.jrag Specification을 참고
  * Jrag 라이브러리 활용
* Implement Code Generator
  * AST를 IR로 변환하는 Code Generator 구현
  * 각 문법별 Jimple 코드 생성 규칙을 작성
  * Soot 라이브러리 활용
