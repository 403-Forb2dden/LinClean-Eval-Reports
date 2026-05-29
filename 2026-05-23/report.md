# 2026-05-23 보안 엔진 검증 보고서

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
| db_dependent | 220 | 220 | 100.00% | 0 | 0 | 109 | 5 | 101 | 5 | 95.45% | 95.61% | 95.61% | 95.61% |
| db_independent | 220 | 220 | 100.00% | 0 | 0 | 110 | 3 | 103 | 4 | 96.82% | 97.35% | 96.49% | 96.92% |

카테고리별 결과:

### db_dependent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| custom_corpus | 5 | 5 | 0 | 4 | 0 | 1 | 0 | 100.00% | 100.00% |
| dga | 5 | 5 | 0 | 5 | 0 | 0 | 0 | 100.00% | 100.00% |
| httpbin_redirect_chain | 5 | 5 | 0 | 0 | 0 | 5 | 0 | 100.00% | 0.00% |
| normal | 100 | 100 | 0 | 0 | 5 | 95 | 0 | 95.00% | 0.00% |
| openphish_malicious | 100 | 100 | 0 | 95 | 0 | 0 | 5 | 95.00% | 95.00% |
| suspicious_url_string | 5 | 5 | 0 | 5 | 0 | 0 | 0 | 100.00% | 100.00% |

### db_independent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| custom_corpus | 5 | 5 | 0 | 4 | 0 | 1 | 0 | 100.00% | 100.00% |
| dga | 5 | 5 | 0 | 5 | 0 | 0 | 0 | 100.00% | 100.00% |
| httpbin_redirect_chain | 5 | 5 | 0 | 0 | 0 | 5 | 0 | 100.00% | 0.00% |
| normal | 100 | 100 | 0 | 0 | 3 | 97 | 0 | 97.00% | 0.00% |
| openphish_malicious | 100 | 100 | 0 | 96 | 0 | 0 | 4 | 96.00% | 96.00% |
| suspicious_url_string | 5 | 5 | 0 | 5 | 0 | 0 | 0 | 100.00% | 100.00% |

- 정상 URL 오탐(FP): 8건
- 악성 URL 미탐(FN): 9건
- verdict 없음: 0건

## 3. 개선 방안

- 정상 오탐은 신뢰 도메인, 정상 리다이렉트, AI suspicious 단독 가산 여부를 우선 확인한다.
  - FP 패턴 3건: domain=TYPO_DOMAIN, content=LOGO_ALT_IMPERSONATION, ai=suspicious
  - FP 패턴 2건: domain=DGA_LIKE, content=LOGO_ALT_IMPERSONATION, ai=suspicious
  - FP 패턴 1건: domain=-, content=PII_COLLECTION_FORM|META_REFRESH, ai=suspicious
- 악성 미탐은 FETCH_FAILED, NEW_DOMAIN 단독, 콘텐츠/AI 미호출 케이스를 분리해 본다.
  - FN: db_dependent openphish_003 score=20 domain=NO_HTTPS content=FETCH_FAILED
  - FN: db_independent openphish_003 score=20 domain=NO_HTTPS content=FETCH_FAILED
  - FN: db_dependent openphish_006 score=25 domain=- content=EXTERNAL_LINK_OVERUSE
  - FN: db_independent openphish_006 score=25 domain=- content=EXTERNAL_LINK_OVERUSE
  - FN: db_dependent openphish_009 score=25 domain=- content=EXTERNAL_LINK_OVERUSE
