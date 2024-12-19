# TickerDisplay

암호화폐 티커 정보를 실시간으로 표시하는 Svelte 프로젝트입니다. 이 프로젝트는 SvelteKit을 사용하여 개발되었으며, 다양한 기능을 포함하고 있습니다.

UPBit API를 사용하여 암호화폐 가격 정보를 가져옵니다.

## 주요 기능

- [x] URL에서 티커 파라미터 추출 (기본값: KRW-BTC)
- [x] 암호화폐 아이콘 로드
- [x] 실시간 가격 정보 로드
- [x] 화면 burn-in 방지
- [x] Wake Lock 활성화 및 해제
- [x] 화면 밝기 토글
- [x] 전체 화면 진입 시 커서 숨기기
- [x] 폰트 크기 자동 조절
- [ ] 주식 티커 처리 추가
- [ ] 통화와 티커 따로 처리 (예: USD-BTC 지원등...)
- [ ] 없는 티커 조합 가능하면 처리

## 개발

프로젝트를 생성하고 `npm install` (또는 `pnpm install` 또는 `yarn`)로 의존성을 설치한 후, 개발 서버를 시작합니다:

```bash
npm run dev
```

## 빌드

앱의 프로덕션 버전을 생성하려면:

```bash
npm run build
```

프로덕션 빌드를 미리 보려면 `npm run preview`를 실행합니다.

> 앱을 배포하려면 대상 환경에 맞는 [어댑터](https://svelte.dev/docs/kit/adapters)를 설치해야 할 수 있습니다.
