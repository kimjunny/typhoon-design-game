# PLAN.md — ENSO 탐험실 태풍 설계 게임

## 범위

이 산출물은 독립 실행형 정적 웹 게임입니다. 사용자는 태풍 환경 카드가 강화 조건인지 약화 조건인지 먼저 모르는 상태에서 선택하고, `결과 시뮬레이션`을 실행한 뒤 실제 바람 데이터(GFS)로 흐르는 동아시아 바람 흐름 지도, 태풍 위력·대한민국 피해도·학습 점수와 결과 풀이를 확인합니다.

구현 파일은 다음으로 한정합니다.

- `typhoon-game.html`
- `typhoon-game.css`
- `typhoon-game.js`
- `README.md`
- `PLAN.md`

## 근거 자료

- NOAA JetStream: 열대저기압 발달 조건과 연직 시어의 역할을 설명합니다. https://www.noaa.gov/jetstream/tropical/tropical-cyclone-introduction
- NOAA/AOML: 큰 연직 시어가 일반적으로 열대저기압 강도 증가에 불리하다는 점을 설명합니다. https://www.aoml.noaa.gov/impact-of-wind-shear-on-tropical-cyclone-intensity/
- NWS Hurricane Anatomy: 따뜻한 해수, 습윤한 대기, 낮은 연직 시어 등 발달 조건을 설명합니다. https://www.weather.gov/source/zhu/ZHU_Training_Page/tropical_stuff/hurricane_anatomy/hurricane_anatomy.html
- KMA 위험기상 기준: 태풍 영향이 강풍, 풍랑, 호우, 폭풍해일과 연결됨을 보여줍니다. https://www.kma.go.kr/neng/forecast/warning.do
- KMA 태풍 안전 가이드: 태풍 상황에서 대피, 외출 자제, 위험정보 공유가 필요함을 안내합니다. https://www.weather.go.kr/w/hazard/safety-guide/typhoon.do

## 게임 로직과 가중치

이 게임은 실제 예보 모델이 아니라 교육용 상대점수 모델입니다. 기본 위력은 48이며, 선택한 요인의 가중치와 상호작용 값을 더한 뒤 5~100 사이로 제한합니다.
선택 화면에서는 아래 분류와 가중치를 공개하지 않습니다. 사용자는 중립적인 환경 카드 목록에서 피해를 줄일 것 같은 조건을 고르고, 결과 시뮬레이션 후 풀이에서 정답과 영향을 확인합니다.


강화요인:

- 높은 해수면 온도: +18
- 깊은 해양 열용량: +13
- 낮은 연직 시어: +12
- 습윤한 중층 대기: +10
- 하층 수렴: +9
- 상층 발산: +8

약화요인:

- 육지 상륙: -20
- 강한 연직 시어: -18
- 건조 공기 유입: -12
- 차가운 해수·용승: -11
- 지형 마찰: -8
- 눈벽 교체 주기: -7

상호작용 규칙:

- 높은 해수면 온도 + 깊은 해양 열용량 + 낮은 연직 시어: +8, 급격한 강화 조건
- 하층 수렴 + 상층 발산: +5, 순환 환기 보너스
- 강한 연직 시어 + 건조 공기 유입: -6, 구조 붕괴 페널티
- 육지 상륙 + 지형 마찰: -7, 내륙 감쇠 페널티

## 시각화 방식

- 카드 선택 중에는 예측 수치와 태풍 시각화를 갱신하지 않고, 선택 상태와 안내문만 바꿉니다.
- `결과 시뮬레이션`을 실행하면 JavaScript가 계산한 `power`, `koreaDamage` 값을 CSS custom property와 바람 흐름 엔진의 합성 태풍 상태로 전달합니다.
- 시뮬레이션 후 바람 흐름선의 밀도·밝기, 입자 속도, 한반도 영향권 크기가 선택 조합에 따라 바뀝니다.
- 시각화는 2D 캔버스 엔진으로, 실제 GFS 바람 격자(u/v)를 이중선형 보간해 입자가 따라 흐르고 잔상을 남기는 nullschool 방식 흐름선으로 그립니다. 지도는 Natural Earth 해안선·육지 폴리곤으로 동아시아를 표현합니다.
- 태풍은 별도 도형이 아니라 합성 소용돌이(북반구 반시계 회전 + 나선형 유입 + 북동 전향)를 바람장에 더해 흐름선이 말려드는 형태로 표현합니다. `power`는 태풍 세기·반경과 흐름 속도를, `koreaDamage`는 태풍 위치(한반도 접근)를 조절합니다.
- 환경 카드 버튼은 강화/약화 분류와 가중치를 숨긴 채 `aria-pressed` 상태만 갱신해 현재 선택 여부를 표시합니다.
- 결과 화면은 실제 바람 흐름 지도와 태풍 소용돌이, 대한민국 피해도, 최종 점수, 위력, 강풍·호우·해안 위험 지표와 선택 카드별 정답 풀이를 표시합니다.

## 점수 계산 공식

1. `power = clamp(5, 100, basePower + factorWeights + synergy)`
2. `windRisk = round(power * 0.62 + selectedHighSST*8 + selectedLowShear*5 - selectedStrongShear*8 - selectedLand*7)`
3. `rainRisk = round(power * 0.45 + selectedMoistAir*12 + selectedConvergence*8 + selectedLand*4 - selectedDryAir*10)`
4. `surgeRisk = round(power * 0.36 + selectedDeepHeat*8 + selectedOutflow*5 - selectedColdWater*8)`
5. `koreaDamage = clamp(0, 100, round(0.46*windRisk + 0.34*rainRisk + 0.20*surgeRisk))`
6. `weakeningEducationBonus = min(80, numberOfWeakeningFactorsSelected * 12)`
7. `score = clamp(0, 1000, round(1000 - koreaDamage*9.2 - max(0, power-70)*1.8 + weakeningEducationBonus))`

점수는 대한민국 피해도가 낮을수록 높아집니다. 약화요인 보너스는 사용자가 소멸·약화 원리를 식별하도록 돕는 작은 교육용 보정이며, 강한 태풍의 위험성을 완전히 상쇄하지는 못합니다.

## 수동 검증 절차

1. 브라우저에서 `typhoon-game.html`을 엽니다.
2. 선택지 화면에서 강화/약화 분류나 가중치가 먼저 표시되지 않는지 확인합니다.
3. 서로 다른 환경 카드를 선택해도 태풍 위력 지수와 피해도는 바로 바뀌지 않고, `결과 시뮬레이션` 클릭 후에만 갱신되는지 확인합니다.
4. 같은 버튼을 다시 눌렀을 때 선택이 해제되고 수치가 되돌아가는지 확인합니다.
5. `결과 시뮬레이션`을 눌러 실제 바람 흐름 지도와 태풍, 점수, 위력, 피해도, 세부 위험 지표, 선택 카드 풀이가 표시되는지 확인합니다.
6. `초기화`를 눌러 선택 상태와 결과 화면이 초기화되는지 확인합니다.
7. 모바일 폭에서 카드와 버튼이 겹치지 않고 1열로 정리되는지 확인합니다.

## 검증 기록

- `node --check typhoon-game.js`: JavaScript 문법 확인.
- HTML 파서 스모크 테스트: `typhoon-game.html` 구조 확인.
- 정적 파일 체크: 필수 파일, DOM hook, 애니메이션·반응형 CSS 선택자 확인.
- 브라우저 상호작용 점검: 버튼 선택, 캔버스 흐름 애니메이션 진행, 결과 렌더링, 초기화, 데스크톱·모바일 레이아웃 확인.
