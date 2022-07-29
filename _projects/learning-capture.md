---
layout: project
title: Learning Capture
spec: Python/Django
excerpt: 비대면 강의영상을 빠르게 캡쳐하고 기록할 수 있는 웹서비스
github: https://github.com/hongroklim/learning-capture
image: https://i.imgur.com/eQ4tOSp.png
order: 1
---

### 기능

1. 강의 영상 플레이어
  * 재생속도 조절 (x0.5 ~ x2.0)
  * `Shift + Space`로 재생 또는 일시정지
  * 초단위 이동 (이전: `Shift + Ctrl` / 이후: `Shift + Alt`)
1. 화면 캡처 및 필기
  * `Ctrl + Q`로 현재 화면 캡처
  * 필기내용 입력
  * `Ctrl + Enter`로 저장
1. 수정 및 삭제
  * 오른쪽에서 필기목록에서 해당 캡처 선택
  * 캡처 또는 필기내용 수정 / 삭제
1. 필기내용 출력
  * 한 페이지에 최대 4개의 화면캡처 자동정렬

### 특징

1. Django가 제공하는 [Built-in 기능들](https://docs.djangoproject.com/en/2.2/intro/overview/)의 적극적인 활용
  * [ORM](https://en.wikipedia.org/wiki/Object–relational_mapping)을 활용해 스키마 및 쿼리 생성
  * [Class-based Views](https://docs.djangoproject.com/en/2.2/ref/class-based-views/) 상속 위주의 간결한 개발
  * 간단한 CRUD는 [Django Admin Site](https://docs.djangoproject.com/en/2.2/ref/contrib/admin/)를 통해 처리
1. Frontend 프레임워크 없이 [jQuery](https://en.wikipedia.org/wiki/JQuery)만으로 구현
  * 이유? 개발 당시 jQuery를 가장 잘 활용할 줄 알았기 때문
  * [AJAX](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX)로 캡처화면 및 필기를 업로드/조회
  * 동적 생성 요소는 HTML 소스코드 수준으로 조작

### STAR

1. **Situation** <u>강의자료가 제공되지 않는 수업의 불편함</u>
  * 평소 PPT를 출력해서 수업 중에 새롭게 언급된 내용만을 기록
  * PPT가 없는 경우 강의 화면의 내용까지 모두 기록
  * 또한 사진이나 도표를 별도로 저장하기에는 번거로움
1. **Task** <u>필기에 온전히 집중할 수 있는 서비스 구상</u>
  * 강의 영상의 캡처화면 저장을 통해 필기할 내용 감축
  * 재생, 캡처, 필기를 키보드로만 조작하여 손 움직임 최소화
  * 페이지 새로고침을 최소화하여 공부 흐름 유지
1. **Action** <u>캡처와 필기를 편리하게 할 수 있는 기능 개발</u>
  * Django의 Built-in 기능들로 Backend를 간결하게 구현하고 Frontend 개발에 집중
  * Keydown EventListener를 통해 단축키 기능 적용
  * 최초 렌더링 이후 화면 변경은 AJAX를 통해 동적으로 처리
1. **Result** <u>공부 부담 경감 및 학습 효율 향상</u>
  * 필기 시간 1/2 단축
  * 깔끔한 출력기능으로 수월하게 복습
  * (\+ 비대면 강의 증가에 따른 서비스 활용가치 상승)
