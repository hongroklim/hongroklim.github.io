---
layout: project
title: Sentiment Analysis
spec: AI/DL
excerpt: 머신러닝을 활용한 화자의 감정 상태 분석
github: https://github.com/2021hyt6-sentimentanalysis/sentiment_analysis_nb
image: https://logo.github.com
order: 3
---

**링크**:&nbsp;
[기술블로그](https://2021hyt6-techblog.github.io/projects-blog/ai/),&nbsp;
[Notebook](https://github.com/2021hyt6-sentimentanalysis/sentiment_analysis_nb)

### 개요

* 2021-2 인공지능및응용 팀 프로젝트 (지도 교수: 원영준 교수님)
* [SK Tekecom](https://www.sktelecom.com) 연계 과제 (인공지능 NUGU 스피커를 이용한 인공지능 서비스 개발)
* 인원: 3명 / 개인 역할: 데이터 전처리 및 시각화

### 역할 

1. 데이터 전처리
  * 문장으로 된 Raw Dataset에서 단어 추출 ([spaCy](https://spacy.io) 활용)
1. 데이터 분석
  * 단어별 긍정/부정 비율 계산 및 최빈 단어 도출
1. 파라미터 튜닝
  * Tokenizer에 쓰일 단어 수(`num_words`)와 padding 시 길이(`maxlen`)의 최적값 계산
1. 모델 평가
  * Epoch별 Loss와 Accuracy 시각화 및 Confusion Matrix, 분류 평가 실행

### 배운점

1. 데이터 전처리를 수행하며 좋은 품질의 데이터의 중요성 인식
1. 데이터 분석을 통해 통계적 접근법의 한계 인식, 그리고 인공지능 적용의 필요성 확인
1. 다양한 시각화 방법을 시도하며 데이터의 특성을 효과적으로 표현하는 방법에 대해 고민
