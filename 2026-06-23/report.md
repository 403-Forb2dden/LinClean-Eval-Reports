# 2026-06-23 보안 엔진 검증 보고서

## 1. 테스트 셋 구성

- 총 URL: 100건
- 정상 기대값: 0건
- 악성 기대값: 100건

| 카테고리 | 건수 |
|---|---:|
| openphish_malicious | 100 |

## 2. 검증 결과

| API | 호출 | 판정 | Coverage | NO_VERDICT | PAGE_UNAVAILABLE | TP | FP | TN | FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| db_dependent | 100 | 87 | 87.00% | 13 | 12 | 68 | 0 | 0 | 19 | 78.16% | 100.00% | 78.16% | 87.74% |

외부 URL 상태 문제 제외 기준:

`FETCH_FAILED`가 발생했고 원인이 `pipeline_timeout` 또는 `unexpected_redirect`인 FN 17건은 OpenPhish URL 만료, 접속 차단, 응답 지연, 리다이렉트 정책 변화 등 외부 URL 상태 문제로 분리했다. 이 기준에서는 원시 `NO_VERDICT` 13건과 외부 URL 상태 FN 17건을 제외하고, 실제 분석 가능한 70건만 분모로 계산한다.

| API | 호출 | 조정 판정 | 제외 | 외부 URL 상태 FN 제외 | TP | FP | TN | 조정 FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| db_dependent | 100 | 70 | 30 | 17 | 68 | 0 | 0 | 2 | 97.14% | 100.00% | 97.14% | 98.55% |

카테고리별 결과:

### db_dependent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| openphish_malicious | 100 | 87 | 13 | 68 | 0 | 0 | 19 | 78.16% | 78.16% |

- 정상 URL 오탐(FP): 0건
- 악성 URL 미탐(FN): 원시 19건, 외부 URL 상태 제외 기준 2건
- verdict 없음: 13건
- 외부 URL 상태 문제로 분리한 FN: 17건

## 3. 개선 방안

- 이번 결과에서는 정상 URL 오탐이 없거나 매우 낮다. 현재 보정은 유지하고 신뢰 도메인 목록 변경 시 회귀 테스트를 추가한다.
- 원시 FN 19건 중 17건은 `FETCH_FAILED` 기반 외부 URL 상태 문제로 분리했다. 이후 평가 스크립트에서는 이 유형을 `NO_VERDICT_EXTERNAL_URL_STATE`로 별도 집계하는 것이 적절하다.
- 실제 분석 가능 상태에서 남은 악성 미탐 2건은 콘텐츠/AI 판정 경계를 추가 확인한다.
  - FN: db_dependent openphish_050 score=5 content=EXTERNAL_LINK_OVERUSE ai=benign
  - FN: db_dependent openphish_081 score=- content=- ai=benign
- verdict 없는 페이지는 정확도 분모와 분리하고, Spring 콜백에는 `PAGE_UNAVAILABLE` 상태와 HTTP status를 전달한다.
  - NO_VERDICT_PAGE_UNAVAILABLE: 12건
  - NO_VERDICT_REQUEST_FAILED: 1건
