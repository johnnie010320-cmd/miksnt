# Firestore 보안 규칙 — MIKS&T 홈페이지 CMS

admin CMS(AXMOS / Partners / Achievements)는 `siteContent` 컬렉션을 사용합니다.
아래 규칙을 **Firebase Console → Firestore Database → 규칙(Rules)** 에 반영하세요.

- 프로젝트: `miks-def1e`
- 콘솔: https://console.firebase.google.com/project/miks-def1e/firestore/rules

## 데이터 구조

```
siteContent (collection)
 ├─ axmos          { items: [ {name, roleKo, roleEn, descKo, descEn}, ... ], updatedAt }
 ├─ partners       { items: [ {name, url, logoUrl}, ... ], updatedAt }
 └─ achievements   { items: [ {titleKo, titleEn, descKo, descEn}, ... ], updatedAt }
```

- 문서가 없거나 `items`가 비어 있으면 사이트는 **HTML 기본값(fallback)** 을 그대로 표시합니다.
- 읽기는 **누구나(public)** — 방문자 모두가 콘텐츠를 봐야 하므로.
- 쓰기는 **role=admin 로그인 사용자만**.

## 규칙 (전체)

> ⚠️ 아래는 이 앱의 동작(유저 관리 + siteContent)에 맞춘 **완성본**입니다.
> 콘솔에 이미 다른 규칙이 있다면 통째로 덮어쓰지 말고 `siteContent` 블록만 병합하세요.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    function isAdmin() {
      return request.auth != null
        && exists(/databases/$(database)/documents/users/$(request.auth.uid))
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }

    // 사이트 콘텐츠(CMS): 공개 읽기, 관리자만 쓰기
    match /siteContent/{docId} {
      allow read: if true;
      allow write: if isAdmin();
    }

    // 사용자: 본인 문서는 본인이, 목록/관리는 관리자
    match /users/{userId} {
      allow read: if request.auth != null && (request.auth.uid == userId || isAdmin());
      allow create: if request.auth != null && request.auth.uid == userId;
      allow update: if request.auth != null && request.auth.uid == userId;
      allow delete: if isAdmin();
      // 관리자 패널의 사용자 '목록' 조회를 허용하려면 컬렉션 list 권한 필요:
      allow list: if isAdmin();
    }
  }
}
```

## 최초 관리자 지정

`users/{uid}` 문서의 `role` 필드를 `admin` 으로 직접 설정해야 admin 버튼/CMS가 활성화됩니다.
(회원가입 시 기본값은 `role: 'user'` — index.html 참조)

1. 웹에서 관리자용 이메일로 회원가입 + 이메일 인증 완료
2. Firebase Console → Firestore → `users` → 해당 uid 문서 → `role` 값을 `admin` 으로 변경
3. 다시 로그인하면 상단에 **Admin** 버튼 표시 → AXMOS / Partners / Achievements 탭에서 편집·저장

## 검증 방법

- admin 로그인 후 CMS에서 항목 저장 → 새로고침 시 해당 섹션이 저장 내용으로 표시되면 성공
- 비로그인 상태에서 저장 시도 시 콘솔에 `permission-denied` 가 떠야 정상(쓰기 차단 확인)
