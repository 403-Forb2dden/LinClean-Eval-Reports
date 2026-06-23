# LinClean Eval Reports

LinClean FastAPI URL 보안 엔진의 날짜별 평가 결과 저장소입니다. 각 날짜 디렉터리는 같은 구조를 유지합니다.

```text
YYYY-MM-DD/
  dataset.csv   # 평가 입력 URL과 기대 라벨
  results.csv   # 실행한 API의 상세 결과
  report.md     # 테스트 셋 구성, 검증 결과, 개선 방안 요약
  report.xlsx   # 한글 Excel 요약, 상세 결과, AI 답변 정리
```

## 최신 결과

기준일: **2026-06-23**

2026-06-23 평가는 실행 시점 OpenPhish 공개 feed의 최신 악성 URL 100건만으로 구성한 악성 URL 회귀 테스트입니다. DB 포함 파이프라인 API(`/api/v1/analyze/sync`)만 사용했습니다.

- 총 URL: 100건
- 정상 기대값: 0건
- 악성 기대값: 100건
- 정상 URL 오탐(FP): 0건
- 악성 URL 미탐(FN): 원시 19건, 외부 URL 상태 제외 기준 2건
- verdict 없음: 13건
- 외부 URL 상태 문제로 분리한 FN: 17건
- verdict 없음 원인: `NO_VERDICT_PAGE_UNAVAILABLE` 12건, `NO_VERDICT_REQUEST_FAILED` 1건
- Excel 리포트: `2026-06-23/report.xlsx`

| API | 호출 | 판정 | Coverage | NO_VERDICT | PAGE_UNAVAILABLE | TP | FP | TN | FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| db_dependent | 100 | 87 | 87.00% | 13 | 12 | 68 | 0 | 0 | 19 | 78.16% | 100.00% | 78.16% | 87.74% |

외부 URL 상태 문제 제외 기준:

| API | 호출 | 조정 판정 | 제외 | 외부 URL 상태 FN 제외 | TP | FP | TN | 조정 FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| db_dependent | 100 | 70 | 30 | 17 | 68 | 0 | 0 | 2 | 97.14% | 100.00% | 97.14% | 98.55% |

## 대표 지표

| 테스트 | API | 기준 | 악성 Recall | Accuracy | 정상 오탐 |
|---|---|---|---:|---:|---:|
| 2026-06-23 | db_dependent | 원시 판정 가능 기준 | 78.16% | 78.16% | 0 |
| 2026-06-23 | db_dependent | 외부 URL 상태 제외 | 97.14% | 97.14% | 0 |

- 2026-06-23의 데이터셋은 실행 시점 OpenPhish 공개 feed 상위 100개 URL을 스냅샷으로 저장했다.
- 이번 평가는 정상 URL을 포함하지 않는 악성 URL recall 중심 회귀 테스트다.
- Accuracy, Precision, Recall, F1은 `NO_VERDICT`와 `PAGE_UNAVAILABLE`을 제외한 판정 가능 건(`TP`, `FP`, `TN`, `FN`)만 분모로 계산했다.
- 외부 URL 상태 제외 기준은 `FETCH_FAILED`가 발생했고 원인이 `pipeline_timeout` 또는 `unexpected_redirect`인 FN 17건을 OpenPhish URL 만료, 접속 차단, 응답 지연, 리다이렉트 정책 변화로 분리해 추가 제외한 지표다.
- 출처: https://openphish.com/feed.txt

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
| 2026-06-05 | 0 | 0 | 14 | APWG/Netcraft 기반 공용 웹 분포 반영. 99 정상, 1 악성. DB 포함 API만 평가. |
| 2026-06-23 | 0 | 19 raw / 2 adjusted | 13 | OpenPhish 최신 악성 URL 100건 평가. 외부 URL 상태 제외 recall 97.14%. |

## 해석 기준

- `caution`과 `danger`는 탐지 양성으로 계산합니다.
- 찾을 수 없는 페이지, 400번대 접근 실패, 사라진 OpenPhish URL은 verdict를 만들지 않고 `PAGE_UNAVAILABLE`로 분리합니다.
- `NO_VERDICT_PAGE_UNAVAILABLE`은 Accuracy, Precision, Recall, F1 계산 분모에서 제외하고, Spring 콜백에는 실패 상태와 HTTP status code를 전달하는 것이 현재 정책입니다.
- `FETCH_FAILED` 기반 `pipeline_timeout`, `unexpected_redirect` FN은 외부 URL 상태 문제로 별도 분리해 조정 지표를 함께 표시합니다.
- 2026-06-23 기준 최신 평가는 OpenPhish 악성 URL 100건만 사용한 recall 중심 회귀 테스트입니다. 정상 URL precision은 별도 공용 웹 분포 테스트와 함께 해석해야 합니다.

## 테스트 셋 구성

2026-06-23 기준 데이터셋 구성은 다음과 같습니다.

| 카테고리 | 건수 | 설명 |
|---|---:|---|
| openphish_malicious | 100 | 실행 시점 OpenPhish 공개 feed 최신 악성 URL |

## 파일 관리 원칙

- 날짜별 디렉터리에는 기본적으로 `dataset.csv`, `results.csv`, `report.md`를 두고, 사람이 검토하기 쉬운 Excel 리포트가 필요한 경우 `report.xlsx`를 함께 둡니다.
- 임시 summary, 로그, 실행 중 산출물은 커밋하지 않습니다.
- 외부 URL 상태는 시간이 지나며 바뀌므로, 날짜별 결과는 같은 날짜의 데이터셋과 함께 해석해야 합니다.
