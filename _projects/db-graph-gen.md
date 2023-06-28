---
layout: project
title: DB Graph Generator
spec: Java/ANTLR
excerpt: 데이터베이스의 Dataflow Diagram 자동 분석 및 생성
github: https://github.com/hongroklim/db-graph-gen
image: https://github.com/hongroklim/quick-atnd/raw/main/public/logo.png
order: 0
---

### 개요

1. 2023년 상반기 삼성전자 DS부문 장기현장실습 과제
1. 사내 인트라넷에서의 개발 과제로 Github에는 Demo 소스코드만 공개
1. 자세한 내용은 다음 블로그 게시글 참조

### 기능

1. Oracle DB 접속 정보로 Procedure, Table/View 조회
1. Procedure 소스코드 내 Table/View 입출력 분석
1. Procedure - Table/View 관계 기반 데이터 흐름 추적
1. 데이터 흐름 자동 시각화

### 특징 

1. [ANTLR](https://www.antlr.org)를 활용한 소스코드 정적분석
1. [BFS](https://en.wikipedia.org/wiki/Breadth-first_search) 알고리즘을 활용한 데이터 흐름 추적
1. [Graphviz](https://www.graphviz.org) 라이브러리를 활용한 그래프 시각화


### STAR

1. **Situation** <u>시스템 구조 설계 직무로 현장실습 참여</u>
  * 진행 중인 과제에 참여하며 DFD(Dataflow Diagram) 문서화하는 것이 기존 목표
  * 현행 파악을 위해 기존 시스템에 대한 현행 분석이 필요
  * SCM 시스템 중 MP(Master Plan) 모듈에 대한 소스코드 분석 시작
1. **Task** <u>방대하고 복잡한 데이터베이스 Procedure</u>
  * 2,000여개의 Procedure 및 외부 I/F 및 Batch 존재
  * 각 Procedure의 소스코드 또한 길고 복잡해 분석하는 데 오랜 시간 소요
  * 분석 후 DFD 생성에도 단순/반복적인 작업 필요
1. **Action** <u>라이브러리 및 알고리즘을 활용한 문제 해결</u>
  * **Procedure 입출력 분석**: Parse Tree 생성 후 DML(INSERT, SELECT 등) 식별
  * **데이터 흐름 추적**: 수집된 입출력 정보 기반 재귀적인 데이터 흐름 탐색
  * **그래프 시각화**: 추적 결과를 바탕으로 그래프 자동 시각화
1. **Result** <u>DFD 생성을 위한 전 과정 자동화</u>
  * Oracle DB 접속 정보 만으로 해당 Schema의 DFD 전부 생성 가능
  * 완전 탐색 기반의 데이터 흐름 추적으로 누락 없는 분석 결과 제공
  * 단순/반복적인 작업 자동화를 통한 고품질 업무 집중 환경 조성
