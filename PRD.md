# [PRD] 사내 실시간 메신저 웹 애플리케이션 (단일 HTML 프로토타입)

> **문서 버전:** 1.0.0  
> **대상 구현체:** 단일 파일 웹 애플리케이션 (`index.html`)  
> **구현 주체:** Antigravity 2.0 AI Agent  
> **작성 목적:** 외부 서버나 번들러 없이 브라우저에서 즉시 실행되는 사내 메신저 UI/동작 프로토타입 구현 및 향후 Supabase 연동 준비

---

## 1. 프로젝트 개요 (Overview)

본 문서는 프론트엔드 빌드 도구 및 백엔드 서버 없이 브라우저에서 바로 열 수 있는 **단일 `index.html` 기반 사내 실시간 메신저 웹페이지**의 제품 요구 사양서(PRD)이다.  
현재 단계에서는 백엔드(Supabase) 및 Google OAuth 인증을 직접 호출하지 않고, 순수 JavaScript 인메모리(Array) 상태 관리로 실제 동작과 동일한 사용자 경험(로그인, 화면 전환, 메뉴 탭 이동, 메시지 송수신)을 제공한다.  
추후 Supabase 연동 시 손쉽게 교체할 수 있도록 연동 대상 코드 위치에 `// TODO: Supabase 연동` 주석을 표준화하여 명시한다.

---

## 2. 기술 스택 및 파일 제약 사항

1. **파일 구조:** 단일 파일 `index.html` (CSS는 `<style>`, JavaScript는 `<script>` 내부에 모두 포함)
2. **외부 종속성:** 불필요 (순수 Vanilla HTML5, CSS3 Flexbox/Grid, Modern ES6+ JavaScript)
3. **브라우저 지원:** Chrome, Safari, Edge 등 최신 브라우저
4. **상태 보존:** 새로고침 시 초기화 허용 (인메모리 배열 기반 관리)

---

## 3. 디자인 시스템 및 컬러 팔레트 (Design System)

| 구분 | 색상명 | Hex 코드 | 적용 대상 |
| :--- | :--- | :--- | :--- |
| **사이드바 배경** | Dark Navy | `#1E2238` | 좌측 240px 사이드바 배경 |
| **사이드바 메뉴 호버** | Navy Hover | `#282D4A` | 비활성 메뉴 마우스 호버 시 |
| **사이드바 활성 탭** | Navy Active | `#2E3557` | 현재 선택된 메뉴 배경 (좌측 파란 보더 포인트) |
| **본문/컨텐츠 배경**| Pure White | `#FFFFFF` | 메인 헤더, 채팅창 바디, 입력 바 |
| **보조 배경** | Warm/Cool Gray | `#F8F9FA` | 채팅 메시지 영역 배경, 로그인 페이지 배경 |
| **포인트/브랜드** | Royal Blue | `#2563EB` | 전송 버튼, 활성 인디케이터, 내 말풍선 배경 |
| **포인트 호버** | Deep Blue | `#1D4ED8` | 파란색 버튼 호버 시 |
| **상대방 말풍선** | Soft Gray | `#E5E7EB` | 상대방 메시지 배경 (글자색: `#1F2937`) |
| **경계선** | Border Gray | `#E5E7EB` | 헤더 하단선, 입력창 테두리, 카드 외곽선 |
| **텍스트 (기본)** | Dark Gray | `#1F2937` | 본문 기본 텍스트 |
| **텍스트 (보조)** | Muted Gray | `#6B7280` | 타임스탬프, 부제목, 플레이스홀더 |

---

## 4. 화면 및 컴포넌트 상세 요구사항

```
[화면 구조도]
+-------------------------------------------------------------+
| 화면 1: 로그인 전 (Centered Card)                            |
|                 +-----------------------+                   |
|                 | 🏢 사내 메신저        |                   |
|                 | [ Google 계정 로그인 ] |                   |
|                 +-----------------------+                   |
+-------------------------------------------------------------+

+-------------------------------------------------------------+
| 화면 2: 로그인 후 (메인 레이아웃)                            |
| +-----------+---------------------------------------------+ |
| | 사이드바  | 상단 헤더: [메뉴명]            [👤 게스트 | 로그아웃] |
| | (240px)   +---------------------------------------------+ |
| | 🚀 메신저 | 메인 컨텐츠 영역:                           | |
| |           | - 사내 채팅방 선택 시:                       | |
| | 💬 채팅방 |   [메시지 스크롤 영역 (상대: 회색 / 나: 파란색)] | |
| |           |   [메시지 입력 인풋창 + 전송 버튼]          | |
| | 📚 게시판 | - 게시판 선택 시:                           | |
| |           |   [ "준비 중입니다" 안내 카드 ]             | |
| +-----------+---------------------------------------------+ |
+-------------------------------------------------------------+
```

### 4.1. 로그인 전 화면 (`#login-screen`)

1. **배치 및 레이아웃:**
   - 전체 화면 100vh 수평/수직 중앙 정렬 (`display: flex; justify-content: center; align-items: center; background: #F3F4F6;`).
2. **로그인 카드 컴포넌트:**
   - 너비: `400px`, 패딩: `40px 32px`, 배경: `#FFFFFF`, 모서리 곡률: `16px`, 그림자: `0 10px 25px rgba(0, 0, 0, 0.08)`.
   - 상단 앱 로고 아이콘(💬 또는 사내 로고 SVG) + 제목: "사내 메신저" (`font-size: 24px; font-weight: 700; color: #1F2937;`).
   - 부제목: "팀원들과 빠르고 안전하게 소통하세요" (`color: #6B7280; font-size: 14px; margin-bottom: 32px;`).
3. **"Google 계정으로 로그인" 버튼:**
   - 너비: 100%, 높이: `48px`, 둥근 모서리 `8px`.
   - 테두리: `1px solid #D1D5DB`, 배경: `#FFFFFF`, 글자색: `#374151`, `font-weight: 600`.
   - 좌측에 Google 컬러 'G' 로고(SVG 인라인) 배치.
   - 호버 시: 배경 `#F9FAFB`, 테두리 `#9CA3AF`.
4. **동작 로직:**
   - 클릭 시 이벤트 핸들러 실행:
     ```javascript
     // TODO: Supabase 연동 - Google OAuth 로그인
     // const { data, error } = await supabase.auth.signInWithOAuth({ provider: 'google' });
     ```
   - 임시 동작: `currentUser = { name: "게스트", email: "guest@company.com" }` 세팅 후 `#login-screen` 숨김(`display: none`), `#main-screen` 표시(`display: flex`).

---

### 4.2. 로그인 후 메인 레이아웃 (`#main-screen`)

전체 화면 높이 `100vh`, 가로 폭 `100vw`, `overflow: hidden`, `display: flex; flex-direction: row;`.

#### A. 좌측 사이드바 (`aside.sidebar`)
- **규격:** 고정 너비 `240px`, 높이 `100%`, 배경색: `#1E2238`, 텍스트 색상: `#E5E7EB`.
- **상단 브랜딩 영역 (높이 `64px`, 패딩 `0 20px`, 하단 보더 `1px solid rgba(255,255,255,0.08)`):**
  - 앱 이름: `🚀 사내 메신저` (White, `font-weight: 700; font-size: 17px;`).
- **메뉴 목록 (`ul.nav-menu`):**
  - 패딩: `16px 8px`, 항목 간 간격: `4px`.
  - 항목 1: `💬 사내 채팅방` (기본 활성화)
  - 항목 2: `📚 게시판`
- **메뉴 아이템 스타일:**
  - 패딩 `10px 14px`, 둥근 모서리 `8px`, 폰트 크기 `14px`, 커서 `pointer`, 전환 애니메이션 `background 0.15s ease`.
  - 마우스 호버: 배경 `#282D4A`, 글자색 `#FFFFFF`.
  - 활성화 상태(Active class): 배경 `#2E3557`, 글자색 `#FFFFFF`, 글씨 두께 `600`, 좌측 `3px solid #2563EB` 인디케이터.
- **클릭 동작:**
  - 클릭된 메뉴로 `active` 클래스 이전.
  - 우측 메인 영역의 헤더 제목 변경 ("사내 채팅방" ↔ "게시판").
  - 본문 뷰 전환: `chat-view` 표시 vs `board-view` 표시.

---

#### B. 우측 상단 헤더 (`header.top-header`)
- **규격:** 높이 `64px`, 배경색: `#FFFFFF`, 하단 보더: `1px solid #E5E7EB`, 패딩: `0 24px`.
- **배치:** Flexbox (`justify-content: space-between; align-items: center;`).
- **좌측:** 현재 활성화된 메뉴명 표시 (예: "💬 사내 채팅방", `font-size: 18px; font-weight: 700; color: #111827;`).
- **우측 유저 프로필 및 액션:**
  - 프로필 영역: 원형 아바타 (`32px x 32px`, 파란색 원 배경에 이모지 👤 또는 이니셜 "G") + 사용자 이름 ("게스트 님", `font-size: 14px; font-weight: 600; color: #374151;`).
  - 로그아웃 버튼:
    - 라벨: "로그아웃", 폰트 크기: `13px`, 패딩: `6px 12px`, 테두리: `1px solid #E5E7EB`, 배경: `#FFFFFF`, 색상: `#4B5563`, 둥근 모서리 `6px`.
    - 호버 시: 배경 `#FEE2E2`, 테두리 `#FCA5A5`, 글자색 `#DC2626`.
    - 클릭 동작:
      ```javascript
      // TODO: Supabase 연동 - 로그아웃
      // await supabase.auth.signOut();
      ```
      임시 동작: `currentUser = null` 처리 후 `#main-screen` 숨김, `#login-screen` 표시, 입력값 초기화.

---

#### C. 메인 컨텐츠 영역 1: 사내 채팅방 (`#chat-view`)
`flex: 1; display: flex; flex-direction: column; height: calc(100vh - 64px); background: #F8F9FA;`

1. **메시지 스크롤 컨테이너 (`#chat-messages`):**
   - `flex: 1; overflow-y: auto; padding: 24px; display: flex; flex-direction: column; gap: 16px;`
   - 스크롤바 커스텀: 슬림 스크롤바 적용 (`width: 6px;`).
2. **초기 더미 메시지 (3개 기본 렌더링):**
   - [상대방 1] 김철수 팀장 (10:30 AM): "안녕하세요! 이번 주 스프린트 배포 일정 안내드립니다."
   - [상대방 2] 이영희 (10:32 AM): "네 팀장님, 프론트엔드 작업 오늘 중으로 마무리 예정입니다."
   - [내 메시지] 게스트 (10:35 AM): "확인했습니다! 저도 채팅 UI 프로토타입 구현 완료했습니다."
3. **말풍선 UI 규칙:**
   - **상대방 메시지 (Left Align):**
     - 구조: `[아바타(36px)]` + `[발신자 이름 + 말풍선 + 시간]`
     - 말풍선 스타일: 배경 `#E5E7EB`, 글자색 `#1F2937`, 모서리 `4px 16px 16px 16px`, 패딩 `10px 14px`, 최대 너비 `65%`, 줄바꿈 `word-break: break-word`.
     - 타임스탬프: 말풍선 우측 하단, 폰트 크기 `11px`, 색상 `#9CA3AF`.
   - **내 메시지 (Right Align):**
     - 구조: `[시간]` + `[말풍선]`
     - 말풍선 스타일: 배경 `#2563EB`, 글자색 `#FFFFFF`, 모서리 `16px 4px 16px 16px`, 패딩 `10px 14px`, 최대 너비 `65%`, 줄바꿈 `word-break: break-word`.
     - 타임스탬프: 말풍선 좌측 하단, 폰트 크기 `11px`, 색상 `#9CA3AF`.
4. **하단 입력창 영역 (`.chat-input-bar`):**
   - 배경: `#FFFFFF`, 상단 보더: `1px solid #E5E7EB`, 패딩: `16px 24px`, Flexbox 정렬.
   - 텍스트 입력 인풋 (`input[type="text"]`):
     - `flex: 1; height: 44px; padding: 0 16px; border: 1px solid #D1D5DB; border-radius: 8px; font-size: 14px;`
     - 포커스 시: `outline: none; border-color: #2563EB; box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.15);`
     - 플레이스홀더: `"메시지를 입력하세요... (Enter를 누르면 전송)"`
   - 전송 버튼 (`button.send-btn`):
     - 크기: 높이 `44px`, 최소 너비 `72px`, 패딩: `0 18px`, 배경 `#2563EB`, 글자색 `#FFFFFF`, `font-weight: 600`, 둥근 모서리 `8px`.
     - 호버 시: `#1D4ED8`.
   - **전송 인터랙션 동작:**
     - 전송 버튼 클릭 또는 인풋창에서 `Enter` 키 입력 시 발생 (단, `Shift+Enter` 예외 처리).
     - 빈 문자열(공백 제외)이면 전송 방지.
     - 동작 코드 구조:
       ```javascript
       // TODO: Supabase 연동 - 데이터베이스에 메시지 INSERT
       // await supabase.from('messages').insert([{ text, sender: currentUser.name, created_at: new Date() }]);
       ```
     - 인메모리 배열에 `{ sender: "게스트", text: value, isMe: true, time: "오후 1:20" }` 추가.
     - 채팅창 DOM에 즉시 렌더링.
     - 입력창 비우기 및 포커스 유지.
     - 채팅 컨테이너 스크롤을 맨 아래로 부드럽게 이동 (`scrollTop = scrollHeight`).

---

#### D. 메인 컨텐츠 영역 2: 게시판 (`#board-view`)
`flex: 1; display: none; height: calc(100vh - 64px); background: #F8F9FA; align-items: center; justify-content: center;`

- **컴포넌트 내용:**
  - 중앙 안내 카드: 배경 `#FFFFFF`, 패딩 `48px`, 테두리 `1px solid #E5E7EB`, 둥근 모서리 `16px`, 텍스트 중앙 정렬.
  - 아이콘: `📚` (폰트 크기 `48px`, 여백 `16px`).
  - 안내 문구: `"게시판 기능은 현재 준비 중입니다"` (`font-size: 20px; font-weight: 700; color: #1F2937; margin-bottom: 8px;`).
  - 설명: `"공지사항 및 게시글 작성 기능은 추후 업데이트될 예정입니다."` (`font-size: 14px; color: #6B7280;`).
  - '채팅방으로 이동' 바로가기 버튼 제공 (클릭 시 사내 채팅방 탭으로 자동 전환).

---

## 5. 자바스크립트 상태 관리 및 구조 명세

스크립트 태그 내에 다음과 같은 명확한 구조로 작성되어야 한다.

```javascript
/* ==========================================================================
   1. 상태 관리 (State Management)
   ========================================================================== */
let currentUser = null; // { name: '게스트', email: 'guest@company.com' }
let currentMenu = 'chat'; // 'chat' | 'board'

// 더미 메시지 데이터
let messages = [
  { id: 1, sender: '김철수 팀장', text: '안녕하세요! 이번 주 스프린트 배포 일정 안내드립니다.', isMe: false, time: '10:30 AM' },
  { id: 2, sender: '이영희', text: '네 팀장님, 프론트엔드 작업 오늘 중으로 마무리 예정입니다.', isMe: false, time: '10:32 AM' },
  { id: 3, sender: '게스트', text: '확인했습니다! 저도 채팅 UI 프로토타입 구현 완료했습니다.', isMe: true, time: '10:35 AM' }
];

/* ==========================================================================
   2. DOM 참조 (Elements)
   ========================================================================== */
// 로그인 화면, 메인 화면, 메뉴 탭, 헤더 제목, 채팅창, 입력창, 전송 버튼 등

/* ==========================================================================
   3. 인증 관련 함수 (Auth Handlers)
   ========================================================================== */
function handleLogin() {
  // TODO: Supabase 연동 - Google OAuth 로그인
  // const { data, error } = await supabase.auth.signInWithOAuth({ provider: 'google' });

  currentUser = { name: '게스트', email: 'guest@company.com' };
  renderAuthView();
}

function handleLogout() {
  // TODO: Supabase 연동 - 로그아웃
  // await supabase.auth.signOut();

  currentUser = null;
  renderAuthView();
}

/* ==========================================================================
   4. 뷰 렌더링 및 탭 전환 함수 (View Switching)
   ========================================================================== */
function switchMenu(menuName) {
  // 메뉴 active 상태 토글 및 컨텐츠 표시 전환 ('chat' -> #chat-view, 'board' -> #board-view)
}

/* ==========================================================================
   5. 채팅 기능 함수 (Chat Handlers)
   ========================================================================== */
function renderMessages() {
  // messages 배열을 순회하여 DOM 말풍선 생성 및 맨 아래 스크롤
}

function sendMessage() {
  const text = messageInput.value.trim();
  if (!text) return;

  // TODO: Supabase 연동 - 데이터베이스 insert 및 실시간 수신 대기
  // await supabase.from('messages').insert([{ sender: currentUser.name, text }]);

  const newMsg = {
    id: Date.now(),
    sender: currentUser ? currentUser.name : '게스트',
    text: text,
    isMe: true,
    time: formatCurrentTime()
  };

  messages.push(newMsg);
  renderMessages();
  messageInput.value = '';
  messageInput.focus();
}

/* ==========================================================================
   6. 이벤트 리스너 등록 (Initialization)
   ========================================================================== */
```

---

## 6. Supabase 연동 주석 가이드라인

향후 Supabase 도입 시 백엔드 개발자 또는 AI가 즉시 작업할 수 있도록 코드 내 필수 삽입 주석:

1. **클라이언트 초기화 영역:**
   ```javascript
   // TODO: Supabase 연동 - 클라이언트 라이브러리 로드 및 초기화
   // import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'
   // const supabase = createClient('YOUR_SUPABASE_URL', 'YOUR_SUPABASE_ANON_KEY')
   ```
2. **구글 로그인:**
   ```javascript
   // TODO: Supabase 연동 - Google OAuth 로그인
   // await supabase.auth.signInWithOAuth({ provider: 'google' })
   ```
3. **로그아웃:**
   ```javascript
   // TODO: Supabase 연동 - 세션 종료
   // await supabase.auth.signOut()
   ```
4. **실시간 메시지 구독 (Realtime Subscription):**
   ```javascript
   // TODO: Supabase 연동 - Realtime 채널 구독 (postgres_changes)
   // supabase.channel('chat-room').on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages' }, payload => { ... }).subscribe();
   ```
5. **메시지 전송 (DB Insert):**
   ```javascript
   // TODO: Supabase 연동 - messages 테이블에 메시지 저장
   // await supabase.from('messages').insert([{ sender: currentUser.name, text: text, created_at: new Date() }]);
   ```

---

## 7. 검수 기준 및 최종 체크리스트 (QA Checklist)

구현 후 아래 항목을 모두 만족해야 한다:

- [ ] 브라우저에서 `index.html` 파일을 직접 열었을 때 에러 없이 로그인 카드가 중앙에 뜬다.
- [ ] 사이드바 배경이 어두운 남색(`#1E2238`), 본문이 흰색(`#FFFFFF`), 버튼 및 내 말풍선이 파란색(`#2563EB`)이다.
- [ ] "Google 계정으로 로그인" 버튼을 누르면 즉시 메인 워크스페이스 화면으로 전환된다.
- [ ] 상단 헤더에 "게스트 님"과 "로그아웃" 버튼이 정상 표시된다.
- [ ] "로그아웃" 버튼을 누르면 다시 첫 로그인 카드로 돌아가며 상태가 리셋된다.
- [ ] 사이드바 "💬 사내 채팅방" 클릭 시 채팅 목록이 나타나고, "📚 게시판" 클릭 시 "준비 중입니다" 화면이 나타난다.
- [ ] 기본 더미 메시지(상대방 회색 말풍선 2개, 내 파란색 말풍선 1개)가 올바르게 렌더링된다.
- [ ] 입력창에 텍스트를 입력하고 '전송' 버튼 클릭 또는 `Enter` 키 입력 시 채팅창 맨 아래에 내 파란색 말풍선으로 즉시 추가된다.
- [ ] 메시지가 추가되면 채팅창 스크롤이 자동으로 최하단으로 이동한다.
- [ ] 코드 내에 지정된 `// TODO: Supabase 연동` 주석 5곳이 누락 없이 명시되어 있다.
