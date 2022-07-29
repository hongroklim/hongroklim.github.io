---
layout: project
title: IDE for Spring
spec: DevOps/Docker
excerpt: 스프링 웹개발에 필요한 모든 기능들을 컨테이너 기반으로 구현
github: https://github.com/hongroklim/ide-for-spring
image: https://logo.github.com
order: 2
---

### 기능

1. [VScode](https://hub.docker.com/r/codercom/code-server)
  * DooD(Docker out of Docker), Java 8/11, Maven 포함
  * WAS 배포, Remote Debugging 지원
1. [PostgreSQL](https://hub.docker.com/_/postgres), [Wildfly](https://hub.docker.com/r/jboss/wildfly)
  * 데이터베이스 및 WAS 초기화 포함
  * 모니터링 및 관리 기능 ([pgAdmin](https://hub.docker.com/r/dpage/pgadmin4),
  [Wildfly Console](https://docs.wildfly.org/17/Admin_Guide.html#web-management-interface)) 지원

### 특징

1. [Docker](https://www.docker.com) 기반의 통합 개발 환경
  * 환경구축 필요 없이 언제 어디서든 개발 가능
1. 서비스 구동에 필요한 초기 설정 포함
  * Dockerfile 및 [`init.sh`](https://github.com/hongroklim/ide-for-spring/blob/master/psql/init.sh),
  [`standalone.xml`](https://github.com/hongroklim/ide-for-spring/blob/master/wildfly/standalone.xml),
  [`settings.json`](https://github.com/hongroklim/ide-for-spring/blob/master/cdr/data/User/settings.json),
  [`maven-settings.xml`](https://github.com/hongroklim/ide-for-spring/blob/master/cdr/maven-settings.xml) 등에서 관리
1. 컨테이너별 로그인 정보 일원화
  * [`.env`](https://github.com/hongroklim/ide-for-spring/blob/master/.env) 에서 모든 이름, 이메일, 비밀번호 관리

### STAR

1. **Situation** <u>빈번한 개발환경 설정에 대한 불편함</u>
  * 군 복무 당시 [사이버지식정보방](https://www.law.go.kr/LSW/admRulInfoP.do?admRulSeq=2000000016242)
  컴퓨터는 JDK 설치불가로 클라우드로 개발 진행
  * 무료 크레딧 소진 시 다른 [CSP](https://en.wikipedia.org/wiki/Category:Cloud_computing_providers)
  를 이용할 때마다 개발환경 다시 구축
  * 휴가 때는 개인 노트북을 이용함으로 일관된 개발환경 설정 필요
1. **Task**
  * 스프링 웹개발에 필요한 서비스들을 구동하고 사용할 수 있어야 함
  * 간단한 명령어로 개발환경을 한 번에 설정할 수 있어야 함
1. **Action**
  * Dockerfile을 통해 일관된 환경 구축 및 프로그램 설치
  * Docker Compose를 통해 컨테이너 설정값 관리
1. **Result**
  * 개발 환경 설정의 번거로움 생략
  * 운영 환경에 관계없이 일관된 개발환경 구축 가능
