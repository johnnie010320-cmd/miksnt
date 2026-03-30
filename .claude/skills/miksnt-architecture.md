---
name: miksnt-architecture
description: MIKS&T 홈페이지 구조, 디자인 시스템, 이미지 규칙. Use when working with the homepage layout, styling, or images.
---

# MIKS&T 홈페이지 Architecture

## 구조
- **단일 파일**: `index.html` (전체 페이지 구성)
- **외부 CSS 없음**: 모든 스타일 `<style>` 태그 내 인라인
- **외부 JS 없음**: 모든 스크립트 `<script>` 태그 내 인라인

## 페이지 섹션 구성
1. **Hero** - 메인 배경 + 핵심 메시지
2. **Services** - Sales, Logistics, Investment, Facility 4개 서비스
3. **Process** - 진행 절차 단계별 안내
4. **Partners** - 파트너사 (Translink 등)
5. **SNS/YouTube** - 영상 갤러리 (썸네일 리스트)
6. **Contact / Footer**

## 디자인 시스템

### CSS 변수
```css
:root {
  --navy: #1A237E;       /* 주색 */
  --navy-light: #3949AB; /* 보조 */
  --gold: #C9A84C;       /* 강조 */
  --gold-light: #E8D48B; /* 연한 강조 */
  --dark: #0D1117;       /* 배경 */
  --gray-900: #1a1a2e;
  --gray-700: #4a4a5a;
  --gray-500: #8892a4;
  --gray-300: #c8cdd5;
  --gray-100: #f0f2f5;
  --white: #ffffff;
}
```

### 폰트
- `Montserrat` — 영문 제목, 숫자
- `Noto Sans KR` — 한국어 텍스트

## 이미지 패턴
```html
<!-- 항상 <picture>로 WebP 우선 제공 -->
<picture>
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="설명" loading="lazy">
</picture>
```

현재 이미지 목록:
- `hero-background`, `header-background` — 배경
- `consulting-card`, `investment-card`, `sales-marketing-card` — 서비스 카드
- `facility-step`, `logistics-step`, `sales-marketing-step` — 프로세스 단계
- `facility`, `logistics`, `sales-marketing` — 섹션 이미지
- `market-analysis`, `client-meeting` — 기타
- `partner-translink` — 파트너
- `logo` — 로고

## 배포
- Netlify 자동 배포 (master push 시)
- `netlify.toml`: CSS/JS/HTML 최소화, 이미지 압축 자동
- `_headers`: 캐싱 + 보안 헤더
