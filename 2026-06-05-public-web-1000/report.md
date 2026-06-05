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

계산 기준:

- 이번 재평가는 DB 포함 파이프라인 API(`/api/v1/analyze/sync`)만 사용했다.
- Accuracy, Precision, Recall, F1은 `NO_VERDICT`와 `PAGE_UNAVAILABLE`을 제외한 판정 가능 건(`TP`, `FP`, `TN`, `FN`)만 분모로 계산했다.

## 2. 검증 결과

| API | 호출 | 판정 | Coverage | NO_VERDICT | PAGE_UNAVAILABLE | TP | FP | TN | FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| db_dependent | 1000 | 626 | 62.60% | 374 | 374 | 1 | 122 | 503 | 0 | 80.51% | 0.81% | 100.00% | 1.61% |

카테고리별 결과:

### db_dependent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| openphish_malicious | 1 | 1 | 0 | 1 | 0 | 0 | 0 | 100.00% | 100.00% |
| public_web_safe | 999 | 625 | 374 | 0 | 122 | 503 | 0 | 80.48% | 0.00% |

- 정상 URL 오탐(FP): 122건
- 악성 URL 미탐(FN): 0건
- verdict 없음: 374건

## 3. 개선 방안

- 정상 오탐은 신뢰 도메인, 정상 리다이렉트, AI suspicious 단독 가산 여부를 우선 확인한다.
  - FP 패턴 17건: domain=DGA_LIKE, content=FETCH_FAILED, ai=-
  - FP 패턴 17건: domain=REDIRECT_CROSS_ORIGIN, content=-, ai=suspicious
  - FP 패턴 8건: domain=REDIRECT_CROSS_ORIGIN, content=LOGO_ALT_IMPERSONATION, ai=suspicious
- 악성 URL 미탐이 없으면 OpenPhish 수집 시각과 원본 URL 스냅샷을 함께 보관해 재현성을 높인다.
- verdict 없는 페이지는 정확도 분모와 분리하고, Spring 콜백에는 `PAGE_UNAVAILABLE` 상태와 HTTP status를 전달한다.
  - NO_VERDICT_PAGE_UNAVAILABLE: 374건
