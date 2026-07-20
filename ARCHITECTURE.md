# 프로젝트 구조 문서

이 문서는 베이킹 클래스 예약 관리 서비스의 전체 구조와 데이터 흐름을 설명합니다.

## 1. 전체 구성

```text
손님
  ↓
index.html
  ↓
Supabase JavaScript Client
  ↓
Supabase reservations 테이블
```

현재 구현된 화면은 고객 페이지 `index.html` 하나입니다. 관리자 화면은 아직 만들지 않았습니다.

## 2. 파일 구조

```text
수업18일차_코덱스로실습/
├─ index.html
├─ PRD.md
├─ CLAUDE.md
├─ README.ME
├─ PROJECT_FLOW_DIAGRAM.md
├─ ARCHITECTURE.md
├─ 수업_인수인계.md
└─ skills/
   └─ supabase-rls-helper/
      └─ SKILL.md
```

## 3. 파일 역할

- `index.html`: 고객 페이지입니다. HTML, CSS, JavaScript가 한 파일 안에 있습니다.
- `PRD.md`: 서비스 기획과 기능 범위를 정리한 문서입니다.
- `CLAUDE.md`: Claude가 작업할 때 지켜야 하는 규칙 문서입니다.
- `README.ME`: 프로젝트를 처음 보는 사람이 읽는 전체 소개 문서입니다.
- `PROJECT_FLOW_DIAGRAM.md`: Mermaid 다이어그램으로 흐름을 보여주는 문서입니다.
- `수업_인수인계.md`: 집에서 이어서 작업하기 위한 메모입니다.
- `skills/supabase-rls-helper/SKILL.md`: Supabase RLS 정책 설계용 로컬 스킬입니다.

## 4. index.html 내부 구조

`index.html`은 세 부분으로 나뉩니다.

1. HTML 구조
   - 대표 문구
   - 클래스 소개 카드
   - 예약 문의 폼

2. CSS
   - 모바일에서 폼을 쓰기 편하도록 입력칸과 버튼을 크게 설정합니다.
   - 따뜻한 베이킹 클래스 느낌의 색을 사용합니다.

3. JavaScript
   - Supabase 클라이언트를 만듭니다.
   - 접수 버튼 클릭을 감지합니다.
   - 필수 입력값을 확인합니다.
   - Supabase `reservations` 테이블에 데이터를 저장합니다.
   - 성공 또는 실패 메시지를 화면에 보여줍니다.

## 5. 데이터 흐름

```text
손님 입력
  ↓
예약 폼
  ↓
JavaScript submit 이벤트
  ↓
Supabase insert
  ↓
reservations 테이블
```

폼 입력값과 DB 컬럼 연결:

```text
name              → reservations.name
phone             → reservations.contact
preferred-date    → reservations.hope_date
request           → reservations.request
```

`status`는 화면에서 보내지 않습니다. 데이터베이스 기본값 `대기`가 자동으로 들어갑니다.

## 6. Supabase 구조

테이블:

```text
reservations
```

컬럼:

- `id`: 자동 증가 기본키
- `name`: 손님 이름
- `contact`: 연락처
- `hope_date`: 희망 날짜
- `request`: 요청사항
- `status`: 상태, 기본값 `대기`
- `created_at`: 접수 시각, 기본값 `now()`

## 7. 권한 구조

RLS는 켜져 있습니다.

현재 정책:

```text
anon_insert
```

현재 허용:

- 익명 사용자 `anon`: 예약 문의 등록 가능

현재 차단:

- 익명 사용자 `anon`: 조회 불가
- 익명 사용자 `anon`: 수정 불가
- 익명 사용자 `anon`: 삭제 불가

조회, 수정, 삭제 정책은 나중에 관리자 화면을 만들 때 추가합니다.

## 8. 아직 없는 구조

아직 만들지 않은 파일:

- `admin.html`
- `styles.css`
- `app.js`
- `admin.js`

현재는 실습을 단순하게 유지하기 위해 `index.html` 한 파일 안에 화면, 스타일, 동작을 함께 넣었습니다.

