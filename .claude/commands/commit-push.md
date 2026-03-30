# 커밋 & 푸시

변경사항을 커밋하고 master에 푸시합니다. (Netlify 자동 배포)

## 실행 순서

1. `git status` — 변경된 파일 확인
2. `git diff` — 변경 내용 확인
3. `git add <파일>` — 변경 파일 스테이징
4. `git commit -m "<메시지>"`
5. `git push origin master`

Netlify가 push를 감지하여 자동으로 www.miksnt.com에 배포합니다.

## 커밋 메시지 예시
- `섹션명 문구 수정`
- `이미지 추가 - WebP 변환 포함`
- `네비게이션 링크 수정`
