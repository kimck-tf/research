# 04. MCP servers for simulation and meshing tools outside Ansys, Cadence/BETA, Siemens/Altair, Dassault and Hexagon (status as of 2026-10-06)

> 서브 에이전트 원본 산출물 (영문, 감사 추적용 보존). 최종 보고서는 상위 폴더의 보고서 파일 참조.

**Limits on this pass:**
- **Read directly:** GitHub, raw.githubusercontent and PyPI pages.
- **Could not fetch:** most vendor and news sites were blocked by the egress proxy. These include mathworks.com and blogs.mathworks.com, comsol.com and doc.comsol.com, autodesk.com and aps.autodesk.com, simscale.com, blender.org, arxiv, GlobeNewswire, The Elec and IT Brief.
- **"(idx)" marker:** a claim marked (idx) comes from the search engine's index or summary of a vendor-owned page, not opened directly. Quotes marked (idx) are as indexed and may not be exact wording.
- **Search budget:** the WebSearch budget for this turn ran out partway through. Everything after that point was checked only through vendor GitHub orgs and PyPI. Items not reached are listed in §5.

## 1. Summary table

| Product | Vendor | MCP status | Server + URL | Date / version | Meshing-relevant capability | Class |
|---|---|---|---|---|---|---|
| MATLAB (+ PDE Toolbox) | MathWorks | Released, open source, 0.x versions | **MATLAB MCP Server** (renamed from "MATLAB MCP Core Server") https://github.com/matlab/matlab-mcp-server | v0.1.0 2025-10-31 → **v0.14.0 2026-09-25**; renamed in v0.11.0 (2026-06-18) | General-purpose: runs any MATLAB code. Through PDE Toolbox it can import STL/STEP, make 3D tet meshes, set BCs, loads and materials, solve and post-process. Transport: stdio | [OFFICIAL] |
| MATLAB Agentic Toolkit, skill `matlab-solve-pde` | MathWorks | Released | https://github.com/matlab/matlab-agentic-toolkit | Toolkit announced 2026-04-13 (idx); skill added in release 2026.07.02; latest MATK-2026.09.b (2026-09-24) | An agent skill that walks the agent through the full FEA workflow: geometry import → `generateMesh` → `femodel` → solve → von Mises | [OFFICIAL] (a skill that sits on top of the MCP server) |
| MCP Framework for MATLAB Production Server | MathWorks | Released | https://github.com/matlab/mcp-framework-matlab-production-server | v1.2.4 on File Exchange (idx) | Publishes your own MATLAB functions as remote MCP tools over HTTP | [OFFICIAL] |
| MATLAB MCP HTTP Client | MathWorks | Released | https://github.com/matlab-deep-learning/mcpHTTPClient | Blog 2025-12-10 (idx); v1.0.0, R2025a+ | Lets MATLAB call external streamable-HTTP MCP servers | [OFFICIAL] [MCP-CLIENT] |
| Simulink Agentic Toolkit | MathWorks | Released | https://github.com/matlab/simulink-agentic-toolkit | Latest SATK-2026.09.d (2026-09-30) | Simulink model tools; no meshing | [OFFICIAL] |
| COMSOL Chatbot window | COMSOL | No MCP | Built into COMSOL (Windows only) | 6.3 (late 2024), extended in 6.4 (2025) | Generates and debugs COMSOL Java API code | [AI-ASSISTANT-NO-MCP] |
| **COMSOL MCP Server** | COMSOL | **Announced only**; ships with "Version 2027" | No public URL, tool list or transport yet | Announced 2026-09-16; release "later this fall" | Vendor says agents use the COMSOL API for "all stages of the modeling workflow", which implies geometry, mesh, solve and results | [OFFICIAL] |
| Coreform Cubit | Coreform | None found | — | Cubit 2026.6 (June 2026) | — | Community only |
| Autodesk Fusion | Autodesk | Reference sample only | **FusionMCPSample** https://github.com/AutodeskFusion360/FusionMCPSample | Updated 2026-01-06, MIT | Runs Fusion API Python scripts, takes screenshots. No simulation or mesh tools | [OFFICIAL] sample, unsupported |
| Fusion Data / Model Data Explorer / Revit MCP | Autodesk | Listed on Autodesk's MCP page; Revit is a tech preview (idx) | https://www.autodesk.com/solutions/autodesk-ai/autodesk-mcp-servers | Revit 2027 tech preview, Apr 2026 (idx) | Data, project management and model quantities only | [OFFICIAL] |
| APS sample MCP servers | Autodesk | Samples | github.com/autodesk-platform-services (12 MCP repos) | 2026-03 to 2026-09 | Data Management, AEC Data Model, Revit Automation; no meshing | [OFFICIAL] samples |
| Inventor Nastran, Autodesk Assistant | Autodesk | None found | — | — | — | Unverified |
| Onshape | PTC | None found | — | API changelog through rel-1.221 (2026-09-18) never mentions MCP or AI | — | Community only |
| Creo | PTC | Not checked | — | — | — | Unverified |
| SimScale / nTop / Rescale / Quanscient | — | No MCP repo in their GitHub orgs | — | SimScale SDK v1 updated 2026-09-07 | — | Unverified |
| Luminary Cloud | Luminary | None found | — | SDK 0.27.1 (2026-10-02) | The SDK can import geometry and make meshes; an "AI Assistant" in the UI generates code | [AI-ASSISTANT-NO-MCP] (tentative) |
| Neural Concept / PhysicsX / Monolith | — | Not checked | — | — | — | Unverified |
| MIDAS Civil NX / Gen NX (NFX, FEA NX, MeshFree) | MIDAS IT | None found | Official Python API libraries only | midas-gen 1.6.6 (2026-05-15), midas-civil 1.7.4 | The API wrappers could be wrapped as an MCP server, but none exists | Community placeholder only |
| Rhino / Grasshopper | McNeel | Early releases (0.x) | **RhinoAI** (formerly RhinoMCP) https://github.com/mcneel/RhinoAI | Plugin 0.3.0 2026-09-30; Connector 0.1.6 2026-09-15 | Creates and edits geometry; no FE meshing | [OFFICIAL] |
| Blender | Blender Foundation | No official server found | — | — | — | Community only |

## 2. Details per product

### MathWorks

**MATLAB MCP Server**
- README: "Run MATLAB® using AI applications with the official MATLAB MCP Server from MathWorks®."
- **Tools:** `detect_matlab_toolboxes`, `check_matlab_code`, `evaluate_matlab_code`, `run_matlab_file`, `run_matlab_test_file`.
- **Resources:** `matlab_coding_guidelines` and `plain_text_live_code_guidelines` (the second needs R2025a+).
- **Transport:** stdio only.
- **Clients:** Claude Code, Claude Desktop (installed as an `.mcpb` extension), GitHub Copilot in VS Code, and Codex. The toolkit also lists Gemini CLI and Amp.
- **Requirements:** MATLAB R2021a or later, on Windows, Linux or macOS.
- **Later features:**
  - Can attach to an already open MATLAB session, via `shareMATLABSession()`.
  - Custom tools in JSON extension files.
  - Telemetry can be switched off.
- **Licence:** "MCP servers are only permitted to be used with MATLAB in accordance with the MathWorks Software License Agreement, and must not be shared by multiple users."
- **Release history:**
  - First release: a MathWorks forum post says "released on Friday 31st of October", and a blog post is dated 2025-11-03 (both idx).
  - GitHub release dates for v0.1.0–v0.5.0 show no year; years inferred from those two posts.
- **Meshing:** no meshing-specific tools. Meshing comes from running PDE Toolbox code through `evaluate_matlab_code`.

**`matlab-solve-pde` skill** (official, added 2026-07-02)
- Skill description: "Build and solve finite element models for thermal, structural, and electromagnetic problems using PDE Toolbox™."
- **Geometry import:** `gm = fegeometry("assembly.step")` or an STL file. Options include `FeatureAngle` and `MaxRelativeDeviation`. Existing meshes can be imported with `fegeometry(nodes, elements)` as linear or quadratic tets.
- **Meshing:** `generateMesh(gm, Hmax=0.1, Hmin=0.01)`, with local face sizing via `generateMesh(gm, HFace={[3,5], 0.02})`.
- **Analysis types:** `femodel(AnalysisType="structuralStatic"|"structuralModal"|"thermalTransient"|…)`.
- **Setup and results:** BCs with `faceBC`, loads with `faceLoad`, catalogue or orthotropic materials, `evaluateVonMisesStress`, and `pdeplot3D` for plots.
- **Caveat (agent's product knowledge, not checked):** PDE Toolbox meshes are tetrahedral only, and there is no native Abaqus `.inp` export.

**MCP Framework for MATLAB Production Server**
- README: "Publish MATLAB® functions as Model Context Protocol tools on MATLAB Production Server™."
- Example exposes a principal-stress function at `http://localhost:9910/principalStress/mcp`.
- Requires MATLAB R2025b+, MATLAB Compiler SDK, and MATLAB Production Server R2022a+. Licence BSD-3-Clause.

**MATLAB MCP HTTP Client**
- README: "Create MCP clients to connect to streamable HTTP servers from MATLAB." Makes MATLAB itself an MCP client.

**Simulink Agentic Toolkit**
- Tools include `model_overview`, `model_edit`, `model_check` and `model_test`. Requires R2023a+ with Simulink.

### COMSOL

**Chatbot window (no MCP)**
- Introduced in 6.3, where it talks to OpenAI GPT models to "generate and debug COMSOL API for use with Java code". Generated code can be run inside the model. Windows only (idx).
- 6.4 added any OpenAI-API-compatible provider (DeepSeek, Gemini, self-hosted models), attaching model-tree nodes, and documentation search (idx). The 6.4 press release also names Anthropic Claude (idx).

**COMSOL MCP Server** — press release dated 2026-09-16, Burlington MA (idx)
- "Version 2027 takes the next step in AI-tool integration with the new COMSOL MCP Server, which uses the Model Context Protocol (MCP) to provide a standardized way for AI agents to interact directly with COMSOL Multiphysics."
- "An agent can build and/or modify a model, run simulations, inspect results, and use those results to decide what to do next."
- Mikael Sterner, director of development: the support will connect "modeling, simulation, optimization, validation, and reporting in an iterative, goal-driven process."
- Engineering.com adds that users can review and adjust the agent's work in the COMSOL Desktop (idx).
- **Not yet public:** the tool list, transport, licensing and supported clients.

**Cosmon Nexus** [PARTNER]
- Listed as a COMSOL ISV partner (idx).
- Automates "CAD preparation and geometry cleanup, simulation setup, solver troubleshooting, parametric sweeps…" in COMSOL "through native API access". Whether it uses MCP is not stated.

### Coreform Cubit
- No official MCP or LLM integration found.
- Cubit 2026.6 (June 2026, per third-party coverage, idx) added anisotropic tet meshing, cohesive elements, quality metrics for higher-order elements, and "ML-based feature recognition" for fasteners (not an AI agent).

### Autodesk

**FusionMCPSample**
- "Reference implementation of an MCP server to run Fusion API scripts".
- A Fusion add-in serving HTTP at `http://localhost:9100/`. Claude Desktop connects through `npx mcp-remote`.
- Tools: `execute_api_script`, `get_screenshot`, `get_api_documentation`.
- README: "provided as-is for educational and development purposes."
- No simulation or meshing tools.

**Other Autodesk servers** (Autodesk MCP page, idx)
- Model Data Explorer: "Access, visualize, and explore model data (volumes, areas, counts, etc.) for all 70+ file formats supported by APS."
- Fusion Data: "Invite Fusion collaborators, manage projects, search for data, and add component properties."
- Revit MCP: a read-only tech preview in Revit 2027 with seven tools (idx).

**APS samples and training**
- GitHub: `aps-mcp-server-nodejs` (stdio; archived 2026-05-07), `aps-aecdm-mcp-dotnet`, `aps-sample-revit-mcp-tools-bundle`, AU2026 MCP workshops (Forma data over Streamable HTTP; updated 2026-09-28).
- Autodesk University 2025 class: "Equip AI with Access to Your Design Data and Projects: Model Context Protocol & Autodesk Platform Services."

### PTC
- **Onshape:** developer-docs repo and API changelog (through rel-1.221, 2026-09-18) contain no MCP or AI entries. onshape-public GitHub org has no MCP repo.
- **Creo:** not checked.

### Cloud CAE and AI-CAE vendors
- **SimScale:** SimScaleGmbH org holds SDKs and a Grasshopper plugin (updated 2026-09-21). No MCP repo.
- **nTop, Rescale, Quanscient:** no MCP repos in their GitHub orgs.
- **Luminary Cloud:** SDK covers "importing geometry and creating meshes, running and post-processing simulations"; tutorials mention an in-UI "AI Assistant" for code generation. Neither mentions MCP.

### MIDAS IT
- Official orgs MIDASIT-Co-Ltd, midasit-dev and midas-rnd checked. Filtering MIDASIT-Co-Ltd for "mcp" returns 0 repos; the other two list no MCP repos.
- Official Python libraries midas-civil and midas-gen "automate tasks, extract structural analysis results, manipulate model data" through the MIDAS Open API.
- Nothing found for NFX, FEA NX or MeshFree. Korean-language press not checked.

### McNeel Rhino
- RhinoAI: "A Rhino MCP Server for AI Agents to create and edit in Rhino."
- Rhino-MCP-Platform plugin from Yak (`MCPStart` command); Claude Desktop `connector.mcpb` ("Rhino 8 only!").
- Clients: Claude Desktop and Code, Copilot, Codex, Gemini CLI, LM Studio. MIT licence, 348 stars. Pre-releases since 2026-05-29.

### Blender
- No official server found; blender.org unreachable. The well-known community server says it is "Not affiliated with the official Blender Foundation."

## 3. Community / unofficial MCP servers

**COMSOL**
- **wjc9011/COMSOL_Multiphysics_MCP**
  - MIT licence, about 805 stars, built on the MPh Python library, COMSOL 5.x/6.x.
  - Geometry: primitives, booleans, CAD import. Mesh: mesh sequences, automatic meshing, mesh statistics.
  - Setup and solve: physics, BCs and materials; sync/async solves; parametric sweeps. Results: evaluation, export and plots.
  - Clients: Claude Desktop and opencode.
  - Published paper: Zhang & Wang, *Neurocomputing* 703:134481 (2026); reports 76 tools and retrieval over "36,798 COMSOL documentation chunks" (idx).
- **garbage-enzyme/COMSOL_Multiphysics_MCP_6_4_Calibrated** — fork pinned to COMSOL 6.4.0.293; stdio; mesh-convergence campaigns; Claude Code and Codex CLI.
- Also: Howard-Lai/COMSOL_MCP (MPh, Feb 2026), plus Qwup2112, Twofruitsgrape and Zhangyoupeng1996 servers (listed on Glama; not inspected).

**MATLAB**
- Tsuchijo/matlab-mcp and WilliamCloudQi/matlab-mcp-server: general MATLAB runners, nothing PDE-specific.

**Coreform Cubit**
- **cubit-mesh-export** (`mcp-server-cubit` command), by Kengo Sugahara (github.com/ksugahar/Radia).
  - MIT. v0.1.0 2026-04-03 → v2.1.4 2026-10-01. Windows only, Python 3.12, Cubit 2025.12.
  - Tools start with `cubit_status` and `cubit_docs`; runs APREPRO headless.
  - Exports `.vol`, Gmsh `.msh`, Nastran `.bdf` and `.vtk`.
- Companion **radia-mcp 2.0.1** (2026-10-03): `cubit_mesh_auto` "walks `auto → sweep → polyhedron → tetmesh`"; checkpoints to `.cub5`; Claude Desktop/Code, Cursor, Continue.

**Fusion**
- Joe-Spencer/fusion-mcp-server: GPL-3.0, SSE, sketch and parameter tools.
- AuraFriday/Fusion-360-MCP-Server: proprietary licence, about 128 stars, runs Python against the full Fusion API.

**Onshape**
- hedless/onshape-mcp: MIT, about 147 stars, 45 tools (sketches, features, assemblies, FeatureScript, STL/STEP/Parasolid export). No FEA.

**MIDAS**
- PyPI `midas-mcp` 0.0.0 (2026-05-13): pre-alpha placeholder, "not affiliated with, endorsed by, or sponsored by MIDAS IT". GitHub repo returns 404.

**SimScale, nTop:** none found (not exhaustive).

## 4. Use cases / demos / case studies
1. **MathWorks, Guy on Simulink blog, 2025-12-10:** "Simulating a Simulink Model through GitHub Copilot and the MATLAB MCP Server" (idx).
2. **MathWorks MCP Framework README:** `principalStress` function published as an HTTP MCP tool (minimal structural-calculation example).
3. **MathWorks `matlab-solve-pde` (2026-07-02):** scripted agent workflow for STEP → tet mesh → structural/thermal/EM solve in PDE Toolbox (capability, not a customer case).
4. **COMSOL blog by Andreas Bick, 2026-09-18 (idx):** a Codex agent tunes a nonlinear solver to speed up a CFD simulation, using a custom skill calling the COMSOL Java API on a COMSOL server, **not** MCP.
5. **COMSOL blog and webinar, 2026-08-27 (idx):** COMSOL SVP Bjorn Sjodin with Cosmon CEO Rui Aguiar on agentic workflows (Chatbot window and Nexus agent).
6. **COMSOL-MCP paper (Neurocomputing, 2026):** an agent ran model creation, geometry, meshing, solving and result extraction without the GUI (idx).
7. **Radia / Cubit pipeline (K. Sugahara):** Claude drives build123d → STEP → Cubit hex mesh → `.vol`/`.msh`/`.bdf`, with a "test-then-reflect" safety pattern.
8. **Autodesk:** AU2025 class above, plus AU2026 beginner and advanced APS MCP workshops on Forma data.
9. **Korean coverage:** The Elec article on the MATLAB MCP Server and Agentic Toolkit (idxno=12306, idx).

## 5. Unverified or conflicting items
- **COMSOL Version 2027 GA:** press release says "later this fall", Engineering.com headline says "released" (idx). Availability as of 2026-10-06 not confirmed. Tool list and transport unknown.
- **MCP Framework for Production Server requirements:** README says MATLAB R2025b+, a search summary says v1.2.4 (May 2026) needs R2026b. README treated as authoritative.
- **Autodesk:** server statuses and dates (incl. Revit 2027 tech preview) come only from index summaries. No product (non-sample) Fusion design or simulation MCP server found. Whether Autodesk Assistant acts as an MCP client not checked.
- **Not reached once the search budget ran out:** PTC Creo, Onshape AI Advisor, SimScale or nTop AI features, Neural Concept, PhysicsX, Monolith, and Korean-language MIDAS sources. "None found" means nothing in their GitHub orgs; it does not prove absence.
- **Blender:** official status unknown (blender.org blocked).
- **MATLAB release years:** years for v0.1.0–v0.14.0 inferred.
- **Cubit author affiliation:** "Kindai University" only from a search snippet.
- **Untested inference:** COMSOL LiveLink for MATLAB + MATLAB MCP Server could in principle let an agent script COMSOL meshing; neither vendor documents this.

## 6. Sources
- https://github.com/matlab/matlab-mcp-server (and /releases)
- https://github.com/matlab/matlab-agentic-toolkit (and /releases)
- https://raw.githubusercontent.com/matlab/matlab-agentic-toolkit/main/skills-catalog/math-and-optimization/matlab-solve-pde/SKILL.md
- …/matlab-solve-pde/references/primitives-and-import.md
- https://github.com/matlab/simulink-agentic-toolkit
- https://github.com/matlab/mcp-framework-matlab-production-server
- https://github.com/matlab-deep-learning/mcpHTTPClient
- (idx) https://blogs.mathworks.com/deep-learning/2025/11/03/releasing-the-matlab-mcp-core-server-on-github/
- (idx) https://www.mathworks.com/matlabcentral/discussions/ai/884857-matlab-mcp-core-server-released-on-github
- (idx) https://blogs.mathworks.com/matlab/2026/04/13/introducing-the-matlab-agentic-toolkit/
- (idx) https://blogs.mathworks.com/simulink/2025/12/10/simulating-a-simulink-model-through-github-copilot-and-the-matlab-mcp-server/
- (idx) https://blogs.mathworks.com/deep-learning/2025/12/10/matlab-mcp-client-on-github/
- (idx) https://www.mathworks.com/matlabcentral/fileexchange/182578-mcp-framework-for-matlab-production-server
- (idx) https://www.thelec.net/news/articleView.html?idxno=12306
- (idx) https://www.comsol.com/press-release/system-level-modeling-and-agentic-ai-take-the-spotlight-in-comsol-multiphysics-version-2027-14722
- (idx) https://www.globenewswire.com/news-release/2026/09/16/3363286/0/en/system-level-modeling-and-agentic-ai-take-the-spotlight-in-comsol-multiphysics-version-2027.html
- (idx) https://www.engineering.com/comsol-multiphysics-v2027-released-for-systems-modeling/
- (idx) https://www.comsol.com/blogs/using-agentic-workflows-with-comsolmph
- (idx) https://www.comsol.com/blogs/agentic-ai-within-the-simulation-engineering-space
- (idx) https://www.comsol.com/partners-consultants/software/cosmon
- (idx) https://www.comsol.com/release/6.4/comsol-desktop
- (idx) https://www.comsol.com/release/6.3/application-builder
- (idx) https://www.comsol.com/press-release/comsol-speeds-simulation-with-expanded-nvidia-gpu-support-for-comsol-multiphysics-version-64-14612
- https://github.com/wjc9011/COMSOL_Multiphysics_MCP
- (idx) https://www.sciencedirect.com/science/article/abs/pii/S0925231226018795
- https://github.com/garbage-enzyme/COMSOL_Multiphysics_MCP_6_4_Calibrated
- https://github.com/Howard-Lai/COMSOL_MCP
- (idx) https://www.digitalengineering247.com/article/coreform-cubit-2026.6-released
- https://pypi.org/project/cubit-mesh-export/
- https://pypi.org/project/radia-mcp/
- https://github.com/ksugahar/Radia
- https://github.com/AutodeskFusion360/FusionMCPSample
- (idx) https://www.autodesk.com/solutions/autodesk-ai/autodesk-mcp-servers
- (idx) https://www.autodesk.com/autodesk-university/es/class/Equip-AI-with-Access-to-Your-Design-Data-and-Projects-Model-Context-Protocol-Autodesk-Platform-Services-2025
- https://github.com/orgs/autodesk-platform-services/repositories?q=mcp
- https://github.com/autodesk-platform-services/au2026-mcp-workshop-advanced
- https://github.com/onshape-public/onshape-public.github.io
- https://github.com/hedless/onshape-mcp
- https://github.com/Joe-Spencer/fusion-mcp-server
- https://github.com/AuraFriday/Fusion-360-MCP-Server
- https://github.com/SimScaleGmbH
- https://github.com/nTopology
- https://github.com/luminarycloud/tutorials
- https://pypi.org/project/luminarycloud/
- https://github.com/rescale
- https://github.com/quanscient
- https://github.com/MIDASIT-Co-Ltd
- https://github.com/midasit-dev
- https://github.com/midas-rnd
- https://pypi.org/project/midas-civil/
- https://pypi.org/project/midas-gen/
- https://pypi.org/project/midas-mcp/
- https://github.com/mcneel/RhinoAI (and /releases)
- https://github.com/ahujasid/blender-mcp
