# 3D FE 메시 프로그램의 공식 MCP 지원 현황 및 활용 사례 조사 보고서

**작성일**: 2026-10-06
**요청**: 3D FE 메시 모델링 프로그램 중 MCP(Model Context Protocol)를 공식 제공하는 것이 있는지 조사하고 사례도 찾아 달라는 요청. 현재 사용 중인 (Simcenter) HyperMesh에는 공식 MCP가 없음
**작성 체계**: 6개 분야 병렬 웹 조사(`_workspace/01~06`) + 핵심 주장 1차 출처 재검증. 대상은 공식 여부, 날짜, 수치이며 GitHub·PyPI 원문과 벤더 보도자료로 확인
**검증 한계**: 조사 환경의 네트워크 정책 때문에 다수 벤더 도메인을 직접 열람할 수 없었다(ansys.com, siemens.com, altair.com, 3ds.com, comsol.com, beta-cae.com, simscale.com, engineering.com 등). 이 도메인의 내용은 **검색엔진 색인 요약**으로 확인했다. GitHub·PyPI·MCP Registry는 **원문을 직접 확인**했다. 표에서는 아래 기호로 구분한다.
- ✅ 원문 직접 확인
- 🔎 검색 색인 요약으로 확인
- 📄 서브 에이전트 조사 결과(근거 URL은 `_workspace/` 원본에 수록)

---

## Executive Summary

### 한 줄 결론

> **2026-10 기준, 3D FE 메시를 다루는 전용 CAE 툴 중 공식 MCP 서버를 실제로 배포한 곳은 Ansys(Synopsys)뿐이며 아직 알파 단계다. COMSOL은 2026-09-16에 공식 MCP 서버를 발표했고 Version 2027(올가을)에 탑재한다.**
>
> **자동차 프리프로세서 주력 툴인 HyperMesh, ANSA, Simcenter 3D/Femap, Abaqus/CAE, MSC Apex/Patran에는 공식 MCP가 없다.** 벤더들은 MCP 대신 **제품 내장 AI 어시스턴트/에이전트**(Copilot, AI Assistant, LEO, Mesh Agent)를 먼저 내놓았다. 공백은 **커뮤니티 MCP**가 채우고 있으며 2026년 5~10월에 급증했다. HyperMesh용 6종 이상, Abaqus용 20종 이상이 확인됐다.

### 공식 MCP 현황 요약표

| 구분 | 프로그램 (벤더) | 공식 MCP | 상태 (2026-10-06) | 메싱 관련 기능 | 검증 |
|---|---|---|---|---|---|
| **✅ 공식 배포** | Ansys Mechanical (Synopsys) | PyMechanical-MCP | Alpha 0.2.1 (2026-09-17) | 전용 메시 도구는 없음. Python 스크립트 실행 + `meshing` 가이드라인 도구 | ✅ |
| | Ansys MAPDL | PyMAPDL-MCP | Alpha 0.3.1 (2026-09-28) | 임의 APDL 명령 실행 (ESIZE/VMESH 등) | ✅ |
| | Ansys Fluent (Fluent Meshing 포함) | PyFluent-MCP | Alpha 0.5.0 (2026-09-25) | Watertight/Fault-tolerant 메싱 워크플로, `mesh_quality` 도구 | ✅ |
| **✅ 공식 (범용 플랫폼)** | MATLAB + PDE Toolbox (MathWorks) | MATLAB MCP Server | 0.x, 최신 v0.14.0 (2026-09-25) 📄 | MATLAB 코드 실행 방식으로 3D 사면체 메시(`generateMesh`)·해석 가능 | ✅ |
| **📢 공식 발표** | COMSOL Multiphysics | COMSOL MCP Server | Version 2027에 탑재, "올가을" 출시 예정 (발표 2026-09-16) | 모델 생성·수정, 해석, 결과 확인 (도구 목록 미공개) | 🔎 |
| **❌ 없음** (AI 어시스턴트만) | **Simcenter HyperMesh** (Siemens, 구 Altair) | — | Copilot: 도움말 Q&A, Python 스크립트 생성(베타). MCP 아님 | — | 🔎📄 |
| | ANSA / META (Cadence, 구 BETA CAE) | — | AI Assistant → 2026.1에서 "Agentic AI" 확대, on-prem LLM 지원 | — | 🔎 |
| | Simcenter 3D / Femap / NX / STAR-CCM+ (Siemens) | — | Simcenter Copilot (도움말 기반) | — | 🔎📄 |
| | Abaqus/CAE, 3DEXPERIENCE SIMULIA (Dassault) | — | 가상 동반자 LEO (2026-07-23 출시, 플랫폼 내부 전용) | — | 🔎 |
| | MSC Apex / Patran / Marc (Hexagon), Visual-Mesh (Keysight·ESI) | — | — | — | 📄 |
| | Ansys Prime(PyPrimeMesh), SpaceClaim/Discovery, LS-PrePost | — | 커뮤니티 MCP만 있음. 참고로 Mechanical의 **Mesh Agent**(2026 R1)는 제품 내장 기능이며 MCP가 아님 | — | ✅🔎 |
| | Coreform Cubit, Onshape/SimScale, Autodesk Fusion Simulation, MIDAS | — | SimScale은 Onshape 내장 AI 에이전트(2026-09), Fusion은 API 스크립트 실행 **샘플** MCP만 | — | 🔎📄 |
| | Gmsh, FreeCAD, Salome, CalculiX, OpenFOAM 등 오픈소스 | — (프로젝트 공식 MCP 없음) | 커뮤니티 MCP 다수(§2.8). 공식 조직의 MCP는 Kitware `vtk-mcp`(API 지식)처럼 메싱과 무관한 것뿐 | — | ✅📄 |

### HyperMesh 사용자 관점 핵심 시사점

1. **"HyperMesh 공식 MCP 없음"은 사실이다.** 2024~2026 릴리스 노트, HyperWorks 2026 발표(2025-12-08), Simcenter HyperMesh 2026.1, 통합 Simcenter 릴리스(2026-07-28), Altair 공식 GitHub 어디에도 MCP가 없다.
2. **다만 Siemens는 회사 차원에서 MCP를 채택 중이다.** 공식 문서 검색 MCP(`mcp.siemens.com`), Mendix MCP Server 모듈, EDA용 Fuse AI Agent(2026-03)가 있다. Simcenter·HyperMesh로 확장한다는 공식 로드맵은 없다.
3. **HyperMesh → Abaqus 흐름을 MCP로 자동화한 공개 사례가 이미 있다.** `jinkeguo/cax-workflow-agent`는 SolidWorks → HyperMesh 2025(C3D8R) → Abaqus 2022 흐름을 다루며, 물리 설정 변경 시 사람 승인을 받는다. `Cai-aa/CAE-Agent-Hub`(998★)에는 HyperWorks MCP가 들어 있다.
4. **공식 MCP 자체가 필요하면 현실적인 선택지는 Ansys(알파)**이다. COMSOL 2027은 출시 대기 중이고, 자동차 구조 프리프로세싱 대체재는 아니다.
5. **권장 경로**: 단기에는 HyperMesh용 **사내 MCP 브리지**를 만든다(커뮤니티 구조 참고, Tcl/Python + hmbatch). 여기에 Abaqus 측 검증·제출·ODB 추출 MCP를 결합한다. 화이트리스트, 승인 게이트, 실행 결과 기반 수정 루프는 필수다(§5, §6).

---

## 1. 배경

### 1.1 MCP란

- **Model Context Protocol**: Anthropic이 2024-11에 공개한 개방형 표준이다. LLM 클라이언트가 외부 도구·데이터를 일관된 방식으로 호출하게 해 준다.
- 2025-12-09에 Anthropic이 MCP를 **Linux Foundation 산하 Agentic AI Foundation(AAIF)**에 기증했다. AAIF는 Anthropic·Block·OpenAI가 공동 설립했고 Google·Microsoft·AWS·Cloudflare·Bloomberg가 지원한다. 발표 시점에 공개 MCP 서버는 1만 개를 넘었다. ChatGPT, Cursor, Gemini, Microsoft Copilot, VS Code 등이 MCP를 채택했다. ([Anthropic 발표](https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation)) ✅
- 구조는 다음과 같다. 전송 방식은 로컬은 stdio, 원격은 Streamable HTTP다.

  ```
  MCP 호스트 (Claude Code/Desktop, Copilot, Cursor, Codex …)
      ⇅  JSON-RPC
  MCP 서버 ("도구" 목록과 입력 스키마 정의)
      ⇅  각 툴의 API (Python/Tcl/gRPC/COM …)
  대상 소프트웨어 (HyperMesh, Abaqus, Ansys …)
  ```
- **CAE 관점의 의미**: 벤더가 MCP 서버를 제공하면 사용자가 브리지를 직접 만들 필요가 없다. LLM이 해당 툴의 API를 "도구"로 바로 호출한다. 벤더가 제공하지 않으면 사용자나 커뮤니티가 툴의 스크립팅 API를 감싸는 MCP 서버를 따로 만들어야 한다.

### 1.2 판정 기준

| 분류 | 정의 |
|---|---|
| **공식(OFFICIAL)** | 벤더 소유 채널에서 배포한 것: 공식 웹사이트·문서·보도자료, 공식 GitHub 조직, PyPI 벤더 계정 |
| **공식 발표(ANNOUNCED)** | 공식 발표는 있으나 아직 배포 전 |
| **커뮤니티(COMMUNITY)** | 개인이나 제3자가 만든 것. 벤더와 무관 |
| **AI 어시스턴트(MCP 아님)** | 제품에 내장된 Copilot·챗봇·에이전트. 외부에 MCP 서버나 클라이언트를 노출하지 않음 |

> ⚠️ **동명이의 주의**: KAIST의 **"MCP-SIM"**(Park, Moon, Ryu)에서 MCP는 *Memory-Coordinated Physics-aware*의 약자다. Model Context Protocol과는 무관하다(§4.3).

---

## 2. 벤더별 상세

### 2.1 Ansys (Synopsys): 유일한 공식 배포, 알파 단계

**근거 (✅ GitHub `ansys` 조직 + PyPI 원문)**

- 공식 GitHub 조직 `github.com/ansys`에 MCP 저장소 7개가 있다(2026-10-06 기준).
  - `pymechanical-mcp`, `pymapdl-mcp`, `pyfluent-mcp`, `pycfx-mcp`, `pyaedt-mcp`, `pylumerical-mcp`, `pyansys-common-mcp`
- PyPI 저자는 `"ANSYS, Inc." <pyansys.core@ansys.com>`다. 공통 속성:
  - 라이선스 Apache-2.0
  - `Development Status :: 3 - Alpha`
  - Python 3.12 이상
- 모든 서버는 **사용자 PC에 설치된 라이선스 Ansys**에 접속하는 방식이다. MCP 서버를 설치한다고 해석 라이선스가 생기지는 않는다.
- 사용 조건(PyMechanical-MCP): Ansys Mechanical **2024 R2(v242) 이상** + 라이선스, Python 3.12~3.14, Windows/Linux ✅. 자세한 내용과 설치 절차는 `_REPORT_Ansys_MCP_알파_사용조건_261006.md`에 정리했다.

**공개 타임라인 (PyPI 업로드일 기준, ✅)**

| 날짜 | 이벤트 |
|---|---|
| 2026-04-22 | `ansys-common-mcp` 0.3.0: 공통 프레임워크, 확인된 최초의 공식 산출물 |
| 2026-06-09 | `ansys-mapdl-mcp` 0.2.0 |
| 2026-06-25 | `ansys-mechanical-mcp` 0.1.dev0 |
| 2026-07-10 | `ansys-fluent-mcp` 0.1.0 |
| 2026-08-14 | `ansys-fluent-mcp` 0.4.0: 메싱 워크플로 지원 추가 📄 |
| 2026-09-17 | `ansys-mechanical-mcp` 0.2.1 (최신) |
| 2026-09-25 | `ansys-fluent-mcp` 0.5.0 (최신) |
| 2026-09-28 | `ansys-mapdl-mcp` 0.3.1 (최신) |

**서버별 메싱 관련 기능**

| 서버 | 메싱 관련 기능 | 비고 |
|---|---|---|
| **PyMechanical-MCP** | 도구는 24개이고 그중 메시 전용 도구는 없다. 메싱은 `run_python_script`로 Mechanical 스크립트를 실행해서 한다. `get_guidelines_for("meshing")`가 사이징·MultiZone·메시 통계 사용법을 안내하고, "skewness > 0.95 또는 aspect ratio > 50이면 문제"라는 기준을 제시한다 📄 | stdio/HTTP 지원. 클라이언트는 VS Code, Claude Code, Claude Desktop ✅. Mechanical에는 gRPC로 연결 📄 |
| **PyMAPDL-MCP** | `run_mapdl_command(s)`와 `run_python_code`로 임의 APDL을 실행하므로 ET/MP/ESIZE/VMESH 등을 쓸 수 있다. 가이드라인에 메시 항목이 있다 📄 | 로컬, 원격, Docker 모두 가능. README에는 메싱이 명시돼 있지 않다 ✅ |
| **PyFluent-MCP** | Fluent를 meshing 모드로 띄워 Watertight Geometry, Fault-tolerant Meshing, 2D Meshing 워크플로를 실행한다. `mesh_quality` 도구는 셀·면·노드 수, skewness, orthogonal quality, aspect ratio를 반환한다 ✅ | 도구 20개. CFD 메싱 위주 |

**공식 예제 (📄 `pymechanical-mcp/examples`)**: 모두 자연어 한 번으로 STEP → 메시 → 경계조건 → 해석 → 후처리를 끝까지 수행한다.
- 구멍 뚫린 평판 STEP: "2 mm로 메시" → 고정 + 10 kN 하중 → von Mises 스크린샷 → Kt ≈ 3 확인
- L-브래킷 모달 해석: 3 mm 메시, 6개 모드
- 캔틸레버: 5 mm 메시, 1 MPa 압력

**공식 MCP가 없는 Ansys 제품**: Ansys Prime(PyPrimeMesh), SpaceClaim/Discovery, LS-DYNA/LS-PrePost(PyDyna), Workbench, SimAI. 이들은 커뮤니티 MCP만 있다 📄.

**제품 내장 AI (MCP 아님, 🔎)**
- **Ansys 2026 R1**(2026-03-11)이 포트폴리오 최초의 에이전트 기능을 넣었다.
  - **Mesh Agent**: Mechanical 안에서 메싱 실패를 디버그하고 해결한다. "exploratory use" 단계다.
  - Discovery Validation Agent, GeomAI도 함께 출시됐다.
- Ansys Innovation Space에 교육 과정 "Agentic AI for Ansys Mechanical and MAPDL Workflows"가 있다. Claude Code, Codex, Cursor, Copilot으로 Mechanical/MAPDL을 구동하는 내용이다.

**시사점**
- 공식 지원은 0.x 알파라서 **API 변경이 잦다**(Mechanical 0.1.0 → 0.2.1이 두 달 만에 나왔다).
- 공식 서버도 "범용 스크립트 실행 + 가이드라인" 구조다. 전용 메시 도구를 촘촘히 정의한 형태가 아니다. 커뮤니티 구현과 설계 철학이 크게 다르지 않다.

### 2.2 COMSOL: 공식 MCP 서버 발표 (Version 2027)

- **2026-09-16 보도자료** 🔎 ([COMSOL](https://www.comsol.com/press-release/la-modellazione-di-sistema-e-lintelligenza-artificiale-agentica-sono-protagoniste-in-comsol-multiphysics-versione-2027-14722), [Laser Focus World 전재](https://www.laserfocusworld.com/directory/services-software/software-cad-cae-cam/press-release/55405579/comsol-system-level-modeling-and-agentic-ai-take-the-spotlight-in-comsol-multiphysics-version-2027))
  - *"Version 2027 … with the new COMSOL MCP Server, which uses the Model Context Protocol (MCP) to provide a standardized way for AI agents to interact directly with COMSOL Multiphysics."*
  - 에이전트가 COMSOL API로 모델을 만들거나 수정하고, 해석을 실행하고, 결과를 보고 다음 행동을 결정한다. 사용자는 COMSOL Desktop에서 그 작업을 검토하고 수정할 수 있다.
  - 출시 시점은 "later this fall"이다. 2026-10-06 기준 정식 출시(GA) 여부는 확인하지 못했다. 도구 목록, 전송 방식, 라이선스 조건도 미공개다.
- **이전 단계 (MCP 아님)** 📄
  - 6.3: Chatbot 창 도입. OpenAI GPT로 COMSOL Java API 코드를 생성하고 디버그한다.
  - 6.4: OpenAI 호환 API를 쓰는 여러 LLM 공급자를 지원한다.
- **커뮤니티** 📄: `wjc9011/COMSOL_Multiphysics_MCP`(약 805★)
  - MPh 기반이며 도구 76개를 제공한다. 지오메트리, 메시 시퀀스, 물리, 해석, 결과를 다룬다.
  - 관련 논문이 *Neurocomputing*(2026)에 실렸다.
- **한계**: COMSOL은 범용 멀티피직스 도구다. 차체·섀시 부품의 대규모 구조 프리프로세싱(중립면 추출, 용접·커넥터, 배치 메싱 품질 기준)에서 HyperMesh나 ANSA를 대체하기는 어렵다.

### 2.3 MathWorks: 범용 MATLAB MCP와 PDE Toolbox

- **MATLAB MCP Server**(구 "MATLAB MCP Core Server")
  - 공식 위치는 `github.com/matlab/matlab-mcp-server`이고 1.6k★다 ✅.
  - README: *"Run MATLAB® using AI applications with the official MATLAB MCP Server from MathWorks®."* ✅
  - 도구: `detect_matlab_toolboxes`, `check_matlab_code`, `evaluate_matlab_code`, `run_matlab_file`, `run_matlab_test_file` ✅
  - 전송은 stdio다. 클라이언트는 Claude Code/Desktop, GitHub Copilot(VS Code), Codex이고, MATLAB R2021a 이상이 필요하다 ✅.
  - 라이선스 조건: *"must not be shared by multiple users"*, 즉 한 서버를 여러 사용자가 공유하면 안 된다 ✅.
  - 최초 공개는 2025-10-31(블로그는 2025-11-03)이고 v0.14.0이 2026-09-25에 나왔다 📄.
- **MATLAB Agentic Toolkit의 `matlab-solve-pde` 스킬**(2026-07-02) 📄
  - STEP/STL을 불러온다(`fegeometry`).
  - `generateMesh(Hmax, Hmin, HFace…)`로 3D 사면체 메시를 만든다.
  - `femodel`로 구조·열·전자기 해석을 하고 von Mises 응력을 계산한다.
- **한계** 📄
  - PDE Toolbox 메시는 사면체뿐이다. 육면체(C3D8) 메시는 만들 수 없다.
  - Abaqus `.inp` 직접 출력 기능이 없다.
  - 그래서 HyperMesh를 대체하기보다는 **간이 검증이나 연구용 보조 도구**에 가깝다.

### 2.4 Siemens: Simcenter HyperMesh 포함

#### 2.4.1 HyperMesh: 공식 MCP 없음 (재확인)

| 근거 | 내용 | 검증 |
|---|---|---|
| 릴리스 노트 | HyperMesh 2024·2025·2025.1·2026 "API and Customization" 항목에는 Python/Tcl API 변경만 있다. MCP나 에이전트 언급은 없다 | 📄 |
| 신제품 발표 | HyperWorks 2026(2025-12-08), Simcenter HyperMesh 2026.1(2026-06), 통합 Simcenter 릴리스(2026-07-28), Realize LIVE 2026(2026-06) 모두 MCP 언급이 없다 | 🔎📄 |
| 공식 GitHub | `altairengineering` 조직 저장소 26개 중 MCP 관련은 없다. HyperWorks 관련은 VS Code 자동완성용 `HyperWorksPyAPI` 하나뿐이다 | 📄 |

**HyperMesh에 들어간 AI 기능 (모두 MCP 아님)**

| 기능 | 내용 | 시기 |
|---|---|---|
| Altair **CoPilot** | 인터넷 연결이 필요하다. 도움말, 지식베이스, 영상만으로 답한다. `#script` 또는 `/script`로 HyperMesh **Python 코드를 생성**하며 이 기능은 베타다. 생성한 코드를 직접 실행한다는 근거는 없다. 2024.1 문서의 솔버 프로파일 목록에 **Abaqus는 빠져 있다** | 2024.1 베타 📄 |
| Simcenter HyperMesh Copilot | Simcenter 도움말을 근거로 답하는 어시스턴트다. 워크플로 구성도 안내한다 | 2026 / 2026.1 🔎 |
| Simcenter **PhysicsAI** | 기하 딥러닝 기반 해석 결과 예측이다(LLM 아님). 2026.1부터 관심 영역 학습과 **요소 단위 필드 학습**을 지원한다 | 2025-12, 2026.1 🔎 |
| **Retrieve (ShapeAI)** | 유사 형상 부품을 찾아 기존 메시를 재사용한다 | 2026.1 🔎 ([Siemens 블로그](https://blogs.sw.siemens.com/simcenter/whats-new-in-simcenter-hypermesh-2026-1/)) |
| Python Recording | GUI 조작을 Python 코드로 기록한다(AI 아님). LLM 학습·그라운딩 예제로 쓰기 좋다 | 2025 📄 |

> 참고: 2026년부터 HyperMesh, HyperView, Inspire, SimLab 등은 **"Simcenter" 브랜드**로 바뀌었다 📄.
> GNN 연구 관점에서는 PhysicsAI의 **요소 단위 필드 학습**(2026.1)이 `1. GNN/260505` 보고서의 노드-요소 도메인 불일치 논의와 맞닿아 있다.

#### 2.4.2 Siemens 전사의 MCP 동향 (HyperMesh 외)

| 항목 | 내용 | 분류 |
|---|---|---|
| **Siemens MCP Server** (`mcp.siemens.com`) | 공개 서버이며 인증이 필요 없다. 플러그인 4개, 도구 8개로 문서·제품·API·웹 콘텐츠를 검색한다. **Simcenter를 조작하는 기능은 없다** | 공식 📄 |
| **Mendix MCP Server** 모듈 | Mendix 앱 로직을 MCP 도구로 노출하는 공식 모듈이다 | 공식 📄 |
| **Fuse EDA AI Agent** | 2026-03-16 발표. MCP와 Agent Skills를 지원한다(EDA 전용) | 공식 📄 |
| Intelligence Center X | 2026-06-01 발표. AI 오케스트레이션 제품이다. MCP 연동 언급은 파트너(CLEVR) 자료에만 있다 | 미확인 📄 |
| Simcenter 3D / Femap / NX / STAR-CCM+ | 공식 MCP가 없다. STAR-CCM+는 Siemens 커뮤니티 포럼에 "Macros with AI" 연재가 있다. Javadoc을 검색하는 MCP 서버를 만드는 튜토리얼이며 저자 소속은 확인하지 못했다 | 없음 / 커뮤니티 📄 |

**해석**: Siemens는 문서, EDA, Mendix에서 이미 MCP를 쓰고 있고, 구 Altair의 RapidMiner Graph Studio도 2025-10에 MCP 연동을 발표했다. 따라서 **Simcenter 계열로의 확장 가능성은 있다.** 그러나 2026-10 현재 **공식 발표는 없다.**

### 2.5 Cadence (구 BETA CAE): ANSA/META, Pointwise

- **공식 MCP: 찾지 못했다.** beta-cae.com을 직접 열람할 수 없었으므로 신뢰도는 중간이다.
- **BETA CAE Suite 2026.1**(2026-07-29) 🔎
  - 핵심 기능으로 *"The Agentic AI gaining ground in ANSA, EPILYSIS, META, FATIQ, KOMVOS and ANSERS"*를 내세웠다 ([BETA CAE 발표](https://www.beta-cae.com/news/20260729_announcement_suite_2026.1.htm), [CIMdata 요약](https://www.cimdata.com/en/industry-summary-articles/item/30549-beta-cae-suite-2026-1-is-now-available)).
  - 출발점은 25.0.0의 AI Assistant다. 문서 RAG와 자연어 기반 Python 스크립트 생성을 제공했다.
  - 이후 에이전트 모드가 추가됐고 **on-prem/self-hosted LLM**을 지원하며 추가 라이선스가 필요 없다 📄.
  - **MCP 지원 여부는 확인하지 못했다.**
- **Fidelity Pointwise**: 공식 GitHub(`pointwise`, 저장소 89개)에 MCP가 없다. Glyph 스크립트 자동화만 있다 📄.
- **커뮤니티 "ANSA MCP"**(`Cai-aa/CAE-Agent-Hub` 안) 📄
  - ANSA 25.1.2에서만 테스트됐다. 도구 18개와 operation 24개를 제공한다.
    - 지오메트리 수리
    - 표면·사면체 메시와 품질 수리
    - Abaqus CLOAD/BOUNDARY/접촉 설정
    - Abaqus·Nastran 덱 입출력
  - ANSA 내부 플러그인과 HMAC 인증 루프백(127.0.0.1)으로 통신한다.

### 2.6 Dassault Systèmes: Abaqus, 3DEXPERIENCE

- **공식 MCP 없음.** 공개 MCP 엔드포인트도 커넥터도 없다 📄.
- **가상 동반자 AURA·LEO·MARIE** 🔎
  - 2026-02 3DEXPERIENCE World에서 공개됐고 **2026-07-23에 출시**됐다(19개 역량) ([3DS 보도자료](https://www.3ds.com/newsroom/press-releases/dassault-systemes-expands-3dexperience-ai-native-agentic-platform-new-virtual-companion-skills-co-engineer-humans)).
  - **LEO**는 회사 베스트 프랙티스를 반영해 시뮬레이션 셋업·실행·결과 분석을 돕는다(SIMULIA R2026x FD03) 📄.
  - Mistral AI 모델 기반이며 OUTSCALE 클라우드에서 동작한다(DEVELOP3D 보도) 📄.
  - 플랫폼 내부에서 MCP와 A2A를 쓴다는 보도(LeMagIT)가 있으나 공식 확인은 안 됐다.
- **Abaqus 커뮤니티 MCP는 20종 이상**이다. 주요 저장소:

| 저장소 | ★ | 연결 방식 | 특징 | 검증 |
|---|---|---|---|---|
| `Cai-aa/abaqus-mcp` | 287 | CAE 플러그인 + 파일 기반 IPC | `execute_script`, `get_model_info`, `submit_job`, `get_odb_info`, `get_viewport_image`. MIT | ✅ |
| `Whfkl/Abaqus-Control-MCP` | 200 | CAE 플러그인 + TCP 127.0.0.1:48152 | `run_python`(현재 mdb에 대해 실행), `monitor_job_status`, `inspect_odb`, `capture_viewport`. 전용 메싱 도구는 없음. Python 2 버전은 별도 포크 | ✅ |
| `jianzhichun/abaqus-mcp-server` | 105 | pywinauto GUI 자동화 | 스크립트 실행과 메시지 로그 스크래핑. 취약한 방식 | 📄 |
| `TutuBarry/abaqus-mcp-pro` | 3 | stdio + TCP | **메싱 도구가 가장 풍부하다**: `seed_part`, `generate_mesh`, `set_element_type`(C3D8R 등), 재료·단면·BC·하중·구속(tie/coupling/MPC). Abaqus 2024 이상 | 📄 |
| `Tomsabay/abaqus_agent` | 28 | MCP + FastAPI/SSE | "Solver Doctor"(오류 패턴 30종 이상 진단), 결과 물리 체크. AGPL-3.0 | 📄 |
| `rutwikg/abaqus-mcp` | 2 | stdio | JSON 스펙 → 모델 → 해석. 덱 자동 수정, 음의 Jacobian이 나오면 재메싱. AGPL-3.0 | 📄 |

### 2.7 Hexagon, Keysight(ESI), 기타 상용

| 프로그램 | 공식 MCP | 비고 | 검증 |
|---|---|---|---|
| MSC Apex / Patran / Marc / Nastran / Simufact (Hexagon) | 없음 | 커뮤니티로는 `pynastran-mcp`(BDF 덱 읽기·쓰기, 메시 품질 체크, OP2 결과)가 있다. Simcenter Nastran과 MSC Nastran 덱 모두 대상 | 📄 |
| Visual-Environment / Visual-Mesh / VPS (Keysight·ESI) | 없음 | Keysight의 공식 MCP는 CyPerf(네트워크 테스트) 전용이다 | 📄 |
| Coreform Cubit | 없음 | 커뮤니티 `cubit-mesh-export`(`mcp-server-cubit`)가 있다. auto → sweep → polyhedron → tetmesh 순서로 전략을 시도하고 `.msh`/`.bdf`/`.vtk`로 출력한다 | 📄 |
| Autodesk Fusion | **공식 샘플**만 | `FusionMCPSample`: Fusion API 스크립트 실행, 스크린샷. 시뮬레이션·메시 도구는 없다. Autodesk의 Fusion Data/Revit MCP는 데이터 중심이다 | 📄 |
| PTC Onshape + **SimScale** | 없음 | SimScale **Engineering AI Agent**가 Onshape 앱으로 출시됐다(IMTS 2026, 2026-09). CAD에서 출발해 셋업 제안 → **메시** → 해석 → 결과 설명까지 한다. **MCP 여부는 확인하지 못했다**(제품 내장 에이전트) | 🔎 |
| MIDAS IT (NFX, FEA NX, Civil/Gen NX) | 없음 | 공식 Python API(midas-civil, midas-gen)만 있다. PyPI `midas-mcp` 0.0.0은 비공식 placeholder다 | 📄 |
| McNeel Rhino | **공식** (RhinoAI 0.x) | 형상 생성·편집용이다. FE 메시는 다루지 않는다 | 📄 |
| Luminary Cloud, nTop, Rescale, Quanscient | 찾지 못함 | — | 📄 |

### 2.8 오픈소스 메셔·솔버와 MCP 레지스트리

**결론**
- 다음 프로젝트는 **모두 공식 MCP가 없다**: Gmsh, FreeCAD, Salome/Code_Aster, Netgen/NGSolve, MeshLab, TetGen/fTetWild, CalculiX/PrePoMax, Elmer, FEniCS, MOOSE, deal.II, OpenFOAM, SU2. 확인 방법은 각 프로젝트 공식 GitHub 조직의 "mcp" 검색이다 📄.
- 프로젝트 공식 조직이 직접 올린 MCP는 3개뿐이다. 셋 다 **메싱을 하지 않는다.**
  - `Kitware/vtk-mcp`: VTK API 지식, 문서 검색, 코드 검증 ✅
  - `pyvista/pyvista-mcp-server`: `hello_world` 데모 수준
  - `CadQuery/cadquery-contrib`의 MCP: CAD 전용
- 실제로 메시를 만드는 오픈소스 계열 MCP는 **모두 커뮤니티 작품**이다. 대부분 2026년에 나왔고 대부분 20★ 미만이다.

| 도구 | 대표 커뮤니티 MCP | 메싱 관련 기능 | 성숙도 |
|---|---|---|---|
| **Gmsh** | `OFFTECH/gmsh-mcp-server` | 도구 36개: OCC 형상·불리언, transfinite·O-H 블록 템플릿, 품질 히스토그램(minSICN/minDetJac), MSH 2.2 출력. 임의 CAD를 자동으로 블록 분할하지는 못한다 | 신생(2026-09) |
| **FreeCAD FEM** | `neka-nat/freecad-mcp` (2.7k★ ✅), `gchen19/AnkusDrive`, `tessalabs-space/freecad-mcp` | neka-nat에는 `run_fem_analysis`(CalculiX 실행)가 있으나 메시 도구는 없다. AnkusDrive는 도구 280개 이상으로 Gmsh/Netgen 메싱을 한다. tessalabs는 중립면·defeaturing과 UNV/INP/MED/BDF 출력을 지원한다 | neka-nat만 대중적 |
| **Salome** | `gnshb/salome-mcp` | GEOM 형상·그룹, SMESH Netgen 1D-2D-3D 메싱과 통계 | 프로토타입 |
| **CalculiX** | `Casys-AI/mcp-calculix`, `mcp-fea`, CAE-Agent-Hub CalculiX | STEP → Gmsh 사면체(C3D10) → ccx. mcp-fea는 캔틸레버 처짐 오차 0.02%, NAFEMS LE10 1.41%를 자체 보고 | 신생, 잘 설계됨 |
| **Elmer / FEniCS / SU2** | FEP-Agent-Hub, `agentfem-mcp`, `cmudrc/su2-mcp` | Gmsh 연동 메싱 → 해석. 해석해와 비교 검증 | 초기 |
| **OpenFOAM** | **Foam-Agent**(RPI, 338★ ✅), `webworn/openfoam-mcp-server`(121★) | Foam-Agent는 MCP 도구 `plan`, `input_writer`, `run`, `review`, `apply_fixes`, `run_case`, `visualization`을 제공하고 stdio와 HTTP(포트 7860)를 지원한다. `claude mcp add foamagent -- foamagent-mcp`로 연결하며 Gmsh `.msh`, blockMesh/snappyHexMesh 메싱을 쓴다 ✅ | 가장 성숙. 단 OpenFOAM 재단 공식은 아님 |
| **ParaView** | `LLNL/paraview_mcp` (IEEE VIS 2025) | 후처리 시각화만 한다. pvserver 동기화 방식이 낡아 불안정하다는 경고가 있다 | 연구 프로토타입 |

**MCP 레지스트리 조사 결과** 📄
- 공식 MCP Registry(`registry.modelcontextprotocol.io`)를 약 70개 검색어로 조회했다.
- 다음 검색어는 **등록 0건**이었다: gmsh, abaqus, comsol, hypermesh·altair·optistruct, ANSA, LS-DYNA, nastran·femap, calculix, salome, fenics.
- Ansys 공식 PyAnsys MCP들도 **레지스트리에 등록돼 있지 않다.**
- awesome-mcp-servers류 목록에도 CAE 분류는 없다.
- → **CAE용 MCP는 레지스트리보다 GitHub에서 직접 찾아야 한다.** 이번 조사도 대부분 GitHub 탐색으로 찾았다.

---

## 3. HyperMesh 심층: 공식 MCP 없이 쓸 수 있는 선택지

### 3.1 커뮤니티 HyperMesh MCP 비교

| 저장소 | 최근 활동 (2026) | ★ / 라이선스 | 연결 방식 | 주요 기능 | 성숙도 |
|---|---|---|---|---|---|
| **`Cai-aa/CAE-Agent-Hub`의 HyperWorks MCP** | v0.10.0 (07-16 ~ 08-18) | 저장소 998★ / MIT | HyperWorks 내부 Python 확장(Qt 메인 스레드, 127.0.0.1, 요청마다 토큰 인증) + HyperMesh Batch로 Tcl 선별 실행 | 프로젝트 관리. CAD 가져오기(STEP/IGES/Parasolid). `automesh_live_surfaces`, `solid_map_live_solids`, `tetra_mesh_live_solids`, 원통형 O-grid. `get_live_mesh_quality`, `repair_live_mesh_quality`. 실패하면 `.hm` 체크포인트로 롤백. 강체·RBE3·용접, 솔버 카드·하중, HyperStudy, Job 제출, HyperView 후처리 | 가장 포괄적이다. **임의 Tcl·Python·셸 실행을 금지**하고 허용 목록만 쓴다. 호출당 5,000 노드·요소 제한. OptiStruct 6종, Radioss 4종 검증. 크래시 박스 등 일부 템플릿은 잠금. Abaqus는 일부만 지원 ✅ |
| **`jinkeguo/cax-workflow-agent`** | 07-26 | 1★ / MIT | Codex 플러그인(MCP 도구 32개). SolidWorks는 COM, HyperMesh는 Tcl/batch 어댑터로 연결 | Solid map 메싱(C3D8R). Jacobian·종횡비·최소 길이 품질 체크. Abaqus 덱 출력, datacheck, 제출, ODB 추출, 실패 진단과 복구 | **HyperMesh 2025 + Abaqus 2022에서 실제로 검증됐다.** 형상·메시 정책·재료·접촉·하중·BC를 바꿀 때는 **사람 승인이 필요하다**. 사용자의 HyperMesh+Abaqus 워크플로에 가장 가깝다 ✅ |
| `times1234/hypermesh-mcp` | 05-11 ~ 08-10 | 10★ / 라이선스 미표기 | hmbatch + GUI 소켓 리스너(47881) | 형상을 탐색(probe)해 자동 분류한다. 압출형은 drag hex, 회전체는 spin hex, 복잡 형상은 기어 인식 tet. v5.0에서 ANSA 연동 추가 | 생성기가 만든 스크립트가 아니면 **메싱 Tcl을 직접 실행하지 못한다.** PulseMCP에 등재됐다 ✅ |
| `yesooner/hyper-dyna-mcp` | 06-12 | 18★ / **AGPL-3.0** | FastMCP stdio + `hmcustom.tcl` 리스너(47883), GUI 전용 | `hm_modeling_action`: 메시·요소 생성, 재료·물성, 구속·하중. LS-DYNA 모델링용 | 규칙: *"Do not guess unverified HyperMesh Tcl commands"*. `*tetmesh`, 표면 automesh, K 파일 출력은 차단 ✅ |
| `wangguan1995/hypermesh-mcp` | 08-15 | 1★ | hmbatch(Altair 2026 경로) | STEP → `.hm` 변환, 자동 메시 | 프로토타입 📄 |
| `times1234/hypermesh-mcp-server` | 05-04 | 4★ | Tcl 생성 → hmbatch/GUI | 형상 탐색, tet/hex | 초기 버전 📄 |

> **공통 패턴**: 위 구현들은 모두 같은 구조를 쓴다. ① HyperMesh 안에 **Tcl 또는 Python 리스너**를 띄우고, ② 외부 **MCP 서버(Python)**가 localhost 소켓으로 명령을 보내며, ③ 대량 작업은 **hmbatch** 배치로 돌린다.
> **공통 교훈**: LLM은 존재하지 않는 HyperMesh Tcl 명령을 지어낸다. 그래서 **검증된 경로, 화이트리스트, 스크립트 생성기만 허용**하는 방어 설계가 표준이 되었다.
> **라이선스 주의**: AGPL-3.0이나 라이선스 미표기 저장소는 사내 재사용·배포 시 법무 검토가 필요하다.

### 3.2 자체 구축을 위한 HyperMesh API 정리 📄

| 경로 | 내용 | 버전 |
|---|---|---|
| **Tcl API** | 변경 명령은 `*` 계열(`*createmark`, `*tetmesh`, `*feoutputwithdata` 등), 조회는 `hm_*` 계열(`hm_getvalue` 등) | 거의 전 버전. 구버전 설치본에서도 동작 |
| **Python API** | `import hm`, `hm.entities`, `hm.Model()`, `hm.Collection(...)`. 2025에 Python Recording, 2025.1에 `Collection.get_values()`(NumPy 반환), 2026에 `Model.get(...)`과 Assembly가 추가됐다 | **HyperMesh 2024에서 처음 출시**(HyperView·HyperGraph는 2023.1). `hm`은 **HyperMesh 내장 Python에서만** import할 수 있다 |
| **배치 모드** | `hmbatch -tcl <script>`. Python 스크립트는 `runhwx … -b -f script.py`(커뮤니티 답변) | 2026에도 hmbatch가 포함돼 있다 |
| **외부에서 실행 중 세션 제어** | 공식 REST·COM 서버는 **없다.** ① Tcl `socket -server` 리스너(모든 커뮤니티 구현이 이 방식), ② 레거시 Process Manager 소켓 프로토콜(2017 문서, 현행 지원 여부 미확인), ③ C External API | — |
| **그라운딩 자료** | `command.tcl` 저널(DB를 바꾸는 모든 조작이 기록됨), Python Recording 결과 | LLM에 줄 예제로 쓴다 |

### 3.3 권장 아키텍처 (사내 구축안)

```
[MCP 호스트: Claude Code / Claude Desktop / Copilot 등]
          │ stdio (JSON-RPC)
          ▼
[HyperMesh MCP 서버 — Python (공식 MCP SDK / FastMCP), 사용자 PC]
  ├─ 도구: get_model_summary · list_components · import_cad · batchmesh / tetmesh(params)
  │        quality_report(Jacobian·warpage·aspect·min length) · assign_material/property
  │        create_sets · export_abaqus_inp · run_datacheck · submit_job · extract_odb
  ├─ 가드레일: Tcl 화이트리스트 · 물리 변경 승인 게이트 · 감사 로그 · 원본 보존(체크포인트)
  │
  ├─(A) 라이브 세션 ── 127.0.0.1 TCP + 토큰 ──▶ [HyperMesh GUI]
  │                                           Tcl socket -server 리스너 (전 버전)
  │                                           / Python(hm) 브리지 (2024+)
  ├─(B) 배치 ── hmbatch -tcl / runhwx -b -f ──▶ [HyperMesh 배치: 대량·재현 작업]
  │
  └─(C) 솔버 ── .inp → abaqus datacheck / job ──▶ [Abaqus]
                 └─ .sta/.msg/.dat 파싱 · ODB 추출(Abaqus Python) → 결과 요약 반환
```

**설계 원칙** (커뮤니티 구현과 학술 연구에서 정리, §5)
1. **원자 단위 도구 + 구조화된 반환**: 도구는 품질 지표, 실패 요소 ID, 로그 요약을 JSON으로 돌려준다. LLM이 실행 결과를 보고 스스로 고칠 수 있게 하기 위해서다.
2. **임의 Tcl 실행 도구는 기본 차단**한다. 열어야 한다면 승인을 받는 별도 도구로 분리한다.
3. **사내 표준 반영**: 요소 타입(C3D8 등), 품질 기준, 컴포넌트·세트 명명 규칙을 도구 내부에 고정하거나 MCP 리소스로 제공한다.
4. **장시간 작업은 비동기**로 처리한다. Job ID를 즉시 반환하고 상태는 따로 조회하게 해서 MCP 클라이언트 타임아웃을 피한다.

---

## 4. 사례

### 4.1 벤더·산업 사례

| 주체 | 내용 | 시기 | MCP | 검증 |
|---|---|---|---|---|
| **Altair + Lucid Motors** | 웨비나 "AI Agents of Change". AI 에이전트가 CAD를 가져오고 재료·BC를 설정했다. Lucid **시트 프레임** 워크플로에서는 메타데이터 태그로 설계 표준을 강제했다. 벤더 주장 효과는 셋업 시간 "수 시간 → 수 분"이다 | 2025-09-25 | 명시 안 됨 | 🔎 ([Altair](https://altair.com/resource/ai-agents-of-change-automate-accelerate-simulate-featuring-lucid-motors)) |
| **SimuTech Group** (Ansys 파트너) | Google Antigravity를 **Ansys Mechanical MCP 서버**를 통해 실행 중인 Mechanical 세션에 연결했다. 브래킷 모델을 살펴보고 모달·정적 해석 워크플로를 구성했으며, 랜덤 진동과 피로까지 탐색했다 | 2025~26 | **예** | 📄 |
| **Ansys (Synopsys)** | Innovation Space 교육 과정 "Agentic AI for Ansys Mechanical and MAPDL Workflows"(Claude Code·Codex·Cursor·Copilot) | 2026 | 추정 | 📄 |
| **BMW Group + Mistral AI** | BMW 충돌해석 데이터 1 PB 이상으로 학습한 도메인 특화 "Large Industry Models". 충돌해석 결과 분석 가속이 목적이고 메싱은 다루지 않는다 | 2026-05 | 아니오 | 📄 |
| **SimScale (Onshape 앱)** | 프롬프트 한 번으로 CAD → 셋업 제안 → 메시 → 해석 → 결과 설명까지 진행한다 | 2026-09 | 미확인 | 🔎 ([SimScale](https://www.simscale.com/press/simscale-launches-engineering-ai-agent-for-onshape/)) |
| **Dassault LEO** | 3DEXPERIENCE 안에서 시뮬레이션 셋업·실행·분석을 자연어로 한다 | 2026-07 | 공개 MCP 없음 | 🔎 |
| **PADT** | Microsoft Copilot으로 APDL 스크립트를 생성해 보고 "Yes, you can, sort of"라고 평가했다. 디버깅이 필요했다 | 2025-03 | 아니오 | 📄 |
| Honda Research Institute + Idiap | "MECHANIC" 프로젝트: LLM이 시뮬레이션 기반 설계를 처음부터 끝까지 조율한다 | 2024-12 ~ 2027-11 | — | 📄 |

> **현황 판단**: **OEM이 MCP 기반 메싱으로 정량 성과를 공개한 사례는 아직 찾지 못했다.** 산업 사례는 벤더 웨비나와 파트너 데모 수준이다.

### 4.2 커뮤니티 구현 사례 (MCP + 메싱/해석)

| 사례 | 흐름 | 결과 | 검증 |
|---|---|---|---|
| `jinkeguo/cax-workflow-agent` | SolidWorks(COM) → **HyperMesh 2025**(Tcl/batch, solid map C3D8R, 품질 체크) → **Abaqus 2022**(datacheck, 제출, ODB) | 각 단계가 산출물, 구조화된 체크, 경고, 로그를 반환한다. 실패를 진단하고 복구한다 | ✅ |
| `TmxjTmxj/microneedle-skin-pullout` | STEP → Abaqus/Explicit. 3층 피부 초탄성·점탄성 모델, 일반 접촉(μ=0.38) → 해석 → 반력 이력. **Codex가 MCP(`text-to-cae`)로 생성하고 실행** | 메시는 피부 5,967 요소, 바늘 7,974 요소. 최대 인발력 0.1818 N | ✅ |
| `times1234/hypermesh-mcp` | 구동계 부품(기어, 베어링, 기어박스): 형상 분류 → 스윕·회전 가능한 솔리드는 hex, 나머지는 기어 치형을 세분화한 tet | 일부 회전체는 tet으로 대체된다고 저장소가 스스로 밝힘 | ✅ |
| `Cai-aa/CAE-Agent-Hub`의 ANSA MCP | ANSA 25.1.2 실기 테스트: 박스 메시(셸 16, tet 33), MAT1 생성, Abaqus·Nastran 덱 왕복 | 테스트 240건 통과. 찌그러진 패치는 명시적 smoothing으로 종횡비를 개선 | 📄 |
| `Patrick-Miles-Z/CAD-CC-Abaqus` | DXF → STEP → Abaqus(메시, 재료, BC, 해석)를 Claude Code와 MCP 서버 2개로 구동 | — | 📄 |
| Radia + Cubit (K. Sugahara) | build123d → STEP → Cubit hex 메시 → `.vol`/`.msh`/`.bdf`. "test-then-reflect" 안전 패턴 | — | 📄 |
| `rutwikg/abaqus-mcp`의 **반면교사** | 오일러 좌굴 하중의 1.85배를 받는 캔틸레버가 안정화 옵션 때문에 좌굴하지 않고 0.14 mm 처짐으로 **"수렴"** | 서버가 "SUCCEEDED (with caveats)"로 표시한다. **수렴했다고 정답은 아니다** | 📄 |

### 4.3 학술 연구

**(a) 구조 FE와 메싱** (HyperMesh+Abaqus 스택에 가장 가까운 연구)

| 연구 | 시기 / 출처 | 도구 | MCP | 자동화 범위 | 핵심 결과 | 검증 |
|---|---|---|---|---|---|---|
| **AbaqusAgent** (Sarker et al.) | 2026-06, arXiv 2606.00138 | Abaqus | 아니오 | 자연어 → .inp → 실행 → 오류 검토 → ODB 시각화. 에이전트 6개 | 고체역학 문제 50건 중 **성공률 86%** | 🔎 |
| **MeshExpert** (Zhao et al.) | 2026-07-08, SSRN 7083227 | 프리프로세서 API(종류 미공개) | 미보고 | 메싱 조작 시퀀스 생성. 20B 모델 미세조정 + GRPO(해석 결과를 보상으로 사용) + RAG | **산업용 자동차 부품 50종**에서 **Pass@3 82.5%**, 평균 Jacobian 0.78 | 🔎 |
| VFEAgent | 2026-05, arXiv 2605.28978 | Abaqus | 미보고 | 이미지+텍스트 → FEA 사양 → 검증 우선 코드 생성 | 스키마 유효율 90.0% | 📄 |
| CAX-Agent | 2026-05, arXiv 2605.15218 | Ansys MAPDL | 미보고 | APDL 생성 + 단계적 복구(규칙 → LLM 재생성 → 사람) | 완료율 0.9267(복구 없음 0.6933) | 📄 |
| PAMF | 2026, J. Mech. Sci. Technol. 40:4497–4507 | Ansys APDL | 아니오 | 메시 생성 에이전트 + 오차 예측 에이전트 → 세분화 | 메시 성공 88.4%, 3.69배 빠름 | 📄 |
| FeaGPT | 2025-10, arXiv 2510.21993 | Gmsh + CalculiX | 아니오 | 형상 → 메시 → 해석 → 분석. LLM이 메시 밀도와 세분화 영역을 지정 | 터보차저, NACA 432 구성 | 🔎 |
| FEABench (Google 외) | 2025-04, arXiv 2504.06260 | COMSOL API | 아니오 | COMSOL API 호출 생성 벤치마크 | 실행 가능한 API 호출 88% | 📄 |
| MooseAgent | 2025-04, arXiv 2504.08621 | MOOSE | 아니오 | 입력 파일 생성 + RAG + 반복 수정 | 평균 성공률 93% | 📄 |
| 서베이 (Sandia, Owen 외) | 2025-12, arXiv 2512.23719 | — | — | 형상 준비와 메시 생성 AI 기법 총정리 | 결론: AI는 메싱 알고리즘을 **대체하지 않고 보완**한다 | 📄 |

**(b) CFD와 일반 에이전트** (MCP 채택이 가장 앞선 분야)

| 연구 | 시기 / 출처 | MCP | 핵심 결과 | 검증 |
|---|---|---|---|---|
| **Foam-Agent 2.0** (RPI) | 2025-09, arXiv 2509.18178. CMAME 2026 | **예**: MCP 서버와 Claude Code 스킬 제공 | FoamBench 110과제에서 88.2%(논문). README 자체 보고로는 Opus 4.6이 기본·고급 모두 100% | ✅ |
| CFD-copilot | 2025-12, arXiv 2512.07917 | **예**: 후처리 기능을 MCP로 제공 | 도메인 적응과 MCP를 함께 쓰면 신뢰성이 오른다 | 📄 |
| ParaView-MCP (LLNL) | 2025-05, arXiv 2505.07064. IEEE VIS 2025 | **예** | 자연어 시각화. 정성 평가 | 📄 |
| "What Do CAE Simulation Agents Really Need…" | 2026-09, arXiv 2609.03718 | 범용 하네스 | **범용 에이전트 하나가 96.4%로 특화 다중 에이전트(88.2%)보다 높았다.** 실행 피드백 기반 수정이 71.8% → 96.4%를 만들었다 | 📄 |
| ChatCFD | 2025-06, arXiv 2506.02019 | 아니오 | 실행 성공 82.1%이지만 **물리 정합성은 68.12%** | 📄 |

**(c) 국내**
- **KAIST MCP-SIM**(Donggeun Park, Hyeonbin Moon, Seunghwa Ryu; 기계공학과): 모호한 프롬프트를 FEniCS 시뮬레이션으로 바꾸는 자기수정 다중 에이전트다. 벤치마크 12/12 성공. **Model Context Protocol과 무관한 이름**이다 📄.
- 한국어 검색(현대차그룹, 마이다스아이티, 태성에스엔이, KIMM, 학회 발표)은 조사 예산 문제로 **충분히 수행하지 못했다** → §7 후속 과제.
- **HyperMesh·ANSA를 LLM이나 MCP로 구동한 학술 논문은 찾지 못했다.**

---

## 5. 교훈: MCP 기반 CAE 자동화 체크리스트

| # | 교훈 | 근거 |
|---|---|---|
| 1 | **실행 결과를 보고 고치는 루프**가 가장 큰 효과를 낸다 | 71.8% → 96.4% (arXiv 2609.03718). 완료율 0.69 → 0.93 (CAX-Agent) |
| 2 | **도메인 예제를 넣으면** 약 15%p 오른다. 튜토리얼, 주석 달린 덱, Python Recording 결과, `command.tcl` 등이다 | 80.9% → 96.4% (arXiv 2609.03718) |
| 3 | LLM과 툴 사이에 **구조화된 스펙(JSON) 계층**을 둔다 | abaqus-mcp, 다중 솔버 JSON 번역(정확도 90% 이상) |
| 4 | **Tcl 명령 환각에 대비**한다. 화이트리스트, 검증된 경로, 생성기 산출물만 실행한다 | times1234, yesooner, CAE-Agent-Hub 공통 설계 |
| 5 | **물리 설정 변경은 사람 승인**을 받는다. 형상, 메시 정책, 재료, 접촉, 하중, BC가 해당한다 | jinkeguo, femis-skill |
| 6 | **수렴했다고 정답은 아니다.** 에너지 균형, 반력 합, 질량, hourglass 비율 등을 자동 점검한다 | ChatCFD(실행 82.1%, 물리 정합성 68.1%), rutwikg 좌굴 사례, K-agent(LS-DYNA 에너지·hourglass 게이트) |
| 7 | **장시간 해석은 비동기**로 돌린다. MCP 클라이언트 타임아웃을 피해야 한다 | Foam-Agent가 진행 알림을 추가함(2026-03) |
| 8 | **라이선스 제약**: 상용 솔버는 컨테이너화가 어렵고 라이선스 서버에 접근할 수 있어야 한다 | 커뮤니티 공통 |
| 9 | **데이터 보안**: 양산 차량 형상·해석 데이터를 외부 LLM으로 보낼 때는 사내 규정을 확인한다. on-prem이나 엔터프라이즈 LLM을 검토한다 | BETA CAE가 on-prem LLM 지원을 강조. 본 저장소 CLAUDE.md의 주의사항 |
| 10 | 현재 벤치마크는 단순하거나 파라메트릭한 형상 위주다. **자동차 중립면 추출, 용접·커넥터, 배치 메싱 품질 기준은 거의 검증되지 않았다** | 06 조사 결론 |

---

## 6. 권장 경로 (HyperMesh + Abaqus 사용자 기준)

| 경로 | 내용 | 장점 | 단점 | 권장도 |
|---|---|---|---|---|
| **A. HyperMesh 사내 MCP 브리지** | §3.3 아키텍처. 처음에는 조회와 품질 리포트 도구로 시작하고, 이어 메싱 템플릿 실행, Abaqus 덱 출력·datacheck 순으로 넓힌다 | 기존 툴, 템플릿, 숙련도를 그대로 쓴다. 사내 표준을 반영할 수 있다 | 개발·유지 부담. 벤더 지원 없음 | **★★★ 단기 권장** |
| B. 커뮤니티 MCP 시험 사용 | CAE-Agent-Hub(MIT), jinkeguo(MIT)로 PoC | 빠르게 체험할 수 있다 | 라이선스·보안·검증 부족. HyperWorks MCP의 Abaqus 지원은 제한적 | ★★ PoC 한정 |
| C. Ansys 공식 MCP 병행 | PyMechanical/PyMAPDL-MCP로 공식 MCP 체험과 비교 벤치마크 | 유일한 벤더 공식 지원 | 알파 단계. 툴 전환 비용. 휠 충격 해석 체계는 Abaqus 기반 | ★ 참고용 |
| D. COMSOL 2027 MCP | 출시 후 평가 | 공식 지원 | 자동차 구조 프리프로세싱 용도가 아니다 | 관망 |
| E. Siemens 동향 모니터링 | Simcenter HyperMesh 릴리스 노트, Realize LIVE, `mcp.siemens.com` 확장 여부를 추적 | 공식 MCP가 나오면 A에서 옮겨가면 된다 | 시점이 불확실 | 병행 |

**단계별 착수안 (경로 A)**
1. **1단계, 읽기 전용** (위험이 낮음): `get_model_summary`, `list_components`, `quality_report`. 기존 메시를 진단하고 LLM이 리포트를 쓰게 한다.
2. **2단계, 결정적 자동화**: 사내 메싱 템플릿(Tcl/Python)을 매개변수화한 도구를 만든다(`batchmesh(template, size)` 등). 임의 코드는 실행하지 않는다.
3. **3단계, 솔버 연계**: `export_abaqus_inp`, `run_datacheck`, `submit_job`, `extract_odb`. 실패를 진단하고 수정하는 루프를 넣는다.
4. **4단계, 승인 게이트를 둔 수정**: 형상·물성·경계조건 변경 도구는 사람 승인이 있어야 실행한다.

> **연계 아이디어 (`1. GNN` 시리즈)**: 3단계까지 갖추면 **GNN 학습 데이터 생성 파이프라인**을 MCP 에이전트로 조율할 수 있다. 흐름은 휠 형상 변형 → HyperMesh 메시(C3D8) → Abaqus 13° 충격 해석 → ODB 추출 → 그래프 변환이다. 현재 학습용 휠은 77개다. 데이터를 늘리는 데 걸리는 반복 작업 시간을 줄일 수 있다. 다만 반드시 5번(승인 게이트)과 6번(물리 점검) 교훈을 적용해야 한다.

---

## 7. 미확인 사항과 후속 과제

| 항목 | 상태 |
|---|---|
| COMSOL MCP Server 정식 출시와 사양 | "올가을" 출시 예정. 2026-10-06 기준 출시 여부, 도구 목록, 라이선스 조건 미확인 |
| BETA CAE(ANSA) AI Assistant의 MCP 지원 여부 | 2026.1 "Agentic AI"는 확인했으나 MCP 언급은 미확인(beta-cae.com 직접 열람 불가) |
| Ansys Engineering Copilot, Mesh Agent의 내부 MCP 사용 여부 | 미확인. 공식 서버의 `--on-aali` 플래그로 보아 Ansys 자체 AI 플랫폼(AALI)과 연동하는 구조로 추정 📄 |
| Siemens Intelligence Center X의 MCP 연동 | 파트너(CLEVR) 자료에만 언급됨 |
| Dassault 플랫폼 내부의 MCP/A2A 사용 | LeMagIT 보도뿐 |
| SimScale Onshape 에이전트의 MCP 여부 | 미확인 |
| **한국어 사례**(국내 OEM, 마이다스아이티, 학회) | 조사 예산 문제로 미흡 → 후속 조사 권장 |
| 벤더 도메인 직접 열람 | 네트워크 정책으로 차단. 해당 항목은 검색 색인 요약에 의존(🔎) |

---

## 부록 A. 주요 출처

**공식 (✅ 원문 확인)**
- Ansys MCP 저장소 목록: https://github.com/orgs/ansys/repositories?q=mcp
- PyMechanical-MCP: https://github.com/ansys/pymechanical-mcp · https://pypi.org/project/ansys-mechanical-mcp/
- PyMAPDL-MCP: https://github.com/ansys/pymapdl-mcp · https://pypi.org/project/ansys-mapdl-mcp/
- PyFluent-MCP: https://github.com/ansys/pyfluent-mcp · https://pypi.org/project/ansys-fluent-mcp/
- PyAnsys Common MCP: https://pypi.org/project/ansys-common-mcp/
- MATLAB MCP Server: https://github.com/matlab/matlab-mcp-server
- MCP의 AAIF 기증: https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation

**공식 발표 (🔎 검색 색인 확인)**
- COMSOL Version 2027 / MCP Server (2026-09-16): https://www.comsol.com/press-release/la-modellazione-di-sistema-e-lintelligenza-artificiale-agentica-sono-protagoniste-in-comsol-multiphysics-versione-2027-14722
- Ansys 2026 R1 (Mesh Agent, 2026-03-11): https://news.synopsys.com/2026-03-11-Synopsys-Launches-Ansys-2026-R1-to-Re-Engineer-Engineering-with-Joint-Solutions-and-AI-Powered-Products
- BETA CAE Suite 2026.1 (2026-07-29): https://www.beta-cae.com/news/20260729_announcement_suite_2026.1.htm
- Dassault 가상 동반자 출시 (2026-07-23): https://www.3ds.com/newsroom/press-releases/dassault-systemes-expands-3dexperience-ai-native-agentic-platform-new-virtual-companion-skills-co-engineer-humans
- Simcenter HyperMesh 2026.1: https://blogs.sw.siemens.com/simcenter/whats-new-in-simcenter-hypermesh-2026-1/
- SimScale Engineering AI Agent for Onshape: https://www.simscale.com/press/simscale-launches-engineering-ai-agent-for-onshape/
- Altair + Lucid Motors 웨비나: https://altair.com/resource/ai-agents-of-change-automate-accelerate-simulate-featuring-lucid-motors

**커뮤니티 (✅ 원문 확인)**
- CAE-Agent-Hub (HyperWorks·ANSA·Abaqus·Workbench MCP): https://github.com/cai-aa/CAE-Agent-Hub
- jinkeguo/cax-workflow-agent: https://github.com/jinkeguo/cax-workflow-agent
- times1234/hypermesh-mcp: https://github.com/times1234/hypermesh-mcp
- yesooner/hyper-dyna-mcp: https://github.com/yesooner/hyper-dyna-mcp
- Cai-aa/abaqus-mcp: https://github.com/Cai-aa/abaqus-mcp
- Whfkl/Abaqus-Control-MCP: https://github.com/Whfkl/Abaqus-Control-MCP
- Foam-Agent: https://github.com/csml-rpi/Foam-Agent
- microneedle-skin-pullout: https://github.com/TmxjTmxj/microneedle-skin-pullout
- neka-nat/freecad-mcp: https://github.com/neka-nat/freecad-mcp
- Kitware/vtk-mcp (공식 조직, 메싱 아님): https://github.com/Kitware/vtk-mcp

**학술 (🔎/📄)**
- AbaqusAgent: https://arxiv.org/abs/2606.00138
- MeshExpert: https://papers.ssrn.com/sol3/papers.cfm?abstract_id=7083227
- FeaGPT: https://arxiv.org/abs/2510.21993
- What Do CAE Simulation Agents Really Need: https://arxiv.org/abs/2609.03718
- Foam-Agent 2.0: https://arxiv.org/abs/2509.18178
- Survey (Sandia): https://arxiv.org/abs/2512.23719

전체 출처는 `_workspace/01~06` 원본 산출물에 있다.

## 부록 B. 작업 파일

```
2. MCP/261006/
├── 3D_FE메시_프로그램_공식MCP_지원현황_및_사례_보고서.md   ← 본 보고서
├── _REPORT_Ansys_MCP_알파_사용조건_261006.md               ← Q&A: 알파 의미, 사용 조건, 설치 절차
└── _workspace/
    ├── 00_input/REQUEST.md                 ← 요청·조사 범위·분업
    ├── 01_ansys-cadence.md                 ← Ansys(Synopsys), Cadence(BETA CAE)
    ├── 02_siemens-dassault-hexagon.md      ← Siemens, Dassault, Hexagon, Keysight
    ├── 03_hypermesh.md                     ← HyperMesh 심층
    ├── 04_other-commercial.md              ← COMSOL, MathWorks, Cubit, Autodesk, PTC, MIDAS 등
    ├── 05_opensource-registries.md         ← 오픈소스, MCP 레지스트리
    └── 06_case-studies.md                  ← 사례, 학술 문헌
```
