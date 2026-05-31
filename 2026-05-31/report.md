# 2026-05-31 보안 엔진 검증 보고서

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
| db_dependent | 220 | 197 | 89.55% | 23 | 23 | 104 | 1 | 92 | 0 | 99.49% | 99.05% | 100.00% | 99.52% |
| db_independent | 220 | 197 | 89.55% | 23 | 23 | 104 | 1 | 92 | 0 | 99.49% | 99.05% | 100.00% | 99.52% |

카테고리별 결과:

### db_dependent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| custom_corpus | 5 | 5 | 0 | 4 | 1 | 0 | 0 | 80.00% | 100.00% |
| dga | 5 | 5 | 0 | 5 | 0 | 0 | 0 | 100.00% | 100.00% |
| httpbin_redirect_chain | 5 | 5 | 0 | 0 | 0 | 5 | 0 | 100.00% | 0.00% |
| normal | 100 | 87 | 13 | 0 | 0 | 87 | 0 | 100.00% | 0.00% |
| openphish_malicious | 100 | 91 | 9 | 91 | 0 | 0 | 0 | 100.00% | 100.00% |
| suspicious_url_string | 5 | 4 | 1 | 4 | 0 | 0 | 0 | 100.00% | 100.00% |

### db_independent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| custom_corpus | 5 | 5 | 0 | 4 | 1 | 0 | 0 | 80.00% | 100.00% |
| dga | 5 | 5 | 0 | 5 | 0 | 0 | 0 | 100.00% | 100.00% |
| httpbin_redirect_chain | 5 | 5 | 0 | 0 | 0 | 5 | 0 | 100.00% | 0.00% |
| normal | 100 | 87 | 13 | 0 | 0 | 87 | 0 | 100.00% | 0.00% |
| openphish_malicious | 100 | 91 | 9 | 91 | 0 | 0 | 0 | 100.00% | 100.00% |
| suspicious_url_string | 5 | 4 | 1 | 4 | 0 | 0 | 0 | 100.00% | 100.00% |

- 정상 URL 오탐(FP): 2건
- 악성 URL 미탐(FN): 0건
- verdict 없음: 46건

## 3. 개선 방안

- 정상 오탐은 신뢰 도메인, 정상 리다이렉트, AI suspicious 단독 가산 여부를 우선 확인한다.
  - FP 패턴 2건: domain=IP_DIRECT, content=FETCH_FAILED, ai=-
- 악성 URL 미탐이 없으면 OpenPhish 수집 시각과 원본 URL 스냅샷을 함께 보관해 재현성을 높인다.
- verdict 없는 페이지는 정확도 분모와 분리하고, Spring 콜백에는 `PAGE_UNAVAILABLE` 상태와 HTTP status를 전달한다.
  - NO_VERDICT_PAGE_UNAVAILABLE: 46건
