# 개요
티케팅과 관련된 동시성 문제를 직접 구현해 보며 학습한다.

# 기능 명세
- 사용자는 좌석을 조회할 수 있다. 각 좌석마다 예매 여부를 확인할 수 있어야 한다.
- 사용자는 좌석을 예매할 수 있다. 이 때 한번에 여러개의 좌석을 예매할 수 있다.
- 결제가 완료되어야 좌석 예매가 확정된다.
- 중복 예매를 방지해야 한다.
- 로그인은 아주 간단하게 구현한다.

# ERD
### performance
- id
- seat_count
- name

### reservation_wait
- id
- performance_id
- seats_number
- user_id

### reservation
- id
- performance_id
- seats_number
- user_id

### user
- id
- name

# API
## 좌석 조회
공연의 좌석을 조회한다. 각 좌석마다 예매 여부를 알 수 있다.
### 요청
GET /performances/{id}/seats

### 응답
```json
{
  "reserved_seats_number": [1, 3, 10, 12],
  "seat_count": 20
}
```

- 200: 조회 성공

## 좌석 예매 대기
공연에서 선택한 좌석에 대한 예약 대기를 시도한다.
### 요청
POST /performances/{id}/reservation_wait/seats/{number}
Authorization 헤더: 사용자 이름

### 응답
- 201: 예매 대기 성공
- 400: 이미 예매 대기 또는 예매 된 좌석인 경우, 존재하지 않는 performance 인 경우, 존재하지 않는 seats number인 경우
- 401: 인증 실패

## 좌석 예매 결제
예약 대기한 좌석에 대해 결제한다.
### 요청
POST /performances/{id}/reservation/seats/{number}

### 응답
- 201: 예매 결제 성공
- 400: 결제 실패, 예약 대기하지 않은 좌석인 경우, 이미 예약된 좌석인 경우, 존재하지 않는 performance 인 경우, 존재하지 않는 seats number인 경우
- 401: 인증 실패

# 동시성 제어 방법
1. seat 을 따로 둘 것인가?
seat을 따로 둔다면, 각 seat에 대한 예약 여부를 관리할 수 있다.
그리고 예약 대기에 대한 상태도 관리가 가능해진다. 추후 예약 취소 여부도 관리할 수 있다.

하지만 UK를 걸기 어려워진다. 예약, 예약 대기 테이블을 따로 관리하는 경우에는 UK를 통해 동시성을 손쉽게 제어할 수 있다.
하지만 상태를 컬럼으로 관리하게 되면 UK를 사용할 수 없고 추가적으로 동시성 제어가 필요해진다.

일단 UK로 성능이 어떤지 측정해 보고, 이후 seat에 대한 정보를 추가하는 방식으로 요구사항을 변경해보자.
