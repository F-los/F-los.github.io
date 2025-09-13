---
layout: projects
title: Projects
description: >
  개발한 프로젝트들을 소개합니다.
show_collection: projects
selected_projects:
  - exercise-community
  - marketplace
---

## 주요 프로젝트

### 🏃‍♂️ 운동 인증 커뮤니티 플랫폼
**기술 스택**: Express.js, Prisma, PostgreSQL, Vitest  
**기간**: 2023.06 - 2023.08  

실시간 운동 타이머와 인증 시스템을 갖춘 커뮤니티 플랫폼입니다.

**주요 기능**:
- 실시간 운동 타이머 서버 구현
- 운동 기록 검증 시스템
- 사용자 랭킹 시스템
- E2E 테스트 완전 자동화

**성과**:
- 동시 접속자 500명 처리 가능한 실시간 시스템 구축
- 테스트 커버리지 95% 달성

[🔗 코드 보기](https://github.com/F-los/exercise-community) | [🚀 데모](https://demo-link.com)

---

### 🛒 중고장터 & 커뮤니티 플랫폼
**기술 스택**: React, Node.js, Express, MongoDB  
**기간**: 2023.03 - 2023.05  

중고 거래와 커뮤니티 기능을 결합한 웹 플랫폼입니다.

**주요 기능**:
- 중고 상품 등록/검색/거래 시스템
- 실시간 채팅 기능
- 이미지 업로드 및 최적화
- 무한 스크롤 페이지네이션
- 사용자 평점 시스템

**기술적 도전**:
- AWS S3를 활용한 이미지 업로드 최적화
- Redis를 활용한 실시간 알림 시스템 구현
- 반응형 디자인으로 모바일/데스크톱 지원

[🔗 코드 보기](https://github.com/F-los/marketplace-community)

---

### 🎯 기타 프로젝트

#### API 게이트웨이 서비스
- **기술**: Spring Boot, Spring Cloud Gateway, Redis
- **설명**: 마이크로서비스 환경을 위한 API 게이트웨이 구현
- **특징**: 라우팅, 로드밸런싱, 인증/인가, 속도 제한 기능

#### 실시간 로그 분석 시스템
- **기술**: Python, FastAPI, Elasticsearch, Kafka
- **설명**: 대용량 로그 데이터의 실시간 수집 및 분석
- **특징**: 초당 10,000건의 로그 처리 가능

#### RESTful API 문서화 도구
- **기술**: Node.js, Express, Swagger, TypeScript
- **설명**: API 명세 자동 생성 및 문서화 도구
- **특징**: 코드 주석에서 자동으로 API 문서 생성