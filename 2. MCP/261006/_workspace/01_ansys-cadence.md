# 01. MCP status for Ansys (Synopsys) and Cadence (BETA CAE), as of 2026-10-06

> 서브 에이전트 원본 산출물 (영문, 감사 추적용 보존). 최종 보고서는 상위 폴더의 보고서 파일 참조.

**How this was checked, and its limits.** Ansys findings were verified from vendor-owned primary sources: PyPI package metadata (JSON API) and the code and docs of repos in the official `github.com/ansys` org (shallow-cloned and read).

The sandbox egress proxy blocked these vendor sites: ansys.com, ansys.synopsys.com, news.synopsys.com, synopsys.com, developer.ansys.com, `*.docs.pyansys.com`, innovationspace.ansys.com, cadence.com, community.cadence.com and beta-cae.com. glama.ai and teratec.eu were also blocked. **The web-search budget shared by all agents ran out before the Cadence/BETA-specific searches could run.** Because of that:
- Synopsys press statements below come only from search-engine snippets.
- The Cadence/BETA conclusion "no official MCP" has lower confidence (see §5).

## 1. Summary table

| Product | Vendor | MCP status | Server + URL | Date / version | Meshing-relevant capabilities | Class |
|---|---|---|---|---|---|---|
| Ansys Mechanical | Ansys/Synopsys | Open source, PyPI "Development Status :: 3 - Alpha" | PyMechanical-MCP `ansys-mechanical-mcp`, github.com/ansys/pymechanical-mcp | 0.1.dev0 2026-06-25; 0.1.0 2026-07-13; 0.2.1 2026-09-17 | No dedicated mesh tool. Geometry, mesh, material and BC steps run through `run_python_script`, guided by `get_guidelines_for("meshing")`. Dedicated tools: `solve_analysis`, `export_results`, `screenshot` | [OFFICIAL] alpha/preview |
| Ansys MAPDL | same | Alpha | PyMAPDL-MCP `ansys-mapdl-mcp`, github.com/ansys/pymapdl-mcp | 0.2.0 2026-06-09 → 0.3.1 2026-09-28 | Any APDL through `run_mapdl_command(s)` and `run_python_code` (ET/MP/ESIZE/VMESH…); `resume_model`, `open_results`, plots | [OFFICIAL] alpha |
| Ansys Fluent incl. Fluent Meshing | same | Alpha | PyFluent-MCP `ansys-fluent-mcp`, github.com/ansys/pyfluent-mcp | 0.1.0 2026-07-10 → 0.5.0 2026-09-25 (meshing workflow added in 0.4.0, 2026-08-14) | Meshing-mode session (watertight, fault-tolerant and 2D workflows) through `run_code`; dedicated `mesh_quality` tool | [OFFICIAL] alpha |
| Ansys CFX | same | Alpha | PyCFX-MCP `ansys-cfx-mcp` | 0.1.0 2026-07-31 → 0.2.0 2026-08-17 | None (CFX-Pre, Solver, CFD-Post) | [OFFICIAL] alpha |
| Ansys Electronics Desktop | same | Alpha | PyAEDT-MCP `ansys-aedt-mcp` | 0.1.0 2026-07-15 → 0.2.4 2026-10-01 | Electromagnetics only | [OFFICIAL] alpha |
| Ansys Lumerical | same | Alpha | PyLumerical-MCP `ansys-lumerical-mcp` | 0.1.0 2026-07-01 | Photonics only | [OFFICIAL] alpha |
| Framework for the above | same | Alpha | `ansys-common-mcp`, github.com/ansys/pyansys-common-mcp | 0.3.0 2026-04-22 → 0.3.6 2026-10-05 | FastMCP base, persistent Python sessions | [OFFICIAL] alpha |
| Ansys Prime / PyPrimeMesh | n/a | **No official MCP** (no repo; PyPI names 404) | Community `ansys-mcp-server` wraps Prime | n/a | n/a | [COMMUNITY] only |
| SpaceClaim / Discovery (PyAnsys Geometry) | n/a | No official MCP | Community only | n/a | n/a | [COMMUNITY] |
| LS-DYNA / LS-PrePost / PyDyna | n/a | **None found**, official or substantive community | n/a | n/a | n/a | none |
| Workbench | n/a | No official MCP | Community (cai-aa, ikaros0902) | n/a | Mesh and validate tools (community) | [COMMUNITY] |
| SimAI | n/a | No official MCP | One 0-star community stub | n/a | n/a | [COMMUNITY] |
| Mechanical "Mesh Agent" | Synopsys | In-product agent, "exploratory use" | n/a | Ansys 2026 R1 (press release 2026-03-11) | Debugs and resolves meshing failures | [AI-ASSISTANT-NO-MCP] (no MCP stated) |
| Engineering Copilot / AnsysGPT (AALI platform) | Synopsys | In-product assistant | n/a | n/a | n/a | [AI-ASSISTANT-NO-MCP] as published (see the AALI note in §2.1) |
| Synopsys AgentEngineer | Synopsys | Agentic AI for chip design (EDA) | n/a | Press release 2025-09-03 (snippet) | None for FE meshing | Not CAE; MCP unverified |
| ANSA | Cadence (BETA CAE) | **No official MCP found** | Community: cai-aa "ANSA MCP" | 0.4.0/0.5.0 2026-09-25; runtime 0.5.1 2026-09-26 | Geometry repair, surface/tetra meshing, quality repair, materials, Abaqus loads/BCs/contact, Abaqus/Nastran deck I/O | [COMMUNITY] |
| ANSA (API docs) | n/a | Community | "ANSA API MCP Server" (glama listing) | Unknown | Searches API docs only | [COMMUNITY], unverified |
| META, Fidelity CFD, Cadence Reality | Cadence | None found (low confidence) | n/a | n/a | n/a | none |
| Fidelity Pointwise | Cadence | None found | Official org github.com/pointwise: 89 repos, 0 match "mcp" | n/a | Glyph / GlyphClientPython automation only | none |

## 2. Details and evidence

### 2.1 Official PyAnsys MCP servers [OFFICIAL]

**Ownership**
- PyPI author is `"ANSYS, Inc." <pyansys.core@ansys.com>` for MAPDL, Mechanical, Fluent, Common and Lumerical.
- It is `"Synopsys, Inc. and ANSYS, Inc."` for CFX and AEDT; AEDT uses `pyansys-core@synopsys.com`.
- Fluent source headers read "Copyright (C) 2026 Synopsys, Inc. and ANSYS, Inc."

**Common traits**
- All are Apache-2.0, need Python 3.12+, are in 0.x versions and are marked Alpha.
- All need a licensed local or remote Ansys install.
- They ship on their own release schedule, separate from the 2026 R1/R2 product releases.

**Inventory**
- On 2026-10-06 the org listing filtered by "mcp" showed exactly 7 repos. Stars: pyfluent-mcp 47, pyansys-common-mcp 23, pymechanical-mcp 22, pyaedt-mcp 17, pylumerical-mcp 13, pymapdl-mcp 11, pycfx-mcp 1.
- None of these are in the official MCP Registry. Searching it for "ansys" returns only one community server.
- No repos or PyPI packages for Prime, Geometry, Dyna, Workbench, optiSLang, Granta or SimAI (about 25 candidate PyPI names checked, all 404).

**PyMechanical-MCP**
- Description: "Use natural language to set up, solve, and postprocess structural, thermal, and multiphysics simulations."
- Tools: 23 in 0.2.1, and 24 on the main branch, which adds `get_results_summary`.
  - Connection: `check_mechanical_status`, `get_session_diagnostics`, `check_mechanical_installed`, `launch_mechanical`, `connect_to_mechanical`, `disconnect_from_mechanical`, `list_mechanical_instances`
  - Files: `list_files`, `upload_file`, `download_file`, `download_project`, `clear_mechanical`, `save_project`, `open_project`
  - Scripting and solve: `run_python_script`, `solve_analysis`, `get_model_info`, `export_results`
  - Visualization: `screenshot`, `create_custom_plot`, `get_mechanical_logs`
  - Other: `run_python_code`, `get_guidelines_for`
- Meshing has no dedicated tool. The topics of `get_guidelines_for` are workflow, geometry, materials, meshing ("Mesh controls, sizing, and quality"), analysis_setup, boundary_conditions, solution, postprocessing, named_selections and general.
- The meshing guideline shows `mesh.ElementSize`, `mesh.AddSizing()`, MultiZone and `GetMeshStatistics()`, and states the rule "skewness > 0.95 or aspect ratio > 50 causes issues".
- Transport: "Use STDIO for local MCP clients. Use HTTP transport for remote access in trusted networks". The gRPC connection supports auto, insecure, mTLS and WNUA modes.
- Clients: VS Code, Claude Code, Claude Desktop. The quick start launches the v261 (2026 R1) `AnsysWBU.exe -grpc`.

**PyMAPDL-MCP**
- Description: "This server enables natural language interaction with MAPDL for finite element analysis tasks."
- Tools (read from source): `check_mapdl_status`, `check_mapdl_installed`, `launch_mapdl_session`, `connect_to_mapdl`, `disconnect_from_mapdl`, `list_mapdl_instances`, `run_mapdl_command`, `run_multiple_mapdl_commands`, `run_python_code`, `upload_file`, `download_file`, `resume_model`, `open_results`, `screenshot`, `custom_plot`, `get_guidelines_for`.
- The guideline topics cover geometry, elements (ET), materials, mesh, BCs, solution and post-processing. The mesh guideline lists "`mapdl.vmesh()`, `mapdl.amesh()`… `mapdl.esize()`… `mapdl.smrtsize()`".
- Transport is stdio by default or streamable HTTP. MAPDL can run locally, remotely or in Docker.
- Agent inference (not documented): because the server accepts any APDL, a HyperMesh-exported CDB could be read with CDREAD.

**PyFluent-MCP**
- Docs: "PyFluent-MCP supports two mesh-related workflows: running Fluent meshing workflows and inspecting loaded meshes from solver sessions."
- Docs: "The API catalog includes common entries for Watertight Geometry, Fault-tolerant Meshing, and 2D Meshing workflows."
- `mesh_quality` returns cell, face and node counts, skewness, orthogonal quality, aspect ratio and `mesh.check` diagnostics.
- 20 tools in total. Transport is STDIO or streamable HTTP. Clients: VS Code Copilot, Claude Desktop, Cursor and custom agents.
- Architecture: the public package is "**solve-only**… Geometry (CAD import) and Prime meshing (readcad) handlers are NOT registered here". Higher-level orchestration is said to live in "host products such as `fluids-mcp`". `fluids-mcp` is not public.
- The backend docstring names "Fluent solver, Fluent meshing, Discovery, Prime, or the Fluids One service". That hints at Discovery and Prime MCP coverage that has not been released (agent inference).

**PyAnsys Common MCP**
- Description: "provides the infrastructure for building Model Context Protocol (MCP) servers for PyAnsys libraries."
- Its first release, 0.3.0 on 2026-04-22, is the earliest official Ansys MCP artifact found.

**Link to Ansys's own AI platform (AALI)**
- The Mechanical and MAPDL servers have an `--on-aali` flag: "To specify whether PyMechanical-MCP server is running in an AALI environment." The flag disables some tools.
- The Fluent and CFX servers reference `aali-flowkit-python` and `AALI_*` environment variables.
- A docs.aali.ansys.com snippet defines AALI as the "Ansys Actionable Language Interface", which "helps power generative AI features in Ansys products".
- Agent inference: Ansys's in-product AI stack can host these MCP servers. Not announced.

### 2.2 Ansys in-product AI that does not use MCP

**Ansys 2026 R1** (news.synopsys.com, 2026-03-11, seen as a snippet)
- Described as "the portfolio's first agentic capabilities".
- "Mesh Agent, a new feature in Ansys Mechanical™ software available for exploratory use, helps engineers debug and resolve meshing failures during model pre-processing."
- The same release added the Discovery Validation Agent and the GeomAI platform.
- The snippet does not mention MCP.

**Other items**
- **Engineering Copilot / AnsysGPT:** no statement found that they act as an MCP server or client. Vendor pages could not be opened.
- **Synopsys AgentEngineer:** snippet from the 2025-09-03 press release. Chip-design agent technology, with no evidence that it covers FE meshing.

### 2.3 Cadence / BETA CAE

No vendor MCP found for ANSA, META, Pointwise, Fidelity CFD or Reality. Checks run:
1. The one web query that ran ("ANSA BETA CAE MCP server…") returned only a community listing.
2. The official Pointwise GitHub org is labelled "Cadence Design Systems, Inc." and "Fidelity Pointwise CFD Meshing" and has 89 repos. Filtering by "mcp" returns 0. Automation exists only through Glyph and `GlyphClientPython` (updated 2024-09-11).
3. MCP Registry searches for ansa, beta-cae, pointwise, lsdyna and cadence returned no CAE servers.
4. PyPI names such as `ansa-mcp`, `pointwise-mcp` and `lsdyna-mcp` all return 404.

Cadence's agentic AI products (JedAI, ChipStack, Cerebrus) were not checked in this session; see §5.

## 3. Community / unofficial MCP servers

| Name / URL | Author | Dates | Stars | Capabilities |
|---|---|---|---|---|
| **ANSA MCP**, github.com/cai-aa/CAE-Agent-Hub (`MCP/ANSA`) | Individual "Cai-aa"; repo MIT; 998 stars / 127 forks; first commit 2026-04-27 | 0.4.0 and 0.5.0 on 2026-09-25; runtime 0.5.1 on 2026-09-26 | (repo) 998 | See the detail below this table |
| ANSYS Workbench MCP (same hub) | same | First 2026-05-19; 0.2.0 2026-09-01 | n/a | About 44 tools, including `mechanical_import_geometry_tool`, `mechanical_create_named_selection_tool`, `mechanical_mesh_and_validate_tool`, `mechanical_validate_mesh_job_tool`, `mechanical_solve_analysis_tool`, results extraction. Uses an ACT bridge; clients Codex, Claude Code, Claude Desktop. The hub also has a Fluent MCP (2026-05-28) and a mirror of the official PyAnsys MCPs (2026-08-22) |
| `ansys-mcp-server` (PyPI), github.com/vorobjewsen30-max/ansys-mcp-server | vorobjewsen30-max | 1.0.0 2026-07-08; MIT; "Beta" | n/a | The only Ansys entry in the official MCP Registry. 24 tools, including `ansys_mesh_generate` (STP/IGES/SCDOC through Prime), `ansys_mesh_refine`, `ansys_mesh_quality`, `ansys_mesh_convert` (MSH↔CDB↔VTU), `ansys_set_material`, `ansys_set_boundary_conditions`, `ansys_run_simulation`. Targets Claude Code. Claims untested |
| github.com/knewnothing-git/ansys-mcp-server | knewnothing-git | One commit, 2025-09-14 (earliest found) | 66 | Fluent and MAPDL sessions and commands, status, file reading; MIT |
| github.com/codersag/mechanical-mcp | codersag | 18 commits, all on 2026-06-23 | 19 | PyMechanical gRPC, Mechanical 2023 R1+. Mesh: "Set element size, generate mesh, get statistics"; BCs, solve, results, DOCX reports. Clients: Claude Desktop, Cursor, Windsurf, VS Code, Continue |
| github.com/Bettertoo2/ansys-mcp-server | Bettertoo2 | 2026-06-13 → 2026-08-08 | 6 | Ansys 2024 R2 Fluent, Mechanical and PyAnsys Geometry; `mechanical_get_mesh_settings`; Codex and Claude Code |
| github.com/ikaros0902/ansys-unified-mcp | ikaros0902 | 86 commits, 2026-07-02 → 2026-10-06 | 0 | Workbench, Mechanical, SpaceClaim, Fluent, optiSLang; LS-DYNA and MAPDL "coming soon"; includes agent skills (e.g. watertight meshing) |
| github.com/JEEVANANTHAMV/mcp-ansys-* | JEEVANANTHAMV | Around 2026-10-04 | 0 | Template-like repos for SpaceClaim, Discovery, MAPDL, SimAI, Granta MI and others. Not evaluated |
| "ANSA API MCP Server" (glama.ai/mcp/servers/jwynhnsypg) | Unknown | Unknown | n/a | Searches ANSA Python API docs ("2,379 functions distributed across 20 modules") to help write scripts; Chinese and English; for Claude Code. Listing only |

**ANSA MCP (cai-aa) in detail**
- How it works: the MCP client talks to an external FastMCP process over stdio. That process talks to a bridge plugin inside the ANSA GUI (installed as a `.bpkg`) over an HMAC-authenticated loopback connection on 127.0.0.1:48762.
- Platform: Windows, Python 3.10+. Tested only on ANSA 25.1.2. Clients: Codex or any stdio MCP client.
- 18 tools: `create_live_entity`, `execute_live_ansa_operation`, `execute_live_ansa_python_step` (arbitrary Python, not sandboxed), `get_environment`, `get_live_ansa_api_help`, `get_live_bridge_status`, `get_live_capabilities`, `get_live_entity`, `get_live_entity_fields`, `get_live_model_summary`, `get_live_session_info`, `get_live_step_history`, `list_live_entities`, `refresh_live_view`, `run_live_model_checks`, `save_live_database`, `search_live_ansa_api`, `set_live_entity_card_values`.
- 24 named operations, grouped:
  - Geometry: `repair_geometry`, `topology_paste`
  - Meshing: `set_perimeter_length`, `mesh_faces`, `generate_surface_mesh`, `generate_volume_mesh` (TETRA FEM/RAPID/CFD), `remesh_shells`
  - Mesh quality: `set_mesh_quality_criterion`, `repair_mesh_quality`
  - Materials and references: `create_isotropic_material`, `set_entity_references`
  - Abaqus setup: `create_nodal_load` (Abaqus CLOAD), `create_nodal_constraint` (Abaqus BOUNDARY), `create_contact_pair`
  - Deck I/O: `import_solver_deck`, `export_solver_deck` (Abaqus Standard / Nastran)
- Limits it states itself: "No solver execution or META post-processing is added here." Boundary layers and hex blocking are not included.

**For slice 03 (HyperMesh):** the same hub has a community "HyperWorks MCP" (v0.10.0, first commit 2026-07-16). It covers HyperMesh automesh, solid map, tetra meshing and quality repair, plus OptiStruct and Radioss deck generation and solve.

## 4. Use cases and demos
1. **Official PyMechanical-MCP example prompts** (repo `examples/`):
   - Plate with a hole from STEP: "Mesh with element size of 2mm", fixed support plus a 10 kN load, von Mises screenshot, expected Kt ≈ 3.
   - L-bracket modal analysis: 3 mm mesh, 6 modes.
   - Cantilever: 5 mm mesh, 1 MPa pressure.
   - Each is an end-to-end natural-language run: STEP → mesh → BC → solve → post.
2. **Official PyMAPDL-MCP demo videos** (docs "Usage examples"): the main features; an LLM answering MAPDL questions and checking them with tools (reading a CSV); automated bug fixing.
3. **Official PyFluent-MCP "Generate a mesh from geometry" workflow:**
   - `connect(mode="meshing")`
   - `find_api("workflow initialize watertight geometry")`
   - `validate_code`
   - `run_code` for geometry import, local sizing, surface mesh, boundary layers, volume mesh and checks
   - then switch to the solver
4. **Ansys Innovation Space webinar "Agentic AI for Ansys Mechanical and MAPDL Workflows"** (vendor channel, date not visible). Snippet: "Claude Code, OpenAI Codex, Cursor, GitHub Copilot… can interface with Ansys Structures products." Probably uses the servers above (agent inference).
5. **Community ANSA MCP live tests on ANSA 25.1.2** (2026-09-25):
   - Generated 16 planar shell elements and 33 tetrahedra in a box.
   - Created MAT1 through a real FastMCP client.
   - Ran Abaqus import/export and Nastran export/re-import.
   - 240 tests passed.
   - Quote: "A distorted patch had two aspect-ratio failures: native fix retained them; explicit shell smoothing made that check pass."

No third-party industrial case study (a named company with outcomes) found for Ansys or ANSA MCP.

## 5. Unverified or conflicting items
- **Unattributed Synopsys MCP quote.** A search summary quoted: "MCP enables AI agents to be users of Synopsys core products like Discovery, Mechanical, Fluent, AEDT, OptiSLang, and Granta…". Source unclear. Candidates:
  - a Teratec/NAFEMS AI seminar PDF (Luc Pontoire, 2026-06)
  - the Synopsys "Vision for Engineering the Future" press release (2026-03-11)
  - an iConnect007 article on Synopsys and NVIDIA

  None could be opened. There are no public Discovery, optiSLang or Granta MCP servers.
- **`fluids-mcp`** is referenced in the official README but is not public.
- **Internal MCP use** by Engineering Copilot or the Mesh Agent is unknown.
- **Ansys 2026 R2** AI/MCP news was not checked.
- **Cadence agentic AI** (JedAI, ChipStack AI Super Agent, Cerebrus AI Studio, Fidelity, Reality) was not checked. Neither were ANSA v25/v26 release notes or the BETA CAE conference for LLM/MCP mentions.
- **`github.com/cadence-design-systems`** exists with no public repos; Cadence ownership not verified.
- **Mechanical MCP tool count:** PyPI says 23; GitHub main says 24.

## 6. Sources
**Fetched or cloned (primary)**
- pypi.org/project/ansys-common-mcp, ansys-mapdl-mcp, ansys-mechanical-mcp, ansys-fluent-mcp, ansys-cfx-mcp, ansys-aedt-mcp, ansys-lumerical-mcp, ansys-mcp-server
- github.com/orgs/ansys/repositories?q=mcp
- github.com/ansys/pyansys-common-mcp
- github.com/ansys/pymapdl-mcp (`src/ansys/mapdl/mcp/{tools,contexts}.py`, `doc/source/examples/usage_examples.rst`)
- github.com/ansys/pymechanical-mcp (`src/.../contexts.py`, `examples/*/prompt.md`)
- github.com/ansys/pyfluent-mcp (README, `doc/source/user_guide/tools_and_capabilities.rst`, `doc/source/changelog.rst`, `src/.../common/{backend,file_handlers}.py`)
- github.com/ansys/pycfx-mcp
- github.com/pointwise and github.com/orgs/pointwise/repositories?q=mcp
- registry.modelcontextprotocol.io/v0/servers?search={ansys,ansa,beta-cae,pointwise,lsdyna,dyna,cadence,mapdl,fluent}
- github.com/cai-aa/CAE-Agent-Hub (`MCP/ANSA/{README,ENGINEERING_OPERATIONS,RELEASE_0.4.0,RELEASE_0.5.0,RELEASE_0.5.1}.md`, `MCP/Ansys/Workbench MCP`, `MCP/HyperWorks/README.md`)
- github.com/knewnothing-git/ansys-mcp-server
- github.com/codersag/mechanical-mcp
- github.com/Bettertoo2/ansys-mcp-server
- github.com/ikaros0902/ansys-unified-mcp

**Search snippets only (pages blocked)**
- news.synopsys.com/2026-03-11-Synopsys-Launches-Ansys-2026-R1-to-Re-Engineer-Engineering-with-Joint-Solutions-and-AI-Powered-Products
- news.synopsys.com/2026-03-11-Synopsys-Outlines-Vision-for-Engineering-the-Future
- news.synopsys.com/2025-09-03-Synopsys-Announces-Expanding-AI-Capabilities-for-its-Leading-EDA-Solutions
- docs.aali.ansys.com
- innovationspace.ansys.com/product/agentic-ai-for-ansys-mechanical-and-mapdl-workflows-2/
- teratec.eu/media/wp-content/uploads/2026/06/6-Luc_Pontoire_Seminaire_IA_NAFEMS_Teratec_2.pdf
- glama.ai/mcp/servers/jwynhnsypg
- github.com/JEEVANANTHAMV (mcp-ansys-* listing)
