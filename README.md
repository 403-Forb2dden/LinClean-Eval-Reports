# LinClean Eval Reports

LinClean FastAPI URL 보안 엔진의 날짜별 평가 결과 저장소입니다. 각 날짜 디렉터리는 같은 구조를 유지합니다.

```text
YYYY-MM-DD/
  dataset.csv   # 평가 입력 URL과 기대 라벨
  results.csv   # db_dependent, db_independent 실행 결과
  report.md     # 테스트 셋 구성, 검증 결과, 개선 방안 요약
  report.xlsx   # 한글 Excel 요약, 상세 결과, AI 답변 정리
```

## 최신 결과

기준일: **2026-06-05**

이번 평가부터 실제 공용 웹에서 phishing URL이 차지하는 비율을 반영한 1000건 테스트와, OpenPhish 악성 URL 100건 보조 테스트를 분리해 보관합니다.

### 2026-06-05-public-web-1000

- 총 URL: 1000건
- 정상 기대값: 999건
- 악성 기대값: 1건
- 정상 URL 오탐(FP): 219건
- 악성 URL 미탐(FN): 0건
- verdict 없음: 885건
- verdict 없음 원인: `NO_VERDICT_PAGE_UNAVAILABLE` 634건, `NO_VERDICT_REQUEST_FAILED` 251건
- Excel 리포트: `2026-06-05-public-web-1000/report.xlsx`

| API | 호출 | 판정 | Coverage | NO_VERDICT | PAGE_UNAVAILABLE | TP | FP | TN | FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| db_dependent | 1000 | 609 | 60.90% | 391 | 367 | 1 | 119 | 489 | 0 | 80.46% | 0.83% | 100.00% | 1.65% |
| db_independent | 1000 | 506 | 50.60% | 494 | 267 | 1 | 100 | 405 | 0 | 80.24% | 0.99% | 100.00% | 1.96% |

### 2026-06-05-openphish-100

- 총 URL: 100건
- 정상 기대값: 0건
- 악성 기대값: 100건
- 정상 URL 오탐(FP): 0건
- 악성 URL 미탐(FN): 17건
- verdict 없음: 35건
- verdict 없음 원인: `NO_VERDICT_PAGE_UNAVAILABLE` 7건, `NO_VERDICT_REQUEST_FAILED` 28건
- Excel 리포트: `2026-06-05-openphish-100/report.xlsx`

| API | 호출 | 판정 | Coverage | NO_VERDICT | PAGE_UNAVAILABLE | TP | FP | TN | FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| db_dependent | 100 | 93 | 93.00% | 7 | 4 | 81 | 0 | 0 | 12 | 87.10% | 100.00% | 87.10% | 93.10% |
| db_independent | 100 | 72 | 72.00% | 28 | 3 | 67 | 0 | 0 | 5 | 93.06% | 100.00% | 93.06% | 96.40% |

## 대표 지표

| 테스트 | API | 전체 악성 기준 탐지율 | 판정 가능 악성 기준 Recall | 정상 오탐 |
|---|---|---:|---:|---:|
| public-web-1000 | db_dependent | 100.00% | 100.00% | 119 |
| public-web-1000 | db_independent | 100.00% | 100.00% | 100 |
| openphish-100 | db_dependent | 81.00% | 87.10% | 0 |
| openphish-100 | db_independent | 67.00% | 93.06% | 0 |

- public-web-1000의 999:1 구성은 APWG Q1 2026 unique phishing Web sites/attacks 971,181건과 Netcraft April 2026 전체 사이트 1,433,742,238개를 기준으로 계산했다.
- 악성 비율은 `971,181 / 1,433,742,238 = 0.06774%`이고, 1000건 테스트 셋에서는 `round(0.6774) = 1`건을 악성으로 배정했다.
- OpenPhish 100건 테스트는 공용 웹 비율 테스트에서 부족한 악성 표본에 대한 보조 회귀 테스트다.
- 출처: https://docs.apwg.org/reports/apwg_trends_report_q1_2026.pdf, https://www.netcraft.com/blog/april-2026-web-server-survey

## 날짜별 추이

| 날짜 | 정상 오탐(FP) | 악성 미탐(FN) | verdict 없음 | 요약 |
|---|---:|---:|---:|---|
| 2026-05-23 | 8 | 9 | 0 | 초기 기준선. 전체 coverage 100%, 정상 URL 오탐 존재. |
| 2026-05-24 | 9 | 21 | 0 | 정상 오탐과 악성 미탐이 모두 증가. |
| 2026-05-25 | - | - | - | 결과 CSV 없음. 데이터셋 기준선만 보관. |
| 2026-05-26 | - | - | - | 결과 CSV 없음. 데이터셋 기준선만 보관. |
| 2026-05-27 | 4 | 31 | 114 | 파이프라인 오류/미판정 분리 전 결과. |
| 2026-05-28 | 7 | 59 | 79 | PAGE_UNAVAILABLE 분리 시작, 정상 오탐 재발. |
| 2026-05-29 | 0 | 10 | 70 | 정상 오탐 0건, precision 100%, page unavailable은 verdict 없이 분리. |
| 2026-05-30 | 2 | 0 | 39 | OpenPhish 최신화 후 판정 가능 악성 recall 100%, custom_corpus 정상 1건이 양쪽 API에서 오탐. |
| 2026-05-31 | 2 | 0 | 46 | OpenPhish 최신화 후 판정 가능 악성 recall 100%, AI 답변 요약 포함 Excel 리포트 추가. |
| 2026-06-05-public-web-1000 | 219 | 0 | 885 | APWG/Netcraft 기반 공용 웹 분포 반영. 999 정상, 1 악성. |
| 2026-06-05-openphish-100 | 0 | 17 | 35 | OpenPhish 악성 URL 100건 보조 회귀 테스트. |

## 해석 기준

- `caution`과 `danger`는 탐지 양성으로 계산합니다.
- 찾을 수 없는 페이지, 400번대 접근 실패, 사라진 OpenPhish URL은 verdict를 만들지 않고 `PAGE_UNAVAILABLE`로 분리합니다.
- `NO_VERDICT_PAGE_UNAVAILABLE`은 정확도 계산 분모에서 제외하고, Spring 콜백에는 실패 상태와 HTTP status code를 전달하는 것이 현재 정책입니다.
- 2026-06-05 기준 개선의 핵심은 테스트 셋 비율을 실제 공용 웹 분포에 맞춰 재구성한 점입니다. public-web-1000에서는 정상 URL 오탐이 크게 드러났고, openphish-100에서는 악성 URL 회귀 탐지 성능을 별도로 확인했습니다.

## 테스트 셋 구성

2026-06-05 기준 데이터셋 구성은 다음과 같습니다.

| 카테고리 | 건수 | 설명 |
|---|---:|---|
| public_web_safe | 999 | Tranco 및 고정 정상 URL 기반 공용 웹 정상 샘플 |
| openphish_malicious | 1 | public-web-1000에 포함된 OpenPhish 악성 샘플 |
| openphish_malicious | 100 | openphish-100 보조 테스트의 OpenPhish 악성 샘플 |

## 파일 관리 원칙

- 날짜별 디렉터리에는 기본적으로 `dataset.csv`, `results.csv`, `report.md`를 두고, 사람이 검토하기 쉬운 Excel 리포트가 필요한 경우 `report.xlsx`를 함께 둡니다.
- 임시 summary, 로그, 실행 중 산출물은 커밋하지 않습니다.
- 외부 URL 상태는 시간이 지나며 바뀌므로, 날짜별 결과는 같은 날짜의 데이터셋과 함께 해석해야 합니다.
