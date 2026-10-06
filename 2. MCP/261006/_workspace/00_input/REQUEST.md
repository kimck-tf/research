# 요청 (2026-10-06)

> 3D FE 메시 모델링 하는 프로그램 중에서 MCP를 공식 제공하는 것이 있는지 조사해주고 사례도 찾아줘.
> 현재 나는 지멘스 Hypermesh 를 쓰고있는데 이건 공식 MCP가 없어.

## 조사 범위 정의

- **MCP**: Model Context Protocol (Anthropic 2024-11 공개, 2025-12 Linux Foundation 산하 Agentic AI Foundation 으로 이관). LLM 클라이언트(Claude, Copilot, Cursor 등)가 외부 도구를 표준 방식으로 호출하는 프로토콜
- **"공식"의 판정 기준**: 벤더 자체 채널(공식 웹사이트·문서·보도자료·공식 GitHub 조직)에서 배포/발표한 것만 공식으로 인정. 개인·제3자 저장소는 커뮤니티로 분류
- **대상 프로그램**: 3D FE 프리프로세서/메셔 (HyperMesh, ANSA, Simcenter 3D, Femap, Abaqus/CAE, Ansys Mechanical/Prime, MSC Apex/Patran, COMSOL, Cubit, Gmsh, FreeCAD FEM 등) + 메시 기능을 포함한 CAE 플랫폼
- **사례**: 벤더 데모, 산업체 적용, 학술 논문(LLM 에이전트 + CAE 도구), 커뮤니티 구현

## 사용자 환경

- 프리프로세서: HyperMesh (Altair → 2025-03 Siemens 인수 완료)
- 솔버: Abaqus (C3D8 등, 휠 충격 해석)
- 관련 연구: `../../1. GNN/` (MeshGraphNets 기반 휠 응력 예측)

## 분업 (서브 에이전트 6개, 병렬)

| # | 범위 | 산출물 |
|---|---|---|
| 01 | Ansys(Synopsys) + Cadence(BETA CAE: ANSA, Pointwise) | `01_ansys-cadence.md` |
| 02 | Siemens(Simcenter/Femap/NX/STAR-CCM+) + Dassault(Abaqus/3DEXPERIENCE) + Hexagon + Keysight(ESI) | `02_siemens-dassault-hexagon.md` |
| 03 | HyperMesh/HyperWorks 심층 (공식 여부, 커뮤니티, 자체 구축 가능성) | `03_hypermesh.md` |
| 04 | 기타 상용 (COMSOL, MathWorks, Cubit, Autodesk, PTC, SimScale, nTop, MIDAS 등) | `04_other-commercial.md` |
| 05 | 오픈소스 (Gmsh, FreeCAD, Salome, OpenFOAM 등) + MCP 레지스트리 전수 조사 | `05_opensource-registries.md` |
| 06 | 사례·학술 문헌 (논문, 산업체, 국내 사례) | `06_case-studies.md` |
