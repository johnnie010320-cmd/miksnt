# MIKS&T 홈페이지 — Development Guide

## 프로젝트 개요
MIKS&T, INC — **AI·클라우드 솔루션 전문 기업** 홈페이지.
루트 `www.miksnt.com`은 AI+Cloud 기업 이미지를 전면에 내세우고, 그 결과물로 두 제품을 소개한다:
- **CloudBridge**(`cloudbridge.miksnt.com`) — 검증 가능한 클라우드 영상 인프라 (SHA-256 해시체인)
- **VeriDash**(`veridash.miksnt.com`) — 무결성 블랙박스 앱 (iOS/Android)

제조 AI 전환(AX) 대행 사업(구 메인 콘텐츠)은 `/axmos/` 폴더로 분리 → **`axmos.miksnt.com`** 별도 배포.
공통 기술 = 무결성 체인(Integrity Chain).

## 기술 스택
- **순수 HTML / CSS / JavaScript** (프레임워크 없음)
- **배포**: Netlify (GitHub 연동 자동 배포)
- **도메인**: `www.miksnt.com`
- **GitHub**: `johnnie010320-cmd/miksnt` (main 브랜치)

## 개발 워크플로우
1. 코드 변경 (`index.html`, 이미지 등)
2. 브라우저에서 로컬 확인
3. 커밋 & 푸시: `git add` → `git commit` → `git push origin main`
4. Netlify 자동 배포 (push 시 자동)

> **별도 빌드 명령어 없음** — push 하면 Netlify가 자동 배포

## 프로젝트 구조
```
mikst-website/
├── index.html          # www.miksnt.com — AI+Cloud 기업 (배너 캐러셀 + CloudBridge/VeriDash 상세). Firebase 미사용
├── banner-ai.svg / banner-cloud.svg / banner-cloudbridge.svg / banner-veridash.svg  # 배너 슬라이드 아트워크(벡터)
├── axmos.html          # AXMOS 상세페이지 (이중언어, 20장 브로슈어 기반) — /axmos/에도 복제본
├── axmos/              # ★ axmos.miksnt.com 별도 사이트 (구 메인 콘텐츠 자립형 복사본)
│   ├── index.html      #   구 AXMOS 주력 홈페이지 (그대로 보존)
│   ├── axmos.html, *.webp/jpg/png, _headers
│   └── CNAME           #   axmos.miksnt.com
├── FIRESTORE_RULES.md  # CMS용 Firestore 보안규칙 + 관리자 지정 안내
├── netlify.toml        # Netlify 빌드 설정
├── CNAME               # 도메인 설정 (www.miksnt.com)
├── _headers            # HTTP 헤더 설정 (캐싱, 보안)
├── *.jpg / *.webp      # 이미지 (WebP + JPG 쌍)
├── *.png               # 로고, 배경 이미지
└── README.md
```

> **⚠️ CF 캐시 함정(신규 자산)**: www는 Cloudflare(miksnt.com zone)→Netlify. **배포 완료 전에 신규 파일 URL을 요청하면 CF가 404를 4시간(max-age=14400) 캐싱**해 방문자에게도 깨져 보인다. 신규 이미지/자산은 **참조에 `?v=N` 쿼리를 붙여** 새 캐시키로 내보낼 것(예: `miks-logo.png?v=2`, `banner-axmos.svg?v=1`). 검증도 HTML 반영 확인 후에 자산 URL을 찌를 것(바레 URL 사전 요청 금지).

> **배너 수정**: `index.html` 안의 `const BANNER={...}` slides 배열만 편집 후 `git push`. 런타임 CMS/로그인/콘솔 없음(정적 마케팅 사이트라 의도적으로 단순화, 2026-09-26 죠니 확정). Firestore CMS(`FIRESTORE_RULES.md`)는 이제 `/axmos` 사이트에만 해당.

> **배포 구조**: 같은 repo, 두 Netlify 사이트. ① 기존 사이트=루트 → www.miksnt.com. ② 신규 사이트=base/publish `axmos/` → axmos.miksnt.com (DNS: axmos CNAME → Netlify).

## 이미지 관리 규칙
- 모든 이미지는 **WebP + JPG 쌍**으로 유지
- `<picture>` 태그로 WebP 우선 제공, JPG fallback
- 새 이미지 추가 시 WebP 변환 필수 (약 52% 용량 절감)
- 이미지에 `loading="lazy"` 적용

## 디자인 시스템 (CSS 변수)
```css
--navy: #1A237E        /* 주색 */
--navy-light: #3949AB  /* 보조 네이비 */
--gold: #C9A84C        /* 골드 강조 */
--gold-light: #E8D48B  /* 연한 골드 */
--dark: #0D1117        /* 배경 */
```

## 폰트
- **Montserrat** (영문 제목)
- **Noto Sans KR** (한국어)
- Google Fonts CDN 사용

## 성능 최적화 (기존 적용)
- WebP 이미지 + lazy loading
- `preconnect` 힌트 (fonts, YouTube, TradingView)
- Netlify 자동 CSS/JS 번들링 & 압축

## 금지 사항
- JPG 없이 WebP만 추가하지 않기 (fallback 필요)
- `netlify.toml` 임의 수정 금지
- `CNAME` 파일 삭제 금지
