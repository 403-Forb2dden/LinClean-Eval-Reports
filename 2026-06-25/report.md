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

탐지율은 대상 페이지가 응답하지 않았거나 콘텐츠 수집이 실패한 URL을 분모에서 제외해 계산했다.

| API | 호출 | 평가 대상 | 응답 불가 제외 | 기타 제외 | TP | FP | TN | FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| ai | 100 | 65 | 34 | 1 | 62 | 0 | 0 | 3 | 95.38% | 100.00% | 95.38% | 97.64% |
| no_ai | 100 | 68 | 31 | 1 | 44 | 0 | 0 | 24 | 64.71% | 100.00% | 64.71% | 78.57% |

카테고리별 결과:

### ai

| 카테고리 | 호출 | 평가 대상 | 응답 불가 제외 | 기타 제외 | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| openphish_malicious | 100 | 65 | 34 | 1 | 62 | 0 | 0 | 3 | 95.38% | 95.38% |

### no_ai

| 카테고리 | 호출 | 평가 대상 | 응답 불가 제외 | 기타 제외 | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| openphish_malicious | 100 | 68 | 31 | 1 | 44 | 0 | 0 | 24 | 64.71% | 64.71% |

- 정상 URL 오탐(FP): 0건
- 평가 대상 내 악성 URL 미탐(FN): 27건
- 응답 불가로 평가 제외: 65건
- 기타 오류로 평가 제외: 2건

## 3. 개선 방안

- 이번 OpenPhish 100건에서는 AI 사용 시 평가 가능한 URL 기준 Recall이 95.38%로, no-AI의 64.71%보다 30.67%p 높았다. no-AI 경로는 운영 비용과 장애 격리에는 유리하지만, 현재 신호 조합만으로는 악성 판단 임계값에 도달하지 못하는 케이스가 많다.
- no-AI 미탐 24건 중 다수는 `EXTERNAL_LINK_OVERUSE`, `SPA_SHELL`, `PII_COLLECTION_FORM`, `LOGO_ALT_IMPERSONATION`, `NEW_DOMAIN`, `NO_HTTPS`, `BRAND_IN_URL`처럼 단일 또는 약한 신호만 잡힌 케이스다. no-AI summary를 유지하려면 이 조합들에 대한 점수 보정과 위험 문구 템플릿을 먼저 보강한다.
- AI 경로의 미탐 3건은 AI가 `benign`으로 판단한 케이스다. OpenPhish 최신 피드처럼 악성 정답이 확실한 데이터에서는 AI benign 판정 전후의 domain/content 신호를 같이 남겨 AI override 또는 재검사 조건을 설계한다.
- 응답 불가 URL은 탐지 성능 분모에서 제외하는 것이 맞다. 대신 별도 품질 지표로 관리해 fetch timeout, 페이지 접근 차단, `PAGE_UNAVAILABLE`, `NO_VERDICT_PIPELINE_ERROR`를 구분해서 수집 안정성을 개선한다.
- 이번 셋은 전부 악성 URL이므로 Precision과 FP 평가는 제한적이다. no-AI 임계값을 낮추기 전에는 정상 URL 세트를 함께 넣어 오탐 증가 여부를 별도로 확인한다.
