# 2026-06-05 보안 엔진 검증 보고서

## 1. 테스트 셋 구성

- 총 URL: 100건
- 정상 기대값: 0건
- 악성 기대값: 100건

| 카테고리 | 건수 |
|---|---:|
| openphish_malicious | 100 |

구성 근거:

- OpenPhish 최신 피드에서 수집한 악성 URL 100건만 포함한다.
- 1000건 공용 웹 분포 테스트에서 악성 표본이 1건뿐이므로, 악성 탐지 회귀 성능을 별도로 확인하기 위한 보조 테스트다.

계산 기준:

- 이번 재평가는 DB 포함 파이프라인 API(`/api/v1/analyze/sync`)만 사용했다.
- Accuracy, Precision, Recall, F1은 `NO_VERDICT`와 `PAGE_UNAVAILABLE`을 제외한 판정 가능 건(`TP`, `FP`, `TN`, `FN`)만 분모로 계산했다.

## 2. 검증 결과

| API | 호출 | 판정 | Coverage | NO_VERDICT | PAGE_UNAVAILABLE | TP | FP | TN | FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| db_dependent | 100 | 94 | 94.00% | 6 | 6 | 73 | 0 | 0 | 21 | 77.66% | 100.00% | 77.66% | 87.43% |

카테고리별 결과:

### db_dependent

| 카테고리 | 호출 | 판정 | NO_VERDICT | TP | FP | TN | FN | Accuracy | Recall |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| openphish_malicious | 100 | 94 | 6 | 73 | 0 | 0 | 21 | 77.66% | 77.66% |

- 정상 URL 오탐(FP): 0건
- 악성 URL 미탐(FN): 21건
- verdict 없음: 6건

## 3. 개선 방안

- 이번 결과에서는 정상 URL 오탐이 없거나 매우 낮다. 현재 보정은 유지하고 신뢰 도메인 목록 변경 시 회귀 테스트를 추가한다.
- 악성 미탐은 FETCH_FAILED, NEW_DOMAIN 단독, 콘텐츠/AI 미호출 케이스를 분리해 본다.
  - FN: db_dependent openphish_006 score=- domain=- content=-
  - FN: db_dependent openphish_024 score=15 domain=- content=FETCH_FAILED
  - FN: db_dependent openphish_025 score=15 domain=- content=FETCH_FAILED
  - FN: db_dependent openphish_026 score=15 domain=- content=FETCH_FAILED
  - FN: db_dependent openphish_027 score=15 domain=- content=FETCH_FAILED
- verdict 없는 페이지는 정확도 분모와 분리하고, Spring 콜백에는 `PAGE_UNAVAILABLE` 상태와 HTTP status를 전달한다.
  - NO_VERDICT_PAGE_UNAVAILABLE: 6건
