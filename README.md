# YumQuest

> 음식점, 리뷰, 미션, 회원 인증 흐름을 Spring Boot로 구현한 지역 기반 맛집 미션 API 서버

YumQuest는 UMC 10기 Spring Boot 학습 과정에서 만든 백엔드 프로젝트입니다. 단순 CRUD를 넘어서 회원 인증, JWT 기반 보안, OAuth2 로그인, JPA 연관관계, 공통 응답 포맷, 도메인별 예외 처리, 페이지네이션과 커서 기반 조회까지 Spring 백엔드의 핵심 흐름을 직접 구현하는 데 초점을 두었습니다.

현재 Gradle 프로젝트명은 `umc10th`, 메인 패키지는 `com.example.umc10th`입니다.

## Tech Stack

| Category | Stack |
| --- | --- |
| Language | Java 21 |
| Framework | Spring Boot 4.0.5 |
| Web | Spring WebMVC |
| Persistence | Spring Data JPA, Hibernate |
| Database | MySQL |
| Security | Spring Security, JWT, OAuth2 Client |
| API Docs | Springdoc OpenAPI, Swagger UI |
| Build | Gradle |
| Test | JUnit Platform, Spring Boot Test |
| Utility | Lombok |

## Features

### Member

- 회원가입
- BCrypt 기반 비밀번호 암호화
- 이메일/비밀번호 로그인
- JWT Access Token 발급
- 인증된 사용자 마이페이지 조회
- 일반 로그인과 OAuth 회원을 함께 다루기 위한 `SocialType`, `socialUid` 구조

### Mission

- 가게별 미션 생성
- 지역별 수행 가능 미션 조회
- 사용자의 미션 상태별 목록 조회
- 가게 미션 목록 커서 기반 페이지네이션
- `READY`, `CHALLENGING`, `COMPLETE` 등 미션 상태 관리

### Review

- 특정 가게 리뷰 작성
- 내가 작성한 리뷰 목록 조회
- ID 기준 커서 페이지네이션
- 평점 기준 커서 페이지네이션

### Store

- 지역과 가게의 JPA 연관관계 모델링
- 미션, 리뷰 도메인에서 가게 정보를 기준으로 비즈니스 로직 처리

### Global

- `ApiResponse<T>` 기반 공통 응답 포맷
- `ProjectException` 기반 도메인 예외 구조
- `@RestControllerAdvice` 기반 전역 예외 처리
- `BaseEntity` 기반 생성/수정/삭제 시간 관리
- Swagger UI와 JWT Bearer 인증 설정

## Domain Model

```text
Member
  |-- MemberFood -- Food
  |-- MemberTerm -- Term
  |-- MemberMission -- Mission
  |-- Review

Region
  |-- Store
        |-- Mission
        |-- Review

Review
  |-- ReviewPicture
```

핵심 엔티티는 `Member`, `Store`, `Region`, `Mission`, `MemberMission`, `Review`입니다. 각 도메인은 controller, service, repository, dto, converter, exception 패키지로 분리되어 Spring 계층형 구조를 따릅니다.

## Project Structure

```text
src/main/java/com/example/umc10th
|-- Umc10thApplication.java
|-- domain
|   |-- BaseEntity.java
|   |-- member
|   |-- mission
|   |-- review
|   `-- store
`-- global
    |-- apiPayload
    |-- config
    `-- security
```

## API Overview

| Method | Endpoint | Description | Auth |
| --- | --- | --- | --- |
| POST | `/api/auth/sign-up` | 회원가입 | No |
| POST | `/api/auth/login` | 로그인 및 JWT 발급 | No |
| POST | `/api/v1/members/me` | 내 정보 조회 | Yes |
| POST | `/api/v1/members/missions` | 내 미션 목록 조회 | Yes |
| GET | `/api/v1/regions/{regionId}/missions` | 지역별 미션 조회 | Yes |
| POST | `/api/v1/stores/{storeId}/missions` | 가게 미션 생성 | Yes |
| GET | `/api/v1/stores/{storeId}/missions` | 가게 미션 커서 조회 | Yes |
| POST | `/api/v1/stores/{storeId}/reviews` | 리뷰 작성 | Yes |
| GET | `/api/v1/members/{memberId}/reviews` | 내가 작성한 리뷰 조회 | Yes |

Swagger UI는 애플리케이션 실행 후 아래 경로에서 확인할 수 있습니다.

```text
http://localhost:8080/swagger-ui/index.html
```

## Response Format

모든 성공/실패 응답은 공통 포맷을 사용합니다.

```json
{
  "isSuccess": true,
  "code": "COMMON200",
  "message": "요청에 성공했습니다.",
  "result": {}
}
```

도메인별 성공/에러 코드는 `BaseSuccessCode`, `BaseErrorCode`를 구현한 enum으로 관리합니다.

## Getting Started

### 1. Requirements

- Java 21
- MySQL 8.x
- Gradle Wrapper

### 2. Clone

```bash
git clone https://github.com/cha-hyunwoo/YumQuest.git
cd YumQuest
```

### 3. Configure

`src/main/resources/application.properties`에 로컬 DB와 JWT 설정을 추가합니다.

```properties
spring.application.name=umc10th

spring.datasource.url=jdbc:mysql://localhost:3306/yumquest
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.show-sql=true

jwt.token.secretKey=your-very-long-secret-key-at-least-32-bytes
jwt.token.expiration.access=3600000
```

OAuth2 로그인을 사용할 경우 Kakao 등 provider 설정을 함께 추가해야 합니다.

### 4. Run

Windows:

```bash
gradlew.bat bootRun
```

macOS/Linux:

```bash
./gradlew bootRun
```

### 5. Test

Windows:

```bash
gradlew.bat test
```

macOS/Linux:

```bash
./gradlew test
```

## What I Focused On

- Spring MVC에서 Controller, Service, Repository 책임 분리
- DTO와 Entity를 분리하고 Converter를 통한 변환 흐름 구성
- JPA 연관관계와 지연 로딩 기반 도메인 모델링
- `Page`, `Slice`를 활용한 페이지네이션 구현
- JWT Filter를 Security Filter Chain에 연결
- 공통 응답/예외 구조를 통한 일관된 API 설계
- Swagger 문서화를 통한 API 탐색성 개선

## Author

| Name | GitHub |
| --- | --- |
| cha-hyunwoo | [@cha-hyunwoo](https://github.com/cha-hyunwoo) |
