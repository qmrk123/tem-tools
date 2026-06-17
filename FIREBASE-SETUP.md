# Q&A 게시판 (Firebase) 설정 가이드

홈페이지의 **문의·피드백 게시판**은 [Firebase Firestore](https://console.firebase.google.com/)
무료 데이터베이스에 글을 저장합니다. 방문자는 GitHub·로그인 없이 이름+내용만 쓰면 됩니다.
아래 한 번만 설정하면 게시판이 켜집니다.

## 1. Firebase 프로젝트 만들기
1. https://console.firebase.google.com/ 접속 → **프로젝트 추가**
2. 프로젝트 이름(예: `tem-tools`) 입력 → 생성. (Google Analytics는 꺼도 됩니다.)

## 2. Firestore 데이터베이스 만들기
1. 좌측 메뉴 **빌드 → Firestore Database** → **데이터베이스 만들기**
2. 위치는 **asia-northeast3 (서울)** 권장
3. 시작 모드는 아무거나 선택(아래 3번에서 규칙을 덮어씁니다)

## 3. 보안 규칙 설정 (중요)
Firestore → **규칙(Rules)** 탭에 아래를 그대로 붙여넣고 **게시(Publish)**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /posts/{post} {
      allow read: if true;
      allow create: if request.resource.data.keys().hasOnly(['name','message','createdAt'])
                    && request.resource.data.message is string
                    && request.resource.data.message.size() > 0
                    && request.resource.data.message.size() < 2000
                    && request.resource.data.name is string
                    && request.resource.data.name.size() < 60;
      allow update, delete: if false;   // 글 수정/삭제는 콘솔에서 관리자만
    }
  }
}
```

이 규칙은 **누구나 읽기 + 글쓰기**는 되지만(길이 제한 포함), **남의 글 수정·삭제는 차단**합니다.

## 4. 웹 앱 등록 후 config 복사
1. 프로젝트 개요 옆 **⚙️ → 프로젝트 설정** → 하단 **내 앱**에서 **웹(`</>`)** 아이콘 클릭
2. 앱 닉네임 입력 → 등록 (호스팅은 체크 안 해도 됨)
3. 표시되는 `firebaseConfig = { apiKey: "...", ... }` 값을 복사
4. `index.html` 안의 `const firebaseConfig = { ... }` 부분(자리표시자 `PASTE_...`)을 복사한 값으로 교체하고 커밋·푸시
   - 이 값들은 **공개되어도 안전**합니다(보안은 위 규칙이 담당).

→ 푸시 후 페이지를 새로고침하면 게시판이 활성화됩니다.

## 글에 답변 달기 / 스팸 삭제 (관리자)
- **답변**: Firestore 콘솔 → `posts` 컬렉션 → 해당 글 문서 열기 → 필드 추가
  `reply` (string) 에 답변 내용 입력 → 저장하면 페이지에 "↳ 운영자 답변"으로 표시됩니다.
- **삭제**: 콘솔에서 해당 문서를 삭제하면 됩니다. (웹에서는 누구도 삭제 못 함)

## 무료 한도
Firestore 무료(Spark) 요금제는 하루 읽기 5만 / 쓰기 2만 건으로, 연구실 내부 게시판에는
차고 넘칩니다.
