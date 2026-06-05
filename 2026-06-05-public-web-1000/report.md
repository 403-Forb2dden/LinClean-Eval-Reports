# 2026-06-05 보안 엔진 검증 보고서

## 1. 테스트 셋 구성

- 총 URL: 1000건
- 정상 기대값: 999건
- 악성 기대값: 1건

| 카테고리 | 건수 |
|---|---:|
| openphish_malicious | 1 |
| public_web_safe | 999 |

분포 근거:

- APWG `Phishing Activity Trends Report Q1 2026`: 2026년 1분기 unique phishing Web sites/attacks 971,181건.
- Netcraft `April 2026 Web Server Survey`: 전체 사이트 1,433,742,238개.
- 악성 비율 계산: `971,181 / 1,433,742,238 = 0.06774%`.
- 1000건 테스트 셋 환산: `round(1000 * 0.0006774) = 1`건 악성, 나머지 999건 정상.
- 출처: https://docs.apwg.org/reports/apwg_trends_report_q1_2026.pdf, https://www.netcraft.com/blog/april-2026-web-server-survey

## 2. 검증 결과

| API | 호출 | 판정 | Coverage | NO_VERDICT | PAGE_UNAVAILABLE | TP | FP | TN | FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| db_dependent | 1000 | 609 | 60.90% | 391 | 367 | 1 | 119 | 489 | 0 | 80.46% | 0.83% | 100.00% | 1.65% |
| db_independent | 1000 | 506 | 50.60% | 494 | 267 | 1 | 100 | 405 | 0 | 80.24% | 0.99% | 100.00% | 1.96% |

카테고리별 결과:

### db_dependent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| openphish_malicious | 1 | 1 | 0 | 1 | 0 | 0 | 0 | 100.00% | 100.00% |
| public_web_safe | 999 | 608 | 391 | 0 | 119 | 489 | 0 | 80.43% | 0.00% |

### db_independent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| openphish_malicious | 1 | 1 | 0 | 1 | 0 | 0 | 0 | 100.00% | 100.00% |
| public_web_safe | 999 | 505 | 494 | 0 | 100 | 405 | 0 | 80.20% | 0.00% |

- 정상 URL 오탐(FP): 219건
- 악성 URL 미탐(FN): 0건
- verdict 없음: 885건

## 3. 개선 방안

- 정상 오탐은 신뢰 도메인, 정상 리다이렉트, AI suspicious 단독 가산 여부를 우선 확인한다.
  - FP 패턴 36건: domain=REDIRECT_CROSS_ORIGIN, content=-, ai=suspicious
  - FP 패턴 28건: domain=DGA_LIKE, content=FETCH_FAILED, ai=-
  - FP 패턴 12건: domain=REDIRECT_CROSS_ORIGIN, content=LOGO_ALT_IMPERSONATION, ai=suspicious
- 악성 URL 미탐이 없으면 OpenPhish 수집 시각과 원본 URL 스냅샷을 함께 보관해 재현성을 높인다.
- verdict 없는 페이지는 정확도 분모와 분리하고, Spring 콜백에는 `PAGE_UNAVAILABLE` 상태와 HTTP status를 전달한다.
  - NO_VERDICT_PAGE_UNAVAILABLE: 634건
  - NO_VERDICT_REQUEST_FAILED: 251건
