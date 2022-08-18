---
layout: project
title: Recycling Assistant
spec: AI/DL
excerpt: 영상을 인식하여 분리수거 요령을 설명하는 AI 가전
github: https://github.com/2021hyt6-recyclingassistant
image: https://logo.github.com
order: 4
---

**링크**:&nbsp;
[서비스 소개(한글)](https://2021hyt6-techblog.github.io/projects-blog/se-kor/),&nbsp;
[기술블로그(영문)](https://2021hyt6-techblog.github.io/projects-blog/se/)

### 개요

* 2021-1 소프트웨어공학 팀 프로젝트 (지도 교수: 원영준 교수님)
* [LG전자](https://www.lge.co.kr) 연계 과제 (인공지능을 활용한 스마트 홈 가전 제시)
* 인원: 3명 / 개인 역할: 개발문서 작성, 모델 학습 및 대시보드 개발

### 역할

1. 재활용품 이미지 데이터셋 전처리 ([TACO](http://tacodataset.org) 활용)
1. [SageMaker](https://aws.amazon.com/sagemaker/)를 통한 [Mask-RCNN](https://github.com/matterport/Mask_RCNN) 모델 학습
   ([AWS example](https://github.com/aws/amazon-sagemaker-examples/blob/main/advanced_functionality/distributed_tensorflow_mask_rcnn/mask-rcnn-scriptmode-s3.ipynb) 활용)
1. 실시간 분리배출 현황을 추적할 수 있는 대시보드 서비스 개발 ([링크](https://github.com/2021hyt6-recyclingassistant/dashboard))

### 배운점

1. 개발문서 작성을 통해 체계적인 기획 및 설계 과정을 경험
1. 학습된 모델 테스트 중 과도한 추론시간 소요로 모델 변경 ([SSD MobileNet](https://docs.openvino.ai/latest/omz_models_model_mobilenet_ssd.html) 활용)
1. 다양한 클라우드 서비스(S3, EFS, EC2 등)를 활용한 모델 학습으로 개발기간 단축
