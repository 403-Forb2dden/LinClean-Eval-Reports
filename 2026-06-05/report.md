# 2026-06-05 보안 엔진 검증 보고서

## 1. 테스트 셋 구성

- 총 URL: 100건
- 정상 기대값: 99건
- 악성 기대값: 1건

| 카테고리 | 건수 |
|---|---:|
| openphish_malicious | 1 |
| public_web_safe | 99 |

분포 근거:

- APWG `Phishing Activity Trends Report Q1 2026`: 2026년 1분기 unique phishing Web sites/attacks 971,181건.
- Netcraft `April 2026 Web Server Survey`: 전체 사이트 1,433,742,238개.
- 악성 비율 계산: `971,181 / 1,433,742,238 = 0.06774%`.
- 100건 테스트 셋 환산 시 `round(100 * 0.0006774) = 0`건이지만, 악성 탐지 경로를 검증하기 위해 최소 악성 1건 규칙을 적용했다.
- 최종 구성은 정상 99건, 악성 1건이다.
- 출처: https://docs.apwg.org/reports/apwg_trends_report_q1_2026.pdf, https://www.netcraft.com/blog/april-2026-web-server-survey

계산 기준:

- DB 포함 파이프라인 API(`/api/v1/analyze/sync`)만 사용했다.
- Accuracy, Precision, Recall, F1은 `NO_VERDICT`와 `PAGE_UNAVAILABLE`을 제외한 판정 가능 건(`TP`, `FP`, `TN`, `FN`)만 분모로 계산했다.

## 2. 검증 결과

| API | 호출 | 판정 | Coverage | NO_VERDICT | PAGE_UNAVAILABLE | TP | FP | TN | FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| db_dependent | 100 | 86 | 86.00% | 14 | 14 | 1 | 0 | 85 | 0 | 100.00% | 100.00% | 100.00% | 100.00% |

카테고리별 결과:

### db_dependent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| openphish_malicious | 1 | 1 | 0 | 1 | 0 | 0 | 0 | 100.00% | 100.00% |
| public_web_safe | 99 | 85 | 14 | 0 | 0 | 85 | 0 | 100.00% | 0.00% |

- 정상 URL 오탐(FP): 0건
- 악성 URL 미탐(FN): 0건
- verdict 없음: 14건

## 3. 개선 방안

- 이번 결과에서는 정상 URL 오탐이 없거나 매우 낮다. 현재 보정은 유지하고 신뢰 도메인 목록 변경 시 회귀 테스트를 추가한다.
- 악성 URL 미탐이 없으면 OpenPhish 수집 시각과 원본 URL 스냅샷을 함께 보관해 재현성을 높인다.
- verdict 없는 페이지는 정확도 분모와 분리하고, Spring 콜백에는 `PAGE_UNAVAILABLE` 상태와 HTTP status를 전달한다.
  - NO_VERDICT_PAGE_UNAVAILABLE: 14건
