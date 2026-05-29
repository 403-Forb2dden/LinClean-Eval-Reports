# 2026-05-28 보안 엔진 검증 보고서

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
| db_dependent | 220 | 182 | 82.73% | 38 | 38 | 56 | 3 | 88 | 35 | 79.12% | 94.92% | 61.54% | 74.67% |
| db_independent | 220 | 179 | 81.36% | 41 | 39 | 66 | 4 | 85 | 24 | 84.36% | 94.29% | 73.33% | 82.50% |

카테고리별 결과:

### db_dependent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| custom_corpus | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% |
| dga | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% |
| httpbin_redirect_chain | 5 | 5 | 0 | 0 | 0 | 5 | 0 | 100.00% | 0.00% |
| normal | 100 | 86 | 14 | 0 | 3 | 83 | 0 | 96.51% | 0.00% |
| openphish_malicious | 100 | 91 | 9 | 56 | 0 | 0 | 35 | 61.54% | 61.54% |
| suspicious_url_string | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% |

### db_independent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| custom_corpus | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% |
| dga | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% |
| httpbin_redirect_chain | 5 | 4 | 1 | 0 | 0 | 4 | 0 | 100.00% | 0.00% |
| normal | 100 | 85 | 15 | 0 | 4 | 81 | 0 | 95.29% | 0.00% |
| openphish_malicious | 100 | 90 | 10 | 66 | 0 | 0 | 24 | 73.33% | 73.33% |
| suspicious_url_string | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% |

- 정상 URL 오탐(FP): 7건
- 악성 URL 미탐(FN): 59건
- verdict 없음: 79건

## 3. 개선 방안

- 정상 오탐은 신뢰 도메인, 정상 리다이렉트, AI suspicious 단독 가산 여부를 우선 확인한다.
  - FP 패턴 4건: domain=TYPO_DOMAIN, content=LOGO_ALT_IMPERSONATION, ai=suspicious
  - FP 패턴 2건: domain=DGA_LIKE, content=LOGO_ALT_IMPERSONATION, ai=suspicious
  - FP 패턴 1건: domain=-, content=PII_COLLECTION_FORM|LOGO_ALT_IMPERSONATION, ai=suspicious
- 악성 미탐은 FETCH_FAILED, NEW_DOMAIN 단독, 콘텐츠/AI 미호출 케이스를 분리해 본다.
  - FN: db_dependent openphish_002 score=- domain=- content=-
  - FN: db_independent openphish_002 score=- domain=- content=-
  - FN: db_dependent openphish_004 score=20 domain=- content=-
  - FN: db_independent openphish_004 score=20 domain=- content=-
  - FN: db_dependent openphish_005 score=- domain=- content=-
- verdict 없는 페이지는 정확도 분모와 분리하고, Spring 콜백에는 `PAGE_UNAVAILABLE` 상태와 HTTP status를 전달한다.
  - NO_VERDICT_PAGE_UNAVAILABLE: 77건
  - NO_VERDICT_REQUEST_FAILED: 2건
