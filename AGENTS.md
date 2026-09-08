# Landing Hub: 하위 경로 기반 다중 랜딩페이지 생성 및 운영 규칙 (AGENTS.md)

본 문서는 에이전트 및 개발자가 신규 광고주/업체/캠페인의 랜딩페이지를 구축할 때 준수해야 하는 표준 가이드라인입니다.

---

## 1. 디렉토리 및 파일 구조 규칙
- **하위 경로 단일 파일 원칙**: 신규 업체/캠페인 요청 시 반드시 `/{folder_name}/index.html` 단일 파일 형태로 생성합니다.
  - 예시:
    - `/lawfirm-a/index.html`
    - `/dental-gangnam/index.html`
    - `/interior-seoul/index.html`
- **루트 파일 보호**: 루트 디렉토리의 `index.html` 및 `AGENTS.md`는 허브 기본 인프라이므로 개별 랜딩페이지 작업 시 임의로 수정하거나 삭제하지 않습니다.

---

## 2. 완전한 인라인 격리 (Style & Script Isolation)
- **외부 CSS/JS 종속성 최소화**: 각 랜딩페이지 파일(`/{folder_name}/index.html`) 내부에 `<style>` 및 `<script>` 태그를 사용하여 모든 디자인과 기능을 완전히 인라인으로 작성합니다.
- **전역 오염 방지**: 서로 다른 하위 경로 간 클래스명 충돌이나 전역 스크립트 충돌이 발생하지 않도록 철저히 독립된 컨텍스트를 유지합니다.
- **웹 폰트 및 공통 에셋**: Google Fonts, Pretendard CDN 등 표준 CDN 외에 로컬 공통 CSS를 공유하지 않고 독립적으로 로드합니다.

---

## 3. 트래킹 코드 독립 삽입 규칙 (Tracking & Analytics)
- **독립 삽입 원칙**: 각 업체에서 요청한 트래킹 스크립트만 해당 랜딩페이지의 `<head>` 또는 `<body>` 끝부분에 단독 삽입합니다. 타 업체의 픽셀이나 전역 코드가 섞이지 않도록 주의합니다.
- **지원 표준 트래킹**:
  1. **Meta Pixel (Facebook/Instagram)**:
     ```html
     <!-- Meta Pixel Code -->
     <script>
       !function(f,b,e,v,n,t,s)
       {if(f.fbq)return;n=f.fbq=function(){n.callMethod?
       n.callMethod.apply(n,arguments):n.queue.push(arguments)};
       if(!f._fbq)f._fbq=n;n.push=n;n.loaded=!0;n.version='2.0';
       n.queue=[];t=b.createElement(e);t.async=!0;
       t.src=v;s=b.getElementsByTagName(e)[0];
       s.parentNode.insertBefore(t,s)}(window, document,'script',
       'https://connect.facebook.net/en_US/fbevents.js');
       fbq('init', '{META_PIXEL_ID}');
       fbq('track', 'PageView');
     </script>
     ```
  2. **Google Analytics 4 (GA4)**:
     ```html
     <!-- Google tag (gtag.js) -->
     <script async src="https://www.googletagmanager.com/gtag/js?id={GA4_MEASUREMENT_ID}"></script>
     <script>
       window.dataLayer = window.dataLayer || [];
       function gtag(){dataLayer.push(arguments);}
       gtag('js', new Date());
       gtag('config', '{GA4_MEASUREMENT_ID}');
     </script>
     ```
  3. **Naver Search Advisor / Naver Premium Log**:
     - 요청된 네이버 프리미엄 로그분석 스크립트 또는 사이트 인증 메타태그 삽입.

---

## 4. CTA 버튼 전환 이벤트(Lead Conversion) 삽입 규칙
- 사용자가 문의하기, 상담 신청, 전화 연결 등의 핵심 행동을 취하는 모든 CTA(Call To Action) 요소에는 `onclick` 또는 이벤트 리스너를 통해 표준 전환 이벤트를 트리거합니다.
- **표준 전환 핸들러 패턴 예시**:
  ```html
  <button type="submit" class="cta-btn" onclick="handleConversion(event)">
    상담 신청 완료하기
  </button>

  <script>
    function handleConversion(e) {
      // 1. Meta Pixel 전환 추적 (Lead)
      if (typeof fbq === 'function') {
        fbq('track', 'Lead', {
          content_name: '{CAMPAIGN_NAME}',
          status: 'submitted'
        });
      }

      // 2. Google Analytics 4 전환 추적 (generate_lead)
      if (typeof gtag === 'function') {
        gtag('event', 'generate_lead', {
          event_category: 'form',
          event_label: '{CAMPAIGN_NAME}'
        });
      }

      // 3. 네이버 전환 스크립트 추적 (설정된 경우)
      if (typeof wcs_do === 'function') {
        // wcs 전환 처리 로직
      }
    }
  </script>
  ```

---

## 5. 성능 및 UX 가이드라인
- 모바일 퍼스트 반응형 레이아웃 필수 (Viewport 메타태그 기본 포함).
- 모든 폼 입력 필드는 유효성 검사(Validation)를 거치도록 구현.
- 로딩 속도 최적화 (무거운 외부 라이브러리 지양, 순수 바닐라 JS 및 CSS 우선 사용).
