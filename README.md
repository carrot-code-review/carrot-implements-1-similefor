# 골든 티켓 이벤트 서비스

## 1. 과제 안내

안녕하세요. 당근 코드리뷰 모임 과제입니다.

과제에 대한 문의는 편하게 해주세요.

---

## 2. 서비스 배경

저희 서비스에서는 특별 시즌마다 **골든 티켓 이벤트**를 진행합니다.

골든 티켓은 한정 수량으로 발급되며, 티켓을 획득한 사용자에게는 프리미엄 혜택이 제공됩니다. 지난 이벤트에서는 오픈과 동시에 수만 명의 사용자가 몰려 서버에 큰 부하가 발생했습니다.

또한 일부 사용자의 네트워크 환경에서 요청이 중복 전송되는 문제가 보고되었고, 이로 인해 CS 문의가 급증했습니다.

이번 과제에서는 이러한 상황에서도 안정적으로 동작하는 티켓 발급 시스템을 구현해주세요.

---

## 3. 기능 요구사항

### 이벤트

- 골든 티켓은 이벤트별로 **한정 수량**이 존재합니다.
- 이벤트는 **시작 시간**과 **종료 시간**이 있습니다.
- 이벤트 기간 외에는 티켓을 발급받을 수 없습니다.
- 이벤트별로 **일일 발급 한도**를 설정할 수 있습니다.

### 티켓 발급

- 사용자는 **선착순**으로 티켓을 발급받을 수 있습니다.
- 사용자당 하나의 이벤트에서 **1장만** 발급받을 수 있습니다.
- 이미 티켓을 발급받은 사용자가 다시 요청하면 **기존 티켓 정보를 반환**합니다.
- 총 수량이 모두 소진된 경우 발급에 실패합니다.
- 일일 발급 한도에 도달한 경우 발급에 실패합니다.
- 일일 발급 한도는 **자정(00:00:00)**에 초기화됩니다.

---

## 4. API 명세

### 필수 API

| API                              | 설명              |
| -------------------------------- | ----------------- |
| `POST /events`                   | 이벤트 생성       |
| `GET /events/:eventKey`          | 이벤트 현황 조회  |
| `POST /events/:eventKey/tickets` | 티켓 발급 요청    |
| `GET /users/:userId/tickets`     | 내 티켓 목록 조회 |

---

### POST /events

이벤트를 생성합니다.

**Request**

```json
{
  "eventKey": "golden-2026-summer",
  "name": "2026 여름 골든티켓",
  "totalQuantity": 100,
  "dailyLimit": 10,
  "startAt": "2026-01-15T00:00:00Z",
  "endAt": "2026-01-20T23:59:59Z"
}
```

**Response `201 Created`**

```json
{
  "eventKey": "golden-2026-summer",
  "name": "2026 여름 골든티켓",
  "totalQuantity": 100,
  "dailyLimit": 10,
  "remainingQuantity": 100,
  "startAt": "2026-01-15T00:00:00Z",
  "endAt": "2026-01-20T23:59:59Z",
  "createdAt": "2026-01-10T09:00:00Z"
}
```

**Response `409 Conflict`** - 이미 존재하는 eventKey

```json
{
  "statusCode": 409,
  "message": "이미 존재하는 이벤트입니다."
}
```

---

### GET /events/:eventKey

이벤트 현황을 조회합니다.

**Response `200 OK`**

```json
{
  "eventKey": "golden-2026-summer",
  "name": "2026 여름 골든티켓",
  "totalQuantity": 100,
  "dailyLimit": 10,
  "remainingQuantity": 23,
  "todayIssuedCount": 7,
  "startAt": "2026-01-15T00:00:00Z",
  "endAt": "2026-01-20T23:59:59Z",
  "isActive": true
}
```

**Response `404 Not Found`**

```json
{
  "statusCode": 404,
  "message": "존재하지 않는 이벤트입니다."
}
```

---

### POST /events/:eventKey/tickets

티켓 발급을 요청합니다.

**Request**

```json
{
  "userId": 1
}
```

**Response `201 Created`** - 신규 발급

```json
{
  "ticketId": "abc-123",
  "userId": 1,
  "eventKey": "golden-2026-summer",
  "issuedAt": "2026-01-15T10:00:00Z"
}
```

**Response `200 OK`** - 이미 발급된 티켓 (중복 요청)

```json
{
  "ticketId": "abc-123",
  "userId": 1,
  "eventKey": "golden-2026-summer",
  "issuedAt": "2026-01-15T10:00:00Z"
}
```

**Response `404 Not Found`** - 이벤트 없음

```json
{
  "statusCode": 404,
  "message": "존재하지 않는 이벤트입니다."
}
```

**Response `400 Bad Request`** - 이벤트 기간 아님

```json
{
  "statusCode": 400,
  "message": "이벤트 기간이 아닙니다."
}
```

**Response `409 Conflict`** - 총 수량 소진

```json
{
  "statusCode": 409,
  "message": "티켓이 모두 소진되었습니다."
}
```

**Response `409 Conflict`** - 일일 한도 초과

```json
{
  "statusCode": 409,
  "message": "오늘의 발급 한도에 도달했습니다. 내일 다시 시도해주세요."
}
```

---

### GET /users/:userId/tickets

사용자의 티켓 목록을 조회합니다.

**Response `200 OK`**

```json
[
  {
    "ticketId": "abc-123",
    "eventKey": "golden-2026-summer",
    "eventName": "2026 여름 골든티켓",
    "issuedAt": "2026-01-15T10:00:00Z"
  }
]
```

---

### 선택 API

| API                       | 설명                    |
| ------------------------- | ----------------------- |
| `GET /events`             | 진행 중인 이벤트 목록   |
| `PATCH /events/:eventKey` | 이벤트 활성/비활성 처리 |

---

## 5. 기술 요구사항

- Typescript, NestJS 사용
- PostgreSQL 사용 (Docker로 제공)
- REST API 형태로 구현
- 제공된 Docker 환경에서 실행 가능하도록 구성

---

## 6. 실행 방법

### Docker (권장)

```bash
# 컨테이너 실행 (PostgreSQL + App)
docker compose up

# 백그라운드 실행
docker compose up -d

# 종료
docker compose down
```

### 로컬 실행

```bash
# 의존성 설치
yarn install

# 개발 서버 실행
yarn start:dev

# 빌드
yarn build

# 프로덕션 실행
yarn start:prod
```

서버가 실행되면 `http://localhost:3000`에서 API를 테스트할 수 있습니다.

---

## 7. 과제 진행 방법

### 일정

- 수행 기간: **2주**
- 제출 마감: 모임 전날 자정까지

### 진행 순서

1. 공유된 **초대 링크**를 클릭합니다.
2. GitHub 계정으로 로그인하면 개인 저장소가 생성됩니다.
3. 생성된 저장소를 clone하여 과제를 진행합니다.
4. 완료 후 **main 브랜치에 push**합니다.
5. README에 설계 의도를 작성해주세요.

### 제출물

- 전체 소스 코드
- 아래 설계 의도 작성

---

## 설계 의도

> **Q. 설계 시 고려했던 점을 자유롭게 작성해주세요.**

```
여기에 작성해주세요.
```

---

## 8. 참고 사항

- 서비스 기준 시간은 **KST (UTC+9)**입니다.
- 과제 내용은 외부에 공개하지 말아 주세요.

감사합니다.