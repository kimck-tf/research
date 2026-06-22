# CLAUDE.md

## 프로젝트 개요
샤시용 **하이드로 부시(유체봉입 부시) 내구(피로) 해석 기술** 개발을 위한 사전연구·방법론·로드맵 워크스페이스.

- **위치**: `0. Research / 2. HydroBushing / 260622`
- **전제**: 고무 부시 피로수명 해석(초탄성·점탄성, CED/tearing-energy 균열발생·전파)은 사내 보유. **본 과제는 "유체봉입 구조의 동적 거동 → 고무 추가부하"를 기존 피로기준에 연결 가능한 형태로 추출하는 것.**
- **핵심 결론**: 피로 *기준*은 불변, **변형률 이력 입력단만 hydro(FSI)로 교체**. FSI 모델링(§2)과 하중추출 브릿지(§3)가 1·2순위.

## 산출물
- `하이드로부시_내구해석_방법론_및_로드맵.md` — 본 보고서(요약·7개 연구영역·gap·권고·로드맵·데이터/민감도·결정사항·문헌부록)
- `_workspace/research_provenance.md` — 리서치 출처·한계 메모

## 핵심 기술 포인트 (변경 시 주의)
- **함정#1**: Abaqus fluid cavity/link는 동해석서 유체관성 무시 → 이너셔트랙 inertance를 **`*MASS`로 부여**. 없으면 notch 주파수 틀림.
- 유체링크 점성손실은 **Direct-Solution Steady-State Dynamics(DSSD)에서만** 지원.
- 유효 bulk modulus는 용기유연성+용존공기 반영(공칭 오일 ~2000MPa 아님).
- EIE = **Efficient Interpolation Engine**. nCode 고무경로는 strain-life(사내 CED 기준엔 부적합).
- 문헌상 **FSI→CED 균열성장 전 구간 연결 선례 부재** = 기여 포인트이자 검증 자체정립 필요.

## 환경
Windows 11 / Python(uv) / Abaqus·Nastran·nCode DesignLife·Adams·HyperMesh 보유. 산출물 한글+영문 병기.
