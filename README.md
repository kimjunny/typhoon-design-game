# ENSO 탐험실 — 태풍 설계 게임

정적 웹 게임 모듈입니다. 사용자는 태풍 환경 카드가 강화 조건인지 약화 조건인지 먼저 모르는 상태에서 선택하고, `결과 시뮬레이션`을 실행한 뒤 실제 바람 데이터(GFS)로 흐르는 동아시아 바람 흐름 지도, 위력·대한민국 피해도·점수와 결과 풀이를 확인합니다. 흐름선이 말려드는 형태로 한반도 주변 태풍이 표현되고, 지도는 드래그로 이동·휠로 확대됩니다.

## 파일 구성

- `typhoon-game.html`: 독립 실행형 HTML 페이지
- `typhoon-game.css`: 반응형 레이아웃, 태풍 애니메이션, UI 스타일
- `typhoon-game.js`: 요인 선택, 위력 계산, 피해도/점수 계산, 실제 바람 격자(GFS) 기반 입자 흐름 애니메이션과 합성 태풍 렌더링
- `PLAN.md`: 구현 계획, 교육용 모델 공식, 검증 기록

## 실행 방법

1. 브라우저에서 `typhoon-game.html`을 엽니다.
2. 환경 카드 퀴즈에서 피해를 줄일 것 같은 조건을 고릅니다. 선택 중에는 예측 수치가 갱신되지 않습니다.
3. `결과 시뮬레이션` 버튼을 눌러 실제 바람 흐름 지도, 한반도 남쪽에 형성되는 태풍, 최종 대한민국 피해도, 점수, 선택 카드의 정답 풀이를 확인합니다.
4. `초기화` 버튼으로 선택 상태를 지울 수 있습니다.

## 수동 검증 절차

- 버튼을 클릭하면 선택 상태가 유지되고, 같은 버튼을 다시 클릭하면 해제되는지 확인합니다.
- 선택지 화면에서 강화/약화 분류나 가중치가 먼저 표시되지 않는지 확인합니다.
- 서로 다른 환경 카드를 선택해도 위력·피해도·점수는 바로 바뀌지 않고, `결과 시뮬레이션` 클릭 후에만 갱신되는지 확인합니다.
- `결과 시뮬레이션` 클릭 시 실제 바람 흐름 지도와 태풍 소용돌이, 점수, 위력, 피해도, 강풍·호우·해안 위험, 선택 카드 풀이가 표시되는지 확인합니다.
- 모바일 폭으로 줄였을 때 카드가 1열로 정리되고 버튼이 정상 클릭되는지 확인합니다.

## 근거 자료

- NOAA JetStream — Tropical Cyclone Introduction: https://www.noaa.gov/jetstream/tropical/tropical-cyclone-introduction
- NOAA/AOML — Wind shear and tropical cyclone intensity: https://www.aoml.noaa.gov/impact-of-wind-shear-on-tropical-cyclone-intensity/
- NWS — Hurricane Facts / Anatomy: https://www.weather.gov/source/zhu/ZHU_Training_Page/tropical_stuff/hurricane_anatomy/hurricane_anatomy.html
- Korea Meteorological Administration — Current Severe Weather Status: https://www.kma.go.kr/neng/forecast/warning.do
- Korea Meteorological Administration — Typhoon safety guide: https://www.weather.go.kr/w/hazard/safety-guide/typhoon.do

## 주의

이 게임의 계산식은 실제 태풍 예보 모델이 아니라 교육용 상대점수 모델입니다. 실제 기상 판단과 방재 행동에는 기상청 및 관련 기관의 공식 정보를 확인해야 합니다.
