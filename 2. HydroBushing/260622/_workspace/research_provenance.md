# 리서치 출처·방법 메모 (audit trail)

**작성일**: 2026-06-22

본 폴더의 보고서(`하이드로부시_내구해석_방법론_및_로드맵.md`)는 3개 병렬 리서치 스레드(WebSearch/WebFetch 기반) + 도메인 종합으로 작성됨.

## 병렬 리서치 스레드
1. **하이드로 부시 작동원리·손상 메커니즘**: 펌핑/notch 물리, lumped 파라미터, 고장모드, 캐비테이션. (Adiguna/Singh, OSU-ADL, SAE 2013-01-1924, 특허군)
2. **Abaqus fluid cavity FSI**: `*FLUID CAVITY`(DOF8/PCAV/CVOL), `*FLUID EXCHANGE`(BULK VISCOSITY/ORIFICE), **유체관성→`*MASS` 우회**, **DSSD 전용 점성손실**, 캐비테이션 압력하한, co-sim. (Abaqus 문서, CJME 2021, IJAT 2010)
3. **Endurica/피로 브릿지**: CL/DT/**EIE(Efficient Interpolation Engine)**, fe-safe/Rubber(.odb 직결), nCode 고무=strain-life(CED 미지원), Mars CED/Lake-Lindley, Barbash&Mars SAE 2016-01-0393, RLDA/Adams. (Endurica, Mars&Fatemi, SAE)

## 신뢰도·한계 caveat
- 다수 출판사(ScienceDirect/SAE/Springer/SAGE) 및 Abaqus 문서 미러가 직접 fetch(HTTP 403) 차단됨 → 검색엔진 추출 기반. **인용 전 원문 DOI/페이지 재확인 필요.**
- 저신뢰 재확인 항목: "Fatigue HUD"·"find it/fill it/check it" 명칭, nCode 고무기준 정확형, EIE ">10,000×" 벤치마크, Abaqus hydraulic-fluid 키워드 세부구문.

## 핵심 공백(문헌상 선례 부재 → 사내 자체정립 필요)
1. hydro vs dry 부시 수명 정량 비교
2. 고무 챔버 캐비테이션 침식 전용 피로연구
3. FSI 동하중 → CED/임계면 균열성장 전 구간 연결 사례
