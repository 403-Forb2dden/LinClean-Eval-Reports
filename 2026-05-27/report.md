# 2026-05-27 보안 엔진 검증 보고서

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
| db_dependent | 220 | 163 | 74.09% | 57 | 0 | 56 | 2 | 90 | 15 | 89.57% | 96.55% | 78.87% | 86.82% |
| db_independent | 220 | 163 | 74.09% | 57 | 0 | 54 | 2 | 91 | 16 | 88.96% | 96.43% | 77.14% | 85.71% |

카테고리별 결과:

### db_dependent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| custom_corpus | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% |
| dga | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% |
| httpbin_redirect_chain | 5 | 5 | 0 | 0 | 0 | 5 | 0 | 100.00% | 0.00% |
| normal | 100 | 87 | 13 | 0 | 2 | 85 | 0 | 97.70% | 0.00% |
| openphish_malicious | 100 | 71 | 29 | 56 | 0 | 0 | 15 | 78.87% | 78.87% |
| suspicious_url_string | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% |

### db_independent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| custom_corpus | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% |
| dga | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% |
| httpbin_redirect_chain | 5 | 5 | 0 | 0 | 0 | 5 | 0 | 100.00% | 0.00% |
| normal | 100 | 88 | 12 | 0 | 2 | 86 | 0 | 97.73% | 0.00% |
| openphish_malicious | 100 | 70 | 30 | 54 | 0 | 0 | 16 | 77.14% | 77.14% |
| suspicious_url_string | 5 | 0 | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% |

- 정상 URL 오탐(FP): 4건
- 악성 URL 미탐(FN): 31건
- verdict 없음: 114건

## 3. 개선 방안

- 정상 오탐은 신뢰 도메인, 정상 리다이렉트, AI suspicious 단독 가산 여부를 우선 확인한다.
  - FP 패턴 2건: domain=DGA_LIKE, content=LOGO_ALT_IMPERSONATION, ai=suspicious
  - FP 패턴 2건: domain=TYPO_DOMAIN, content=LOGO_ALT_IMPERSONATION, ai=suspicious
- 악성 미탐은 FETCH_FAILED, NEW_DOMAIN 단독, 콘텐츠/AI 미호출 케이스를 분리해 본다.
  - FN: db_dependent openphish_006 score=25 domain=- content=EXTERNAL_LINK_OVERUSE
  - FN: db_independent openphish_006 score=25 domain=- content=EXTERNAL_LINK_OVERUSE
  - FN: db_dependent openphish_014 score=20 domain=- content=-
  - FN: db_independent openphish_014 score=20 domain=- content=-
  - FN: db_dependent openphish_015 score=20 domain=- content=-
- verdict 없는 페이지는 정확도 분모와 분리하고, Spring 콜백에는 `PAGE_UNAVAILABLE` 상태와 HTTP status를 전달한다.
  - NO_VERDICT_PIPELINE_ERROR: 114건
