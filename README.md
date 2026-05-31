# LinClean Eval Reports

LinClean FastAPI URL 보안 엔진의 날짜별 평가 결과 저장소입니다. 각 날짜 디렉터리는 같은 구조를 유지합니다.

```text
YYYY-MM-DD/
  dataset.csv   # 평가 입력 URL과 기대 라벨
  results.csv   # db_dependent, db_independent 실행 결과
  report.md     # 테스트 셋 구성, 검증 결과, 개선 방안 요약
  report.xlsx   # 한글 Excel 요약, 상세 결과, AI 답변 정리
```

## 최신 결과

기준일: **2026-05-31**

- 총 URL: 220건
- 정상 기대값: 106건
- 악성 기대값: 114건
- 정상 URL 오탐(FP): 2건
- 악성 URL 미탐(FN): 0건
- verdict 없음: 46건
- verdict 없음 원인: `NO_VERDICT_PAGE_UNAVAILABLE` 46건
- Excel 리포트: `2026-05-31/report.xlsx`

| API | 호출 | 판정 | Coverage | NO_VERDICT | PAGE_UNAVAILABLE | TP | FP | TN | FN | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| db_dependent | 220 | 197 | 89.55% | 23 | 23 | 104 | 1 | 92 | 0 | 99.49% | 99.05% | 100.00% | 99.52% |
| db_independent | 220 | 197 | 89.55% | 23 | 23 | 104 | 1 | 92 | 0 | 99.49% | 99.05% | 100.00% | 99.52% |

## 대표 지표

전체 악성 기준 탐지율 : 91.23%

- 계산식: `탐지 성공 104건 / 전체 악성 기대값 114건`
- OpenPhish 최신 URL과 자체 악성 샘플을 모두 포함해, 실제 악성으로 준비한 전체 테스트셋 중 몇 건을 탐지 판정까지 연결했는지 보여줍니다.
- 사라진 페이지나 400번대 접근 실패처럼 `PAGE_UNAVAILABLE`로 조기 종료된 악성 URL도 전체 악성 기준 분모에 포함합니다.
- 참고: 미탐(FN)은 0건이고, 탐지하지 못한 10건은 악성으로 오판한 것이 아니라 페이지 미발견/접근 불가로 verdict 없이 종료된 케이스입니다.

| 구분 | 값 | 설명 |
|---|---:|---|
| 전체 악성 기준 탐지율 | 91.23% | 전체 악성 114건 중 104건 탐지 |
| 판정 가능 악성 기준 Recall | 100.00% | PAGE_UNAVAILABLE 제외 후 악성 미탐 0건 |
| Accuracy | 99.49% | 판정 가능한 정상/악성 전체 중 맞춘 비율. 탐지율과는 다른 지표 |
| Precision | 99.05% | 악성이라고 판정한 것 중 실제 악성 비율 |
| 정상 URL 오탐 | 2건 | custom_corpus 정상 1건이 양쪽 API에서 caution |
| PAGE_UNAVAILABLE | 46건 | 전체 미판정 건수. 이 중 악성 기준 10건 |

## 날짜별 추이

| 날짜 | 정상 오탐(FP) | 악성 미탐(FN) | verdict 없음 | 요약 |
|---|---:|---:|---:|---|
| 2026-05-23 | 8 | 9 | 0 | 초기 기준선. 전체 coverage 100%, 정상 URL 오탐 존재. |
| 2026-05-24 | 9 | 21 | 0 | 정상 오탐과 악성 미탐이 모두 증가. |
| 2026-05-25 | - | - | - | 결과 CSV 없음. 데이터셋 기준선만 보관. |
| 2026-05-26 | - | - | - | 결과 CSV 없음. 데이터셋 기준선만 보관. |
| 2026-05-27 | 4 | 31 | 114 | 파이프라인 오류/미판정 분리 전 결과. |
| 2026-05-28 | 7 | 59 | 79 | PAGE_UNAVAILABLE 분리 시작, 정상 오탐 재발. |
| 2026-05-29 | 0 | 10 | 70 | 정상 오탐 0건, precision 100%, page unavailable은 verdict 없이 분리. |
| 2026-05-30 | 2 | 0 | 39 | OpenPhish 최신화 후 판정 가능 악성 recall 100%, custom_corpus 정상 1건이 양쪽 API에서 오탐. |
| 2026-05-31 | 2 | 0 | 46 | OpenPhish 최신화 후 판정 가능 악성 recall 100%, AI 답변 요약 포함 Excel 리포트 추가. |

## 해석 기준

- `caution`과 `danger`는 탐지 양성으로 계산합니다.
- 찾을 수 없는 페이지, 400번대 접근 실패, 사라진 OpenPhish URL은 verdict를 만들지 않고 `PAGE_UNAVAILABLE`로 분리합니다.
- `NO_VERDICT_PAGE_UNAVAILABLE`은 정확도 계산 분모에서 제외하고, Spring 콜백에는 실패 상태와 HTTP status code를 전달하는 것이 현재 정책입니다.
- 2026-05-31 기준 개선의 핵심은 판정 가능한 악성 URL 미탐을 0건으로 유지한 점입니다. 다만 custom_corpus 정상 URL 1건이 양쪽 API에서 caution으로 오탐되어, IP 직접 접근/콘텐츠 fetch 실패 조합의 정상 예외 처리가 다음 개선 대상입니다.

## 테스트 셋 구성

2026-05-31 기준 데이터셋 구성은 다음과 같습니다.

| 카테고리 | 건수 | 설명 |
|---|---:|---|
| normal | 100 | 정상 URL 샘플 |
| openphish_malicious | 100 | OpenPhish 기반 악성 URL 샘플 |
| custom_corpus | 5 | 자체 수집 회귀 샘플 |
| dga | 5 | DGA 유사 도메인 샘플 |
| httpbin_redirect_chain | 5 | 정상 리다이렉트 체인 샘플 |
| suspicious_url_string | 5 | 문자열 기반 의심 URL 샘플 |

## 파일 관리 원칙

- 날짜별 디렉터리에는 기본적으로 `dataset.csv`, `results.csv`, `report.md`를 두고, 사람이 검토하기 쉬운 Excel 리포트가 필요한 경우 `report.xlsx`를 함께 둡니다.
- 임시 summary, 로그, 실행 중 산출물은 커밋하지 않습니다.
- 외부 URL 상태는 시간이 지나며 바뀌므로, 날짜별 결과는 같은 날짜의 데이터셋과 함께 해석해야 합니다.
