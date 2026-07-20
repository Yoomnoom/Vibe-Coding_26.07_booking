# 프로젝트 전체 흐름 다이어그램

이 문서는 현재 프로젝트의 모든 파일, 실행 함수, 함수 간 호출 관계, 데이터 이동을 Mermaid 형식으로 정리한 문서입니다.

## 파일 관계

```mermaid
flowchart TD
  PRD["PRD.md<br/>기능 범위, 화면 구성, 데이터, 권한 기준"]
  CLAUDE["CLAUDE.md<br/>Claude 작업 규칙"]
  HANDOFF["수업_인수인계.md<br/>집에서 이어 작업할 설치/진행 메모"]
  SKILL["skills/supabase-rls-helper/SKILL.md<br/>Supabase RLS 설계 절차"]
  INDEX["index.html<br/>고객 페이지 + CSS + JavaScript"]
  SUPABASE["Supabase<br/>reservations 테이블"]

  PRD -->|"고객 페이지 요구사항 제공"| INDEX
  PRD -->|"권한/RLS 기준 제공"| SKILL
  CLAUDE -->|"작업 규칙 제공"| INDEX
  HANDOFF -->|"환경 재설치 참고"| INDEX
  SKILL -->|"RLS 정책 설계 참고"| SUPABASE
  INDEX -->|"예약 문의 insert"| SUPABASE
```

## index.html 함수 호출 관계

```mermaid
flowchart TD
  LOAD["브라우저가 index.html 로드"]
  CDN["Supabase CDN 로드<br/>@supabase/supabase-js@2"]
  CONFIG["supabaseUrl, supabaseAnonKey 설정"]
  CREATE["supabase.createClient(url, anonKey)"]
  QUERY_FORM["document.querySelector('#reservation-form')"]
  QUERY_MSG["document.querySelector('#form-message')"]
  LISTENER["form.addEventListener('submit', async event => ...)"]

  LOAD --> CDN
  CDN --> CONFIG
  CONFIG --> CREATE
  LOAD --> QUERY_FORM
  LOAD --> QUERY_MSG
  QUERY_FORM --> LISTENER

  SUBMIT["사용자가 접수 버튼 클릭"]
  PREVENT["event.preventDefault()"]
  CHECK["form.checkValidity()"]
  INVALID["입력값 비어 있음"]
  INVALID_MSG["message.textContent = '빈 칸을 모두 채워주세요.'"]
  REPORT["form.reportValidity()"]
  STOP["return"]

  VALID["입력값 모두 채움"]
  READ_VALUES["form.name.value<br/>form.phone.value<br/>form['preferred-date'].value<br/>form.request.value"]
  FROM["supabaseClient.from('reservations')"]
  INSERT["insert({ name, contact, hope_date, request })"]
  ERROR_CHECK["error 확인"]
  ERROR_MSG["message.textContent = '접수 실패: ...'"]
  SUCCESS_MSG["message.textContent = '접수되었습니다.'"]
  RESET["form.reset()"]

  SUBMIT --> LISTENER
  LISTENER --> PREVENT
  PREVENT --> CHECK
  CHECK -->|"false"| INVALID
  INVALID --> INVALID_MSG
  INVALID_MSG --> REPORT
  REPORT --> STOP
  CHECK -->|"true"| VALID
  VALID --> READ_VALUES
  READ_VALUES --> FROM
  FROM --> INSERT
  INSERT --> ERROR_CHECK
  ERROR_CHECK -->|"error 있음"| ERROR_MSG
  ERROR_MSG --> STOP
  ERROR_CHECK -->|"error 없음"| SUCCESS_MSG
  SUCCESS_MSG --> RESET
```

## 데이터 이동

```mermaid
flowchart LR
  USER["손님"]
  NAME["이름 input<br/>name"]
  CONTACT["연락처 input<br/>phone"]
  DATE["희망 날짜 input<br/>preferred-date"]
  REQUEST["요청사항 textarea<br/>request"]
  JS["submit 이벤트 핸들러"]
  PAYLOAD["insert payload<br/>{ name, contact, hope_date, request }"]
  DB["Supabase reservations 테이블"]
  DEFAULTS["DB 기본값<br/>status='대기'<br/>created_at=now()"]
  MSG["화면 메시지<br/>form-message"]

  USER --> NAME
  USER --> CONTACT
  USER --> DATE
  USER --> REQUEST
  NAME --> JS
  CONTACT --> JS
  DATE --> JS
  REQUEST --> JS
  JS -->|"유효성 실패"| MSG
  JS -->|"유효성 성공"| PAYLOAD
  PAYLOAD -->|"anon insert 정책 통과"| DB
  DEFAULTS --> DB
  DB -->|"성공"| MSG
  DB -->|"error 반환"| MSG
```

## Supabase 권한 흐름

```mermaid
flowchart TD
  ANON["익명 사용자 anon"]
  INSERT_POLICY["RLS 정책: anon_insert<br/>INSERT 허용<br/>with check true"]
  NO_SELECT["SELECT 정책 없음<br/>자동 차단"]
  NO_UPDATE["UPDATE 정책 없음<br/>자동 차단"]
  NO_DELETE["DELETE 정책 없음<br/>자동 차단"]
  TABLE["public.reservations"]

  ANON -->|"insert"| INSERT_POLICY
  INSERT_POLICY --> TABLE
  ANON -->|"select"| NO_SELECT
  ANON -->|"update"| NO_UPDATE
  ANON -->|"delete"| NO_DELETE
```

## 현재 실행 함수 목록

| 파일 | 함수/호출 | 역할 |
|---|---|---|
| `index.html` | `supabase.createClient()` | Supabase 클라이언트 생성 |
| `index.html` | `document.querySelector()` | 폼과 메시지 요소 찾기 |
| `index.html` | `form.addEventListener('submit', async event => ...)` | 접수 버튼 클릭 처리 |
| `index.html` | `event.preventDefault()` | 기본 폼 제출 막기 |
| `index.html` | `form.checkValidity()` | 필수 입력값 확인 |
| `index.html` | `form.reportValidity()` | 브라우저 기본 경고 표시 |
| `index.html` | `supabaseClient.from('reservations').insert()` | 예약 문의 DB 저장 |
| `index.html` | `form.reset()` | 성공 후 폼 초기화 |

