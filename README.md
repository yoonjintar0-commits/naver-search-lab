# Search Lab · 네이버 키워드 분석

BMW·벤츠·테슬라의 브랜드명과 모델 키워드를 그룹으로 묶어 최근 365일 검색 흐름을 비교합니다. PC·모바일 막대, 성·연령 상대지수, 그룹 연간 추정 쿼리, 전체 XLSX 다운로드를 제공합니다.

## GitHub Pages 공개 조회본

`docs/`는 실제 수집된 22개 키워드 데이터를 포함하는 공개 조회본입니다. 소유자가 데이터 공개를 승인했습니다. 그래프·필터·그룹 비교와 전체 엑셀 다운로드를 사용할 수 있습니다. 데이터 기준일은 페이지에 표시됩니다. API 키·비밀키·개인 인증 토큰은 포함하지 않습니다.

GitHub Pages는 정적 호스팅이므로 네이버 API의 비밀키를 이용하는 새 검색·갱신·영구 그룹 편집은 실행하지 않습니다. 해당 기능은 [개인 작업실](https://auto-search-pulse.yoonjintar0.chatgpt.site)에서 사용합니다. 개인 작업실의 접근 권한은 별도입니다. 공개 조회본에서 이 기능은 비활성화되어 API 호출량을 사용하지 않습니다.

GitHub 저장소 Settings → Pages에서 Deploy from a branch, `main`, `/docs`를 선택합니다. 별도 자동 수집이나 예약은 없습니다.

## 코드 구조

- `web/`: 원래 전체 기능의 프런트엔드
- `worker/runtime.js`: 서버 API, R2 저장, 누락 기간 캐시, 지수 연결
- `web/data/trend-latest.json`: 서버 초기 데이터 형식(현재 빈 데이터)
- `docs/manifest.json` 및 `docs/data/*.json.gz`: 공개 승인된 전체 수집 데이터 스냅샷
- `scripts/build-public.mjs`: GitHub Pages 파일 준비
- `scripts/compile-worker.mjs`: 서버용 자산 포함 ESM 생성

공개 페이지 빌드: `npm run build`

서버 빌드: `npm run build:worker`. Cloudflare Worker 또는 호환 런타임에 배포하고 R2 바인딩 `BUCKET`을 연결합니다. 서버에는 `NAVER_HUB_CLIENT_ID`, `NAVER_HUB_CLIENT_SECRET`, `NAVER_SEARCHAD_CUSTOMER`, `NAVER_SEARCHAD_LICENSE`, `NAVER_SEARCHAD_SECRET`을 비밀 환경변수로 설정해야 합니다. 키를 소스나 브라우저 JavaScript에 넣지 마세요. 서버를 공개 운영할 경우 인증과 호출 제한을 운영 환경에 맞게 추가해야 합니다. 이 저장소에는 배포 계정 정보나 기존 서버의 비밀 환경변수가 없습니다.

## 추정 방법

일별 추정 검색수 = 월간 PC 검색수 × 해당일 PC 지수 / 기준30일 PC 지수 합 + 월간 모바일 검색수 × 해당일 모바일 지수 / 기준30일 모바일 지수 합.

월간 값의 정확한 집계 기간이 응답에 없어 수집일 전날까지 30일로 가정한 소급 추정입니다. 네이버가 제공하는 실제 일별 건수가 아닙니다. 누락은 0으로 채우지 않습니다. 성·연령별 값은 독립적으로 정규화된 상대지수이며, 성·연령별 절대 검색량이나 구성비로 해석할 수 없습니다. 자세한 설명은 사이트 하단과 엑셀 안내 시트를 참고하세요.
