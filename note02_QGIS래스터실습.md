# QGIS 래스터 데이터 실습 노트

QGIS를 활용한 래스터 데이터 분석 및 플러그인 실습 자료입니다.

## 목차

- [ex01. 래스터 데이터 다루기](#ex01-래스터-데이터-다루기)
- [ex02. 구역 통계 (Zonal Statistics)](#ex02-구역-통계-zonal-statistics)
- [ex03. 래스터 계산기 (Raster Calculator)](#ex03-래스터-계산기-raster-calculator)
- [ex04. 조건에 맞는 지역 찾기](#ex04-조건에-맞는-지역-찾기)
- [ex05. 지형 분석](#ex05-지형-분석)
- [ex06. 홍수취약지역 분석](#ex06-홍수취약지역-분석)
- [ex07. 위성영상 조합하기](#ex07-위성영상-조합하기)
- [ex08. 식생지수 구하기](#ex08-식생지수-구하기)
- [ex09. 플러그인: Time Manager 시계열 모니터링](#ex09-플러그인-time-manager-시계열-모니터링)
- [ex10. 플러그인: 종단면 분석](#ex10-플러그인-종단면-분석)
- [ex11. 플러그인: 온라인 라우팅](#ex11-플러그인-온라인-라우팅)
- [ex12. 커스텀 스크립트: Create WindRose Maps](#ex12-커스텀-스크립트-create-windrose-maps)

---

## ex01. 래스터 데이터 다루기

1. `dem30.tif` (서울시 고도 데이터) 불러오기
2. 지도 확대 > 픽셀/그리드 확인
3. 피처 확인 / 속성 조회
4. 래스터 > 분석 > **음영기복도**
   - 입력 레이어: `dem30` > 실행
5. `dem30` 레이어 > 속성 > 심볼
   - 단일 밴드 유사색상
   - 투명도: `40%`
6. `landsat.tif` 불러오기 > 레이어 순서 가장 아래로

---

## ex02. 구역 통계 (Zonal Statistics)

1. `admin_emd.shp`, `dem30.tif` 불러오기
2. 공간 처리 툴박스 > **Zonal Statistics(구역 통계)**
   - 래스터 레이어: `dem30.tif`
   - 구역을 담고 있는 벡터 레이어: `admin_emd.shp`
   - 출력 열 접두어: `alti_`
   - 계산할 통계: 개수, 합, 평균 등 선택
   - 실행 > 속성 테이블 확인
3. 산출물 레이어 > 속성 > 심볼
   - 단계 구분 > 색상표 > 모드 > 분류 > 범주 > 확인

---

## ex03. 래스터 계산기 (Raster Calculator)

1. `dem30.tif` 불러오기
2. `dem30` 레이어 > 우클릭 > 내보내기 > 다른 이름으로 저장
   - 파일 이름: `dem50.tif`
   - 해상도: `50 by 50`
3. 래스터 > **래스터 계산기**
   - 표현식: `"dem30@1" - "dem50@1"` > 확인
4. 산출물 레이어 > 속성 > 심볼
   - 단일 밴드 유사색상 > 확인

---

## ex04. 조건에 맞는 지역 찾기
**목표: 래스터 계산기를 활용하여 고도 40m 이하, 경사 5도 이하 지역 찾기**

1. 경로: `/data/raster/flood`
2. `dem30.tif`, `slope.tif` 불러오기
3. 래스터 > **래스터 계산기**
   - 표현식: `"dem30@1" <= 40 AND "slope@1" <= 5`
4. 산출물 레이어 속성 > 심볼 > 단일 밴드 유사색상

---

## ex05. 지형 분석
**목표: DEM 데이터를 활용하여 경사도 레이어를 만들기**

1. `dem30.tif` 불러오기
2. 래스터 > 분석 > **경사(Slope)**
   - 입력 레이어: `dem30` > 실행
3. 래스터 > 분석 > **경사 방향(Aspect)**
   - 입력 레이어: `dem30` > 실행
4. 경사 방향 레이어 > 속성 > 심볼 > 단일 밴드 유사색상

---

## ex06. 홍수취약지역 분석

1. 홍수 관련 폴더 레이어 불러오기
2. 래스터 > **래스터 계산기**
   ```
   ( "D_MAX_PRE@1" >= 254 ) + ( "dem30@1" <= 10 ) + ( "slope@1" <= 5 ) + ( "ratio@1" >= 40 ) + ( "pop_den@1" >= 41000 )
   ```
3. 산출물 레이어 속성 > 심볼 > 단일 밴드 유사색상

---

## ex07. 위성영상 조합하기

1. `LC~B2.tif`, `LC~B3.tif`, `LC~B4.tif` 불러오기
2. 래스터 > 기타(Miscellaneous) > **병합**
   - 각 입력 파일을 개별 밴드로 배치
   - 입력 레이어: `LC~B2.tif`, `LC~B3.tif`, `LC~B4.tif`
3. 병합 산출물 레이어 > 속성 > 심볼 > 밴드: `3, 2, 1`

---

## ex08. 식생지수 구하기

1. `LC~B5.tif`, `LC~B4.tif` 불러오기
2. 래스터 > **래스터 계산기**
   ```
   ( "LC09_L1TP_116034_20250605_20250605_02_T1_B5@1" - "LC09_L1TP_116034_20250605_20250605_02_T1_B4@1" ) / ( "LC09_L1TP_116034_20250605_20250605_02_T1_B5@1" + "LC09_L1TP_116034_20250605_20250605_02_T1_B4@1" )
   ```
   > NDVI(정규식생지수) 계산 공식입니다.
3. 산출물 레이어 > 속성 > 심볼 > 단일 밴드 유사색상
4. 확인 사항
   - 최대값 확인
   - 색상표: 녹색 계열 적용

---

## ex09. 플러그인: Time Manager 시계열 모니터링

1. 플러그인 > 플러그인 관리 및 설치 > `TimeManager` 검색 > 설치
2. 플러그인 > **Time Manager** > Toggle visibility
3. Add Layer 설정
   - Layer: `senior_att`
   - Start time: `time`
   - End time: `No end time – accumulate features`
4. 재생하여 시계열 확인

---

## ex10. 플러그인: 종단면 분석

1. Plugins > Manage and Install Plugins… > `Profile tool` 검색 > 설치
2. Plugins > **Profile Tool** > Terrain Profile
3. `dem30.tif` 불러오기 > Add Layer > 클릭 & 클릭 & 클릭 (단면 경로 지정)
4. `road_link2.shp` 불러오기
   - Selection: `Selected polyline`
   - 라인 선택 후 종단면 확인

---

## ex11. 플러그인: 온라인 라우팅

1. 플러그인 > 플러그인 관리 및 설치 > `Online Routing Mapper` 검색 > 설치
2. 플러그인 > **Online Routing Mapper**
   - API 선택
   - 출발지 / 도착지 지정

---

## ex12. 커스텀 스크립트: Create WindRose Maps

1. 공간 처리 툴박스 > 스크립트 도구 > **스크립트를 툴박스에 추가하기**
   - `CreateWindRoseMaps.py` 불러오기
2. 공간 처리 툴박스 > 스크립트 > **Create WindRose Maps**
3. Input point: `subway_station.shp`

---

*본 문서는 QGIS 래스터 데이터 분석 실습 교육 자료를 마크다운 형식으로 정리한 것입니다.*
