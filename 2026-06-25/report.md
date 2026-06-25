# 2026-06-25 보안 엔진 검증 보고서

## 1. 테스트 셋 구성

- 총 URL: 100건
- 정상 기대값: 0건
- 악성 기대값: 100건
- 테스트 모드: AI 포함 `/api/v1/analyze/sync`, AI 비사용 `/api/v1/analyze/no-ai/sync`
- 출처: https://openphish.com/feed.txt

| 카테고리 | 건수 |
|---|---:|
| openphish_malicious | 100 |

## 2. 검증 결과

| API | 호출 | 판정 | Coverage | NO_VERDICT | PAGE_UNAVAILABLE | TP | FP | TN | FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| ai | 100 | 93 | 93.00% | 7 | 6 | 79 | 0 | 0 | 14 | 84.95% | 100.00% | 84.95% | 91.86% |
| no_ai | 100 | 92 | 92.00% | 8 | 7 | 58 | 0 | 0 | 34 | 63.04% | 100.00% | 63.04% | 77.33% |

카테고리별 결과:

### ai

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| openphish_malicious | 100 | 93 | 7 | 79 | 0 | 0 | 14 | 84.95% | 84.95% |

### no_ai

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| openphish_malicious | 100 | 92 | 8 | 58 | 0 | 0 | 34 | 63.04% | 63.04% |

- 정상 URL 오탐(FP): 0건
- 악성 URL 미탐(FN): 48건
- verdict 없음: 15건

## 3. 개선 방안

- 이번 결과에서는 정상 URL 오탐이 없거나 매우 낮다. 현재 보정은 유지하고 신뢰 도메인 목록 변경 시 회귀 테스트를 추가한다.
- 악성 미탐은 FETCH_FAILED, NEW_DOMAIN 단독, 콘텐츠/AI 미호출 케이스를 분리해 본다.
  - FN: no_ai openphish_001 score=- domain=- content=-
  - FN: no_ai openphish_011 score=25 domain=NEW_DOMAIN content=SPA_SHELL
  - FN: no_ai openphish_016 score=15 domain=- content=FETCH_FAILED
  - FN: ai openphish_017 score=15 domain=- content=FETCH_FAILED
  - FN: no_ai openphish_017 score=15 domain=- content=FETCH_FAILED
- verdict 없는 페이지는 정확도 분모와 분리하고, Spring 콜백에는 `PAGE_UNAVAILABLE` 상태와 HTTP status를 전달한다.
  - NO_VERDICT_PAGE_UNAVAILABLE: 13건
  - NO_VERDICT_PIPELINE_ERROR: 2건
