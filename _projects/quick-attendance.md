---
layout: project
title: Quick Attendance
spec: Javascript/React
excerpt: 수업조교들이 출석체크를 쉽고 빠르게 할 수 있는 모바일 웹서비스 
github: https://github.com/hongroklim/quick-atnd
image: https://github.com/hongroklim/quick-atnd/raw/main/public/logo.png
order: 0
---

### 기능

1. 출석체크를 담당하는 강의 선택
1. 'Add' 버튼을 눌러 금일 출석부 생성
1. 출결 처리를 하고자 하는 학생 검색
1. 해당 학생을 선택하여 출석/미결 처리
1. 'Export' 버튼을 눌러 엑셀 형식 출결기록 복사

### 특징 

1. Frontend만으로 구현한 서비스
  * CRUD는 브라우저의 [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)에서 수행
1. [WAL(Write-Ahead Logging)](https://en.wikipedia.org/wiki/Write-ahead_logging)을 통한 서비스 신뢰성 향상
  * 출결과 관련된 트랜잭션을 [LocalStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)에 기록
1. 필터링 및 키워드 검색 기능
  * [Hangul.js](https://github.com/e-/Hangul.js/)를 활용한 초성검색 지원

### STAR

1. **Situation** <u>수업조교를 맡으면서 출결관리에 대한 어려움</u>
  * 출석체크 방식은 조교가 수업 중에 강의실을 돌아다니며 학생들의 학생증을 확인하는 방식
  * 출석부가 학과, 학번 순으로 정렬되어 있어 출석체크에 오랜시간 소요
  * 이는 출석체크의 실수 가능성을 높이고 학생들의 집중에 방해가 됨
1. **Task** <u>신속성과 정확성에 초점을 둔 기능 개발</u>
  * 신속성: 개별 학생에 대한 검색시간 단축
  * 정확성: 인적/기술적 오류(Human/Technical Error) 최소화
1. **Action** <u>모바일 웹서비스 구현</u>
  * 필터링, 키워드 및 초성 검색, 단축키 (신속성 확보)
  * 출석인원 표시, 트랜잭션 로깅 (정확성 보장)
1. **Result** <u>출석체크 소요시간 1/10 단축</u>
  * 학생: 신속한 출결로 수업에 더욱 집중
  * 수업조교: 출결관리 업무의 부담을 경감
  * [Heroku](https://www.heroku.com)를 통해 다른 수업조교들에게도 공유
