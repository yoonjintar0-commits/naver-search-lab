# 네이버 쿼리 추정기 — Claude 인수인계

작성일: 2026-10-20 KST. 기준: 2026-10-19 공개 배포 버전 14.

## 1. 현재 제품과 접근 주소

최종 방향은 **쿼리 추출 전용 도구**다. 과거 추가했던 광고비·유튜브·뉴스·이슈 기능은 사용자가 모두 취소했고 화면과 서버에서 제거했다. 재추가하지 말 것.

- 서비스: https://auto-search-pulse.yoonjintar0.chatgpt.site
- 공개 GitHub: https://github.com/yoonjintar0-commits/naver-search-lab
- GitHub 기준 커밋: 548f2ffa70cc00e9170972faff8d98fe652cc6c0
- Sites 실제 배포 소스 커밋: 570f5439ff742e700dc55fa2b44daa8dd5c2ab1c
- Sites project_id: appgprj_6ad2b0de396c8191af6d76035a780db9
- 마지막 확인 버전: 14, 공개, 사이트명 네이버 쿼리 추정기.

GitHub와 Sites는 별도 저장소·배포다. GitHub Pages 프런트엔드는 공개 Sites 서버 API를 호출한다. GitHub 푸시만으로 Sites 서버가 갱신되지는 않는다. 동봉 source/는 현재 동작을 재현하는 소스·테스트·빌드 파일이며 Git 이력·인증키·런타임 R2 전체는 포함하지 않는다.

## 2. 확정된 요구사항

- 키워드를 줄바꿈으로 입력하고 전체를 하나의 분석 대상으로 묶는다. 최대 100개, 각 공백 제외 30자. NFKC와 대문자 비교로 중복 제거.
- 브랜드명 옆 자동 버튼: 검색광고 API 연관 키워드 최대 30개를 채운다. 브랜드명 우선, 브랜드 포함 여부와 검색량 기준 정렬. 추천 후 직접 수정·되돌리기 가능. 추천만으로 추출/교체하지 않는다.
- 시작/종료일 선택. 최대 366일, 2016-01-01부터 한국시간 어제까지. 기본 최근 365일.
- 검색 버튼에 Budget N건 사용 예상 표시. full 성·연령 포함 수집 고정이며 추출 범위 옵션은 없다.
- 상단 오른쪽 작은 MONTHLY API BUDGET, 전체 엑셀, 그 오른쪽 같은 디자인의 기존 분석 불러오기, 결과 새로고침.
- 새 분석 전 엑셀 저장 안내. 성공하면 공유 서버의 현재 결과를 교체하고 이전 작업 데이터를 삭제한다. 실패하면 기존 결과 유지. 자동 예약 수집/영구 키워드 캐시/추출 이력 메뉴는 없다.
- 추출 중 전체 화면 반투명 덮개에 진행 막대·백분율만 표시. 배경 inert, 스크롤 잠금, 키보드 초점 제한. 실제 완료 단계 기반이며 다운로드 바이트 진행률은 아니다. 완료/실패 후 해제, 실패 단계 재시도 가능.
- 가장 큰 그래프 제목: 일별 흐름. 전체 키워드의 추정 쿼리 합계를 굵고 부드러운 한 선으로 표시. monotone cubic 보간, 결측 구간은 연결하지 않는다.
- 필터 버튼으로 세부 키워드 선택을 펼친다. 메인은 항상 전체 합계이고 필터는 하단 세부 흐름/PC·MO 막대에 적용한다.
- 세부 흐름은 선택 키워드의 일별 추정 건수. 상승률/차분이나 상대 관심지수를 표시하지 않는다.
- PC/MO는 선택 기간의 추정량 합계를 누적 막대로 표시한다. 100% 비율 막대가 아니며 키워드마다 높이가 다르다.
- 성·연령 지수는 원자료로 수집하지만 현재 화면에 성·연령 그래프는 없다. 실제 성·연령 절대 쿼리 수를 확보했다고 표현하지 않는다.
- 전체 엑셀 보이는 열: 그룹/키워드/성별/연령/PC/MO/추정 쿼리수(정수). 날짜·시계열은 숨김 _복원정보 시트의 JSON 청크로 보존한다. 절대량 근거 없는 성·연령행 숫자는 공란. 일반 임의 엑셀은 날짜별 복원이 불가능하다.
- 페이지 하단 작은 글씨로 추정 공식·한계를 설명한다.
- 팀원이 회사 PC에서 새 검색 가능하도록 키는 서버에만 저장. 별도 로그인 잠금은 사용자 요청상 없다. 현재 결과는 모든 방문자가 공유한다.

## 3. 데이터 링크와 동봉 데이터

현재 서버 전체 내보내기(data+groups):
https://auto-search-pulse.yoonjintar0.chatgpt.site/api/export

원시 데이터:
https://auto-search-pulse.yoonjintar0.chatgpt.site/api/data

그룹:
https://auto-search-pulse.yoonjintar0.chatgpt.site/api/groups

사용량 추정:
https://auto-search-pulse.yoonjintar0.chatgpt.site/api/quota

위 링크는 서버의 현재 값이다. 새로운 분석 성공 시 내용이 달라진다. 일부 외부 환경에서는 Sites 경계에서 HTTP 403을 받을 수 있다. HTML/403 응답을 JSON으로 저장하지 말 것. 브라우저 전체 엑셀 또는 동봉 데이터가 대안이다. 이번에는 기존 서버 인증 경로로 실제 JSON을 확보했으며 인증 토큰은 포함하지 않았다.

| 동봉 파일 | 설명 |
|---|---|
| data/current-export.json | 실제 현재 서버에서 받은 data+groups |
| data/current-daily-estimates.csv | 같은 보정식으로 계산한 날짜별 PC/MO 소수·합계 정수. 공란은 미확보 |
| data/archive-22-keywords.json | 별도의 이전 22개 키워드 백업 |
| data/manifest.json | 기간·수집시각·키워드·SHA256 해시·출처 |

현재 서버: 4개 키워드(BMWi5, 비엠더블유i5, BMWiX, 비엠더블유iX), 2026-07-17~2026-10-16(92일). collected_at=2026-10-19T08:48:23.607Z. 두 한글 별칭의 PC 월간값은 "< 10"이어서 현재 로직은 해당 키워드의 전체 일별 추정량을 결측 처리한다. 0으로 대체하지 않는다.

이전 백업: BMW·벤츠·테슬라 브랜드 및 모델 22개. 날짜 배열 2025-10-19~2026-10-18(365일), collected_at=2026-10-17T01:31:50.639Z. 수집시각이 날짜 배열 마지막보다 이르므로 날짜마다 모두 관측됐다는 뜻이 아니다. coverage/결측을 확인해야 한다. 모델별 30일 calibration 배열이 없는 구형 구조다. 현재 데이터와 자동 병합하거나 최신으로 덮어쓰지 않는다.

이번 내보내기는 현재 분석과 별도 이전 백업이다. R2의 usage/quota/job 메타·부분 결과 전체 또는 다른 과거 분석까지 보존한 것은 아니다.

## 4. 추정 방식과 해석

키워드 k, 일자 d:
Q(k,d) = M_pc(k) × I_pc(k,d) / Σ_anchor I_pc(k) + M_mo(k) × I_mo(k,d) / Σ_anchor I_mo(k)

- M: 검색광고 /keywordstool에서 현재 조회한 월간 PC/MO 검색수.
- I: API HUB 검색어트렌드의 일별 상대지수.
- anchor: 추출 시점 한국시간 어제까지 30일. 사용자가 선택한 과거 기간 마지막 30일과 다를 수 있다.
- PC/MO는 요청 기간과 anchor를 포함하는 한 요청에서 조회해 정규화 스케일을 맞춘다. calibration에 anchor 날짜/PC/MO 지수를 따로 저장한다.
- 월간 검색수의 정확한 집계 시작/종료일은 API가 제공하지 않아 30일은 가정이다. 과거 검색량은 현재 월간값의 고정 보정계수를 소급 적용한 추정이며 실제 일별 건수가 아니다.
- PC/MO 월간값 모두 수치, anchor 각 30일 결측 없음, 분모 양수일 때만 계산한다. <10, < 10, 누락/0 분모를 0으로 바꾸지 않는다.
- 성·연령 필터 지수는 요청마다 독립 정규화된다. 상대지수끼리 나눠 성/연령 점유율이나 절대 검색수를 얻을 수 없다. 초기 대화에서 가능하다고 가정했던 방식은 계산 근거로 쓰지 않는다.
- 메인 합계는 확보된 키워드만 더하면 부분 집계로 표시하며, 모두 결측인 날은 결측이다.
- 내부 소수 유지, 화면 반올림. CSV는 합계 반올림, 제품 엑셀은 PC/MO 각각 반올림 후 합계라서 정수 총계가 1 다를 수 있다.

## 5. API Budget

full 23세그먼트 = 전체 1 + PC/MO 2 + 성별 2 + 연령 6 + 성×연령 12.
한 트렌드 요청에 최대 5개 키워드 그룹, 각각 단어 1개씩. 예상 사용량은 23 × ceil(중복 제거 키워드 수 / 5). 선택 기간이 짧아져도 호출 수는 같다. 추천·월간 검색광고 조회는 별도 API이며 이 트렌드 Budget에 포함하지 않는다.

현재 앱 기본 설정은 월 50,000회, 일간 제한 null(없음)이다. 계정/상품 전체의 보편적 한도를 의미하지 않는다. 계정 공식 실시간 사용량을 조회하는 API는 구현하지 않았고 사이트 호출 시도 및 체크포인트로 추정한다. 다른 앱 사용·실패·반영 지연에 따라 콘솔과 다를 수 있다. 남은 키워드 수 계산에는 5개 배치 단위 올림을 반영한다.

## 6. 파일과 실행

| 파일 | 역할 |
|---|---|
| web/index.html | UI/CSS, 상단 버튼, 추출 덮개 |
| web/app.js | 입력·추천·추출 단계·복원·필터·추정·세부 그래프 |
| web/query-chart.js | 메인 합계 그래프와 부드러운 보간 |
| web/xlsx.js | 외부 의존성 없는 XLSX 생성·해제·복원·구형 엑셀 호환 |
| web/config.js | 같은 origin에서 빈 API base |
| worker/runtime.js | Naver 호출·서명, R2, 작업 단계, CORS, quota |
| worker/keywords.js | query-only 메타 정리, 연관 키워드 추천 |
| scripts/compile-worker.mjs | web 자산/빈 SEED/코드를 단일 worker/index.js로 합침 |
| scripts/verify.mjs | 외부 API mock 회귀 테스트 |
| scripts/preview.mjs | 화면 전용 미리보기, POST 차단 |
| scripts/build-public.mjs | GitHub Pages docs 자산·backend URL·CNAME 생성 |
| .openai/hosting.json | 기존 Sites 식별자와 BUCKET 출처 |
| wrangler.example.jsonc | 다른 Worker 환경으로 이관할 때 참고할 예시 |

Node.js 22+ 권장. 앱 런타임 외부 npm 의존성 없음.

source 폴더에서 순서대로:
1. node scripts/compile-worker.mjs
2. bash scripts/build.sh
3. node scripts/validate-artifact.mjs
4. node scripts/verify.mjs
5. npm run dev

미리보기는 localhost:4173이고 실제 API 추출 POST를 의도적으로 거부한다. 시작 SEED는 빈 배열이다. 로컬에서 결과를 보려면 current-export.json의 data 객체만 .openai/preview-data.json으로 저장한다. 운영 /api/import를 테스트 목적으로 호출하지 말 것.

동봉 source는 Sites의 테스트/빌드 스크립트와 GitHub의 공개 빌드 스크립트를 결합했다. 공개 GitHub에 없는 테스트 파일도 있다. 오래된 Python 수집 스크립트와 monthly/trends 자산은 제외했다. 현재 UI에 의존성이 없다. .openai의 기존 프로젝트 ID는 출처 기록이며 다른 계정에서 새 프로젝트 ID로 임의 재사용하지 않는다.

## 7. 서버 계약과 저장소

- GET /api/data, /api/groups, /api/export, /api/quota, /api/status.
- POST /api/keywords/suggest: {brand} → {keywords, limit, source, trend_calls:0,...}.
- POST /api/jobs: {groups:[{id,name,keywords,color}],start_date,end_date} → {id,steps,estimated_calls,...}.
- POST /api/jobs/{id}/step?name={step}, 빈 JSON body. monthly → 23세그먼트 → finalize. 완료 단계는 캐시 재사용.
- GET /api/jobs/{id}, GET /api/jobs/{id}/result.
- POST /api/import: {data,groups}, 검증 후 공유 현재 분석 교체.
- /api/analysis 및 /api/issues는 제거되어 404.
- R2 BUCKET: latest.json, groups.json, workspace/active.json, workspace/current.json, workspace/group-mapping-v1.json, jobs/{id}/meta.json, 단계/부분 배치 결과, usage 및 quota.
- 완료 시 이전 cache/snapshots/runs/auto/jobs 정리, 현재 작업 meta/result만 유지. usage/quota 유지. 최신 결과 하나만 보존하는 의도된 동작.
- 실행 중 작업으로 새 작업 제한, 2시간 초과 작업은 새 작업 허용. 실패 배치 재사용으로 재시도 호출 절약. 서로 다른 분석 간 영구 캐시는 없다.
- CORS: 같은 origin, keyword.jintar.co.kr의 HTTP/HTTPS, https://yoonjintar0-commits.github.io. 이관 프런트엔드 origin이 바뀌면 갱신.

## 8. 비밀값과 호스팅

필수 서버 환경변수 이름:
NAVER_HUB_CLIENT_ID
NAVER_HUB_CLIENT_SECRET
NAVER_SEARCHAD_LICENSE
NAVER_SEARCHAD_SECRET
NAVER_SEARCHAD_CUSTOMER

실제 값은 기존 서버 환경변수에 있다. 패키지·GitHub·브라우저 JS에는 넣지 않았다. Claude가 기존 서버 관리 권한을 갖는 것은 아니다. 서버 이관 시 사용자에게 안전한 환경변수 방식으로 다시 설정받는다.

트렌드: POST https://naverapihub.apigw.ntruss.com/search-trend/v1/search
헤더 X-NCP-APIGW-API-KEY-ID / X-NCP-APIGW-API-KEY.
검색광고: GET https://api.searchad.naver.com/keywordstool
Timestamp + ".GET./keywordstool"를 검색광고 secret으로 HMAC-SHA256 후 base64 서명.

Claude에서 프런트엔드만 수정할 때 기존 API를 유지할 수 있다. 서버까지 수정하려면 Sites 관리 기능/권한이 필요하다. 없으면 사용자 소유 Cloudflare Workers/R2 등으로 이전한다. ESM Worker와 R2 get/put/list/delete 사용. wrangler 예시는 bucket 이름을 채우고 별도 CLI/권한 설정이 필요하며 자동 배포를 만든 것이 아니다. 새 서버 데이터 복원 전 백업하고 API base와 CORS를 함께 바꾼다.

## 9. 미완료: keyword.jintar.co.kr

도메인 DNS/HTTPS 완료를 선언하지 말 것. 2026-10-20 확인 시 Sites custom domain 목록은 비어 있었다. GitHub docs/CNAME에는 hostname이 있고, 이전 작업 보고상 GitHub Pages custom domain 등록은 했으나 DNS 변경과 HTTPS 검증은 끝나지 않았다.

관리 서비스: 호스팅케이알.
실제 공식 도메인 목록 URL: https://app.hosting.kr/domains/portfolio/owned
사용자가 처음 적은 app.hosing.kr에는 t가 빠져 있다.
Megazone Accounts 로그인으로 이동. 지정 Google 계정: yoonjintar2.

마지막 시도: 보안 browserAuth Google 로그인 요청이 시간 초과했다. 로그인 성공/DNS 변경을 확인하지 못했다. 일반 Google 버튼 직접 클릭은 자동 보안 검토에서 거부되어 안전한 인증 경로로 전환했으나 완료되지 않았다. 새 세션에서 로그인부터 확인한다. 비밀번호/OTP를 채팅으로 받지 않는다.

GitHub Pages 경로를 유지하면 예정 record:
host keyword / type CNAME / target yoonjintar0-commits.github.io
https://나 /naver-search-lab 경로를 붙이지 않는다. 아직 저장하지 않은 계획이다.
기존 DNS/권한 네임서버를 먼저 확인하고 다른 서브도메인·메일·루트 레코드는 수정하지 않는다.
GitHub Pages main /docs, custom domain, DNS check, HTTPS 인증서, Enforce HTTPS, 실제 사이트/API CORS까지 검증한다.

Sites 직접 custom domain 경로를 선택한다면 Sites가 반환한 CNAME 및 검증 레코드를 사용해야 한다. GitHub target과 혼합하지 않는다.

## 10. 확인 범위와 다음 작업

버전14에서 compile/build/artifact 검사와 mock 회귀 테스트를 통과했다. 브라우저에서 상단 버튼 배치, 외부 분석 기능 제거, 추출 덮개 및 inert 배경 확인. 성공/실패 해제·재시도, 추천/편집/되돌리기/입력 변경 경쟁 조건, 구형 엑셀 복원, 보정, CORS, XLSX 다운로드도 mock 검증했다. 운영 결과를 덮어쓰는 새 추출 테스트는 하지 않았다.

우선순위:
1. 데이터 보관, 현재 소스 실행·회귀 검증.
2. 도메인 DNS·HTTPS·CORS 마무리.
3. 기존 서버 유지 vs 서버 이관 결정.
4. 현재 화면에 쓰지 않는 성·연령 20개 세그먼트 수집을 계속할지는 사용자 확인 후 변경. full 23회 계약을 몰래 3회로 바꾸지 않는다.
5. 수치 품질을 보완할 때 추정 불가/부분 집계/보정 가정을 유지하고 관측값으로 표현하지 않는다.

