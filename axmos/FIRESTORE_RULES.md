# Firestore 보안 규칙 — MIKS&T 홈페이지 CMS

admin CMS(AXMOS / Partners / Achievements)는 `siteContent` 컬렉션을 사용합니다.
아래 규칙을 **Firebase Console → Firestore Database → 규칙(Rules)** 에 반영하세요.

- 프로젝트: `miks-def1e`
- 콘솔: https://console.firebase.google.com/project/miks-def1e/firestore/rules

## 데이터 구조

```
siteContent (collection)
 ├─ nav            { items: [ {labelKo, labelEn, target, visible}, ... ], updatedAt }   # 상단 메뉴(순서=배열순서)
 ├─ hero           { heroImage, taglineKo, taglineEn, updatedAt }                        # 메인 배경/문구
 ├─ about          { subtitleKo, subtitleEn, bodyKo, bodyEn, updatedAt }                 # 회사소개(문단=빈 줄 구분)
 ├─ services       { items: [ {image, titleKo, titleEn, descKo, descEn, visible}, ... ] }
 ├─ axmos          { items: [ {name, roleKo, roleEn, descKo, descEn, visible}, ... ] }
 ├─ partners       { items: [ {name, url, logoUrl, visible}, ... ] }
 ├─ achievements   { items: [ {titleKo, titleEn, descKo, descEn, visible}, ... ] }
 └─ downloads      { items: [ {titleKo, titleEn, descKo, descEn, file, visible}, ... ] } # file=업로드 URL 또는 외부 URL
```

- 리스트 항목의 `visible: false` → 사이트에서 숨김 (admin 체크박스로 토글)
- `nav`는 문서가 없어도 사이트가 기본 메뉴(About/Services/AXMOS/Media/Downloads/Contact 노출)를 자동 표시
- 이미지/파일은 **Firebase Storage 업로드**(admin의 파일 선택) 또는 **URL 직접 입력** 둘 다 가능

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

## Storage 보안 규칙 (파일/이미지 업로드용)

admin이 이미지·자료 파일을 업로드하면 **Firebase Storage**의 `uploads/` 경로에 저장됩니다.
**Firebase Console → Storage → Rules** 에 아래를 반영하세요.

- 콘솔: https://console.firebase.google.com/project/miks-def1e/storage/miks-def1e.firebasestorage.app/rules

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /uploads/{allPaths=**} {
      allow read: if true;                       // 누구나 다운로드/이미지 표시
      allow write: if request.auth != null
        && firestore.get(/databases/(default)/documents/users/$(request.auth.uid)).data.role == 'admin';
    }
  }
}
```

> ⚠️ **Spark(무료) 요금제 주의**: 최근 Firebase는 Storage 사용 시 **Blaze(종량제) 업그레이드**를 요구할 수 있습니다.
> - Storage가 활성화돼 있으면 → 위 규칙 반영 후 admin에서 **파일 선택 업로드**가 동작합니다.
> - 업그레이드가 부담되거나 Storage 미활성 상태면 → 업로드 대신 **URL 직접 입력**란을 쓰면 됩니다.
>   (Google Drive/Dropbox 공유링크, 기존 사이트 이미지 파일명 등). 이 경우 Storage 설정은 불필요합니다.

## 검증 방법

- admin 로그인 후 CMS에서 항목 저장 → 새로고침 시 해당 섹션이 저장 내용으로 표시되면 성공
- 비로그인 상태에서 저장 시도 시 콘솔에 `permission-denied` 가 떠야 정상(쓰기 차단 확인)
- 메뉴(nav) 저장 후 상단 메뉴가 바뀌면 성공. 파일 업로드가 막히면 URL 직접 입력으로 대체
