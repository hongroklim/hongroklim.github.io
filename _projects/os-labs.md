---
layout: project
title: OS Lab Project
spec: OS/C
excerpt: xv6-public 기반으로 Kernel 기능 확장
github: https://github.com/hongroklim/os-labs
image: https://logo.github.com
order: 6
---

### 개요

* 2022-1 운영체제 랩 프로젝트
* 지도 교수: 정형수 교수님

### 구현

* MLFQ & Stride Scheduling
  * Multi-Level Feedback Queue와 Stride Scheduling을 적용하여 CPU Scheduler 구현
* Thread
  * Process Kernel을 기반으로 Light-Weight Process 구현
* Semaphore & Read-Write Lock
  * POSIX의 `sem`과 `pthread_rwlock` 구현
* File System
  * Triple-Indirect 구조의 File System 구현
