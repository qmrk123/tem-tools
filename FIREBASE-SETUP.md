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
    function isAdmin() {
      return request.auth != null && request.auth.token.email == 'qmrk123@hanyang.ac.kr';
    }
    match /posts/{post} {
      allow read: if true;
      allow create: if request.resource.data.keys().hasOnly(['name','message','createdAt','parentId','admin'])
                    && request.resource.data.name is string
                    && request.resource.data.name.size() < 60
                    && request.resource.data.message is string
                    && request.resource.data.message.size() > 0
                    && request.resource.data.message.size() < 2000
                    && (!('parentId' in request.resource.data.keys())
                        || request.resource.data.parentId is string)
                    && (!('admin' in request.resource.data.keys())
                        || (request.resource.data.admin == true && isAdmin()));
      allow update: if false;
      allow delete: if isAdmin();          // 삭제는 로그인한 관리자만
    }
  }
}
```

이 규칙은 **누구나 읽기·글쓰기·답글**은 되지만(길이 제한 포함), **글 삭제와 "운영자 답변" 표시는
로그인한 관리자(아래 5단계)만** 가능합니다. `isAdmin()`의 이메일은 5단계에서 만들 관리자 계정과
같아야 합니다(기본값 `qmrk123@hanyang.ac.kr`).

## 4. 웹 앱 등록 후 config 복사
1. 프로젝트 개요 옆 **⚙️ → 프로젝트 설정** → 하단 **내 앱**에서 **웹(`</>`)** 아이콘 클릭
2. 앱 닉네임 입력 → 등록 (호스팅은 체크 안 해도 됨)
3. 표시되는 `firebaseConfig = { apiKey: "...", ... }` 값을 복사
4. `index.html` 안의 `const firebaseConfig = { ... }` 부분(자리표시자 `PASTE_...`)을 복사한 값으로 교체하고 커밋·푸시
   - 이 값들은 **공개되어도 안전**합니다(보안은 위 규칙이 담당).

→ 푸시 후 페이지를 새로고침하면 게시판이 활성화됩니다.

## 5. 관리자 로그인 만들기 (Authentication)
페이지의 **관리자 버튼**으로 글 삭제·운영자 답변을 하려면, 관리자 암호를 Firebase 로그인
비밀번호로 등록합니다(이렇게 해야 삭제 권한이 서버에서 실제로 보호됩니다).

1. 좌측 메뉴 **빌드 → Authentication → 시작하기**
2. **Sign-in method** 탭 → **이메일/비밀번호** 사용 설정(Enable) → 저장
3. **Users** 탭 → **사용자 추가**
   - 이메일: `qmrk123@hanyang.ac.kr` (위 규칙·`index.html`의 `ADMIN_EMAIL`과 동일해야 함)
   - 비밀번호: 관리자 암호 입력
4. 끝! 이제 홈페이지에서 **관리자 버튼 → 암호 입력**으로 로그인하면 각 글에 🗑(삭제)와
   "운영자 답변"으로 등록할 수 있습니다.

## 글 관리 (관리자)
- **운영자 답변**: 관리자로 로그인 → 해당 글의 **답글** 버튼 → 입력하면 "↳ 운영자 답변"(초록)으로 표시.
- **삭제**: 관리자 모드에서 각 글/답글의 🗑 클릭. (질문을 지우면 그 답글도 함께 삭제됩니다.)
- 웹에서의 삭제는 **로그인한 관리자만** 가능하며, 일반 방문자는 불가합니다. 콘솔에서 직접
  삭제·수정하는 것도 언제든 가능합니다.

## 무료 한도
Firestore 무료(Spark) 요금제는 하루 읽기 5만 / 쓰기 2만 건으로, 연구실 내부 게시판에는
차고 넘칩니다.
