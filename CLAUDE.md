# MIKS&T 홈페이지 — Development Guide

## 프로젝트 개요
한국 기업의 미국 시장 진출 컨설팅 회사 MIKS&T, INC 홈페이지

## 기술 스택
- **순수 HTML / CSS / JavaScript** (프레임워크 없음)
- **배포**: Netlify (GitHub 연동 자동 배포)
- **도메인**: `www.miksnt.com`
- **GitHub**: `johnnie010320-cmd/miksnt` (master 브랜치)

## 개발 워크플로우
1. 코드 변경 (`index.html`, 이미지 등)
2. 브라우저에서 로컬 확인
3. 커밋 & 푸시: `git add` → `git commit` → `git push origin master`
4. Netlify 자동 배포 (push 시 자동)

> **별도 빌드 명령어 없음** — push 하면 Netlify가 자동 배포

## 프로젝트 구조
```
mikst-website/
├── index.html          # 메인 (단일 페이지)
├── netlify.toml        # Netlify 빌드 설정
├── CNAME               # 도메인 설정 (www.miksnt.com)
├── _headers            # HTTP 헤더 설정 (캐싱, 보안)
├── *.jpg / *.webp      # 이미지 (WebP + JPG 쌍)
├── *.png               # 로고, 배경 이미지
└── README.md
```

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
