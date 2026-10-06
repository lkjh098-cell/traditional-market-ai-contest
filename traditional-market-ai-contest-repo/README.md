# 마포한끼 — 지도에서 식탁까지

마포농수산물시장의 상인 팀 공동 메뉴를 가족 조건에 맞춰 AI로 구성하고, 실제 시장 위성지도 위 공동 픽업대에서 한 번에 받는 모바일 웹 시제품입니다.

**점포 배치·원안·가격·재고는 가상 시연 데이터입니다. 실제 결제·식품 준비·송금·배송은 발생하지 않습니다.** 시장 지도는 실제 Google 위성지도(서울 마포구 월드컵로 235)이며, 그 위의 점포 위치만 가상 배치입니다.

## 메뉴 순서와 체험 흐름

상단·모바일 하단 메뉴는 `lib/navigation.ts` 한 곳에서 정의하며, 첫 화면은 **AI 한 끼**입니다.

1. **AI 한 끼:** 인원·끼니·상황·맛·예산·시간·도구·보유 재료/양·추가·제외·대체·기타 총 12개 입력을 읽고 상인 원안과 변경 구성을 비교합니다.
2. **공동 메뉴 9종:** 채소·수산·정육 메뉴와 각 메뉴를 만든 **상인 팀**, 참여 점포를 확인합니다. 메뉴별 음식 사진은 각각 생성한 AI 예시입니다.
3. **시장 지도:** 마포농수산물시장의 **실제 Google 위성지도** 위, 시장 건물 한가운데에 **공동 픽업대**가 있고 6개 가상 점포가 둘러쌉니다. 점포를 누르면 공급 재료·공동 메뉴(팀 표시)·점포 QR이 열리고, 선택한 점포에서 픽업대까지 경로가 표시됩니다. 픽업대를 누르면 주문·픽업으로 이동합니다.
4. **주문·픽업:** 장바구니의 구매 재료·준비비·추가 재료 금액을 확인해 시장 중심 공동 픽업대로 시연 예약합니다. 상인 화면에서 준비 시작→점포 준비 완료를 누르고, 모든 참여 점포가 완료하면 수령을 확인합니다. 수령 후 점포별 밀키트 재료·준비비·현장 추가 금액을 확인해 시연 정산을 확정합니다.
5. **추가 장보기:** 현장에서 실물을 보고 재료를 더 담으면 선택한 주문과 공급 점포 매출에 반영됩니다. 수령 전 추가 품목은 점포 준비를 다시 요구합니다.

QR의 `?shop=`(시장 지도), `?menu=`(AI 한 끼), `?ingredient=`(추가 장보기), `?order=`(주문·픽업)가 각 화면으로 바로 연결됩니다. `source`로 최초 안내 점포를 기록하여 점포 간 구매 연결을 추적합니다. 주문 링크는 같은 브라우저 체험 데이터 또는 같은 서버 저장소에서 열립니다.

## 시장 위성지도

- **좌표:** 공동 픽업대 = 시장 건물 중심 `37.56519, 126.8985`(OpenStreetMap 건물 way 38170384 중심점). Waze·Tripadvisor 표기 좌표와 30m 이내로 일치하는지 테스트로 확인합니다.
- **가상 점포 배치:** 픽업대 기준 북·동 방향 미터 값(`lib/market-geo.ts`의 `DEFAULT_SHOP_GEO`, 또는 점포별 `geo`)으로 정하고 Web Mercator 투영으로 지도 위에 올립니다. 실제 점포 위치가 아닙니다.
- **기본(키 없음):** Google 지도 위성 임베드(`maps.google.com/maps?…&t=k&output=embed`)를 씁니다. API 키 없이 실제 Google 위성 영상이 열리고, 같은 투영으로 계산한 점포·픽업대 마커를 겹칩니다. `지도 직접 움직이기`를 누르면 마커를 숨기고 Google 지도를 자유롭게 확대·이동할 수 있습니다. GitHub Pages 체험판은 이 방식입니다.
- **선택(브라우저 키):** `VITE_GOOGLE_MAPS_API_KEY`가 있으면 Maps JavaScript API(`mapTypeId: 'satellite'`)로 전환하여 마커가 좌표에 고정된 채 확대·이동됩니다. 키 인증이나 스크립트 로딩이 실패하면 자동으로 키 없는 임베드로 돌아갑니다.
- 화면 폭 560px 미만은 확대 18, 그 이상은 19를 씁니다. 길찾기는 Google·네이버·카카오 지도 링크를 제공합니다.

### Google Maps 브라우저 키(선택)

Maps 키는 **브라우저에 공개되는 키**입니다. 서버 비밀값인 Gemini 키와 섞지 마세요.

1. Google Cloud 콘솔에서 Maps JavaScript API만 사용 설정하고 API 키를 만듭니다.
2. 키 제한: **애플리케이션 제한 = 웹사이트(HTTP 리퍼러)** — 예: `https://lkjh098-cell.github.io/traditional-market-ai-contest/*`, `http://127.0.0.1:5173/*`. **API 제한 = Maps JavaScript API**.
3. `.env.local`에 `VITE_GOOGLE_MAPS_API_KEY=…`를 적고 `npm run build:pages`로 빌드합니다.

`vite.pages.config.ts`는 `VITE_GOOGLE_MAPS_` 접두사 변수만 번들에 넣습니다. 키를 넣어 빌드한 `docs/`는 키 문자열을 포함하므로, 리퍼러 제한을 건 키일 때만 커밋하세요. 키 없이 빌드하면 키 없는 위성 임베드로 작동합니다.

## 공동 메뉴와 팀

| 메뉴 | 팀 |
| --- | --- |
| 버섯 가득 두부전골 | 초록한상 |
| 온가족 맑은 해물전골 | 바다채소 |
| 새우버섯 한 팬 볶음 | 바다초록 |
| 소고기 채소 샤브전골 | 든든초록 |
| 무를 곁들인 생선조림 | 바다한술 |
| 오징어 채소볶음 | 바다채소 |
| 닭고기 버섯볶음 | 든든버섯 |
| 바지락 채소 맑은탕 | 바다맑음 |
| 구운 채소 두부 한상 | 초록두부 |

참여 점포는 초록채소·고소한두부·한술양념·바다한상·든든정육·오늘과일입니다. 과일점은 현장 추가 장보기로 연결됩니다. 준비비는 원안에 지정된 준비 담당 점포에 배분하며 전체 점포 합계가 고객 주문 금액과 일치해야 합니다.

## Gemini와 서버의 역할

서버의 실제 AI는 **Gemini 3.1 Flash-Lite** 네이티브 `generateContent` API를 사용합니다. `GEMINI_MODEL`로 모델을 변경할 수 있습니다. 등록된 원안·재료·대체 규칙으로 요청마다 JSON Schema를 생성하고 `responseMimeType: application/json` 및 `responseJsonSchema`를 지정합니다.

Gemini는 고객 요청을 `intent` JSON으로 해석합니다. 인원·예산·시간·도구·맛과 재료 변경에는 `evidence`(입력 원문 인용)와 `reason`(해석 이유)이 붙습니다. 서버가 인용의 존재와 허용된 변경 범위를 검사하고, 분량·가격·재고·주문 허용을 계산합니다. **인용 검사는 의미 해석의 완전한 정확성을 보증하지 않습니다.**

미해결 요청은 `not_registered`, `no_rule`, `health_claim`, `ambiguous`로 분류합니다. Gemini 안전 차단·출력 잘림·일일/결제 한도·일시 요청 한도·형식 오류를 구분하며 실패를 성공으로 표시하지 않습니다. 원안 직접 선택은 입력 조건이 적용되지 않는다는 확인을 거칩니다. 알레르기·건강 요청은 원안 구매로 우회하지 못합니다.

주문 시 저장된 원안 버전과 구성을 다시 검증합니다. 중복 예약·품절·변경된 가격을 차단하고 브라우저 가격을 신뢰하지 않습니다. 점포 준비·수령·추가 구매·정산은 `lib/order-workflow.ts`의 동일 계산 함수를 공개 체험판과 서버에서 사용합니다.

## 공개 체험판과 실제 AI

[GitHub Pages 체험판](https://lkjh098-cell.github.io/traditional-market-ai-contest/)은 **외부 AI 호출 없이 규칙으로 작동**합니다. 모든 기능은 해당 브라우저 저장소에 저장됩니다. 다른 기기에서 스캔하면 점포·공동 메뉴 안내를 볼 수 있지만 주문 데이터는 공유되지 않습니다.

Gemini 키는 서버의 `GEMINI_API_KEY` 또는 `GOOGLE_API_KEY`로만 읽습니다. 브라우저·GitHub·QR에 Gemini 키를 넣지 않습니다. (지도용 `VITE_GOOGLE_MAPS_API_KEY`는 별개의 공개 브라우저 키이며 위 리퍼러 제한이 필수입니다.) 2026-10-05 개발 시점에는 Gemini 키가 없어 실제 모델 응답 성공은 아직 검증하지 못했습니다. Gemini 응답 모의 검증과 주문 흐름 검증은 완료했습니다.

## 실행

Node.js24 이상을 권장합니다. GitHub에는 전체 소스와 `docs/` 체험판을 함께 보관합니다.

```sh
npm install
cp .env.example .env.local
# 실제 Gemini 사용 시 .env.local에 GEMINI_API_KEY를 로컬에서 설정
# (선택) Maps JavaScript API 위성지도: VITE_GOOGLE_MAPS_API_KEY — 리퍼러 제한 키만
npm run build:pages
npm run dev:local
```

빠른 로컬 서버는 `http://127.0.0.1:5173/`에서 같은 화면과 실제 API를 제공합니다. SQLite에 시연 자료를 저장하고 기존 D1 API와 동일한 경로를 사용합니다. 기본 접속은 이 컴퓨터에서만 허용합니다. `.local-data/`와 `.env.local`은 Git에서 제외합니다.

Cloudflare Workers/D1 배포용 구현도 유지합니다.

```sh
npm run build
npm run db:local
npm run dev
```

```sh
npm test
npm run typecheck
npm run test:flows
```

`npm test`는 AI 해석·주문 흐름·메뉴 9종·지도/메뉴 순서(`tests/map.test.mjs`: 메뉴 순서, 픽업대=시장 중심, 투영 정확도, 화면 안 마커 배치, 위성 URL, 키 노출 범위)를 검사합니다. `test:flows`는 로컬 서버에서 시연 예약·준비·수령·추가 구매·정산을 생성해 검사합니다. 실운영 데이터에 실행하지 마세요. 테스트는 실제 결제를 하지 않습니다.

## 주요 파일

- `app/page.tsx`, `components/market-map.tsx`, `components/order-center.tsx`: 고객·상인 화면
- `lib/navigation.ts`: 메뉴 순서(AI 한 끼→공동 메뉴→시장 지도→주문·픽업→추가 장보기)와 첫 화면
- `lib/market-geo.ts`, `lib/google-maps-loader.ts`: 시장 좌표·공동 픽업대·가상 점포 배치, Google 위성지도 URL·투영·Maps JS 로더
- `components/qr-code.tsx`: 로컬 QR 생성·저장·복사
- `lib/catalog.ts`: 9메뉴·6점포·18재료와 공급 관계
- `lib/ai.ts`, `lib/intent.ts`, `lib/adapt.ts`: Gemini 해석·스키마·근거·제약 검증
- `lib/order-workflow.ts`: 점포별 준비·정산 계산과 상태 전이
- `lib/static-api.ts`: 공개 체험판 브라우저 저장
- `app/api/workflow/route.ts`: 서버 준비·수령·추가 구매·정산 API
- `scripts/local-server.mjs`: 빠른 로컬 서버 실행
- `public/images/recipes/manifest.json`: 9장 이미지의 실제 생성 프롬프트와 built-in 도구 사용 기록

실제 점포 위치·픽업대 설치 장소, 참여 계약, 조리 검증, 상인 관리 권한, 결제·정산·배송 운영은 현장 실증 단계에서 확보합니다. 아파트 공동 배송은 후속 계획입니다.

Gemini 공식 문서: [모델](https://ai.google.dev/gemini-api/docs/models/gemini-3.1-flash-lite), [API](https://ai.google.dev/api/generate-content).
