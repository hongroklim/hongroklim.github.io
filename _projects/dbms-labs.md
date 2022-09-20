---
layout: project
title: DBMS Lab Project
spec: DBMS/C++
excerpt: Disk Layer부터 Concurrency Control까지 구현
github: https://github.com/hongroklim/dbms-labs
image: https://logo.github.com
order: 5
---

### 개요

* 2021-2 데이터베이스시스템및응용 랩 프로젝트
* 지도 교수: 정형수 교수님

### 구현

* Disk Space Management
  * POSIX 라이브러리를 활용하여 File Systems R&W 직접 관리
* B+ Tree Index
  * ISAM 아키텍쳐를 따르는 B+ Tree 구현
* Buffer Management
  * dirty, pin block 속성과 LRU 알고리즘을 적용하여 Buffer 설계
* Lock Table
  * HashMap 기반으로 Mutex를 활용하여 Lock Table 관리
* Concurrency Control
  * Buffer Latch, Lock Mode (RW), Deadlock Detection, Implicit Locking, Lock Compression 구현
