# 2026-05-29 보안 엔진 검증 보고서

## 1. 테스트 셋 구성

- 총 URL: 220건
- 정상 기대값: 106건
- 악성 기대값: 114건

| 카테고리 | 건수 |
|---|---:|
| custom_corpus | 5 |
| dga | 5 |
| httpbin_redirect_chain | 5 |
| normal | 100 |
| openphish_malicious | 100 |
| suspicious_url_string | 5 |

## 2. 검증 결과

| API | 호출 | 판정 | Coverage | NO_VERDICT | PAGE_UNAVAILABLE | TP | FP | TN | FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| db_dependent | 220 | 185 | 84.09% | 35 | 35 | 89 | 0 | 91 | 5 | 97.30% | 100.00% | 94.68% | 97.27% |
| db_independent | 220 | 185 | 84.09% | 35 | 35 | 88 | 0 | 92 | 5 | 97.30% | 100.00% | 94.62% | 97.24% |

카테고리별 결과:

### db_dependent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| custom_corpus | 5 | 1 | 4 | 1 | 0 | 0 | 0 | 100.00% | 100.00% |
| dga | 5 | 5 | 0 | 5 | 0 | 0 | 0 | 100.00% | 100.00% |
| httpbin_redirect_chain | 5 | 5 | 0 | 0 | 0 | 5 | 0 | 100.00% | 0.00% |
| normal | 100 | 86 | 14 | 0 | 0 | 86 | 0 | 100.00% | 0.00% |
| openphish_malicious | 100 | 84 | 16 | 79 | 0 | 0 | 5 | 94.05% | 94.05% |
| suspicious_url_string | 5 | 4 | 1 | 4 | 0 | 0 | 0 | 100.00% | 100.00% |

### db_independent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| custom_corpus | 5 | 1 | 4 | 1 | 0 | 0 | 0 | 100.00% | 100.00% |
| dga | 5 | 5 | 0 | 5 | 0 | 0 | 0 | 100.00% | 100.00% |
| httpbin_redirect_chain | 5 | 5 | 0 | 0 | 0 | 5 | 0 | 100.00% | 0.00% |
| normal | 100 | 87 | 13 | 0 | 0 | 87 | 0 | 100.00% | 0.00% |
| openphish_malicious | 100 | 83 | 17 | 78 | 0 | 0 | 5 | 93.98% | 93.98% |
| suspicious_url_string | 5 | 4 | 1 | 4 | 0 | 0 | 0 | 100.00% | 100.00% |

- 정상 URL 오탐(FP): 0건
- 악성 URL 미탐(FN): 10건
- verdict 없음: 70건

## 3. 개선 방안

- 이번 결과에서는 정상 URL 오탐이 없거나 매우 낮다. 현재 보정은 유지하고 신뢰 도메인 목록 변경 시 회귀 테스트를 추가한다.
- 악성 미탐은 FETCH_FAILED, NEW_DOMAIN 단독, 콘텐츠/AI 미호출 케이스를 분리해 본다.
  - FN: db_dependent openphish_031 score=25 domain=NEW_DOMAIN content=FETCH_FAILED
  - FN: db_independent openphish_031 score=25 domain=NEW_DOMAIN content=FETCH_FAILED
  - FN: db_dependent openphish_032 score=25 domain=NEW_DOMAIN content=FETCH_FAILED
  - FN: db_independent openphish_032 score=25 domain=NEW_DOMAIN content=FETCH_FAILED
  - FN: db_dependent openphish_034 score=25 domain=NEW_DOMAIN content=FETCH_FAILED
- verdict 없는 페이지는 정확도 분모와 분리하고, Spring 콜백에는 `PAGE_UNAVAILABLE` 상태와 HTTP status를 전달한다.
  - NO_VERDICT_PAGE_UNAVAILABLE: 70건
