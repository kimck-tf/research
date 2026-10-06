# 05. Open-source meshing/FE tools and MCP registry sweep (as of 2026-10-06)

> 서브 에이전트 원본 산출물 (영문, 감사 추적용 보존). 최종 보고서는 상위 폴더의 보고서 파일 참조.

**How this was gathered, and its limits.**
- **Primary sources:** GitHub READMEs and git history (read through partial clones), the PyPI JSON API, and the official MCP Registry API (`registry.modelcontextprotocol.io/v0.1/servers?search=`). The registry search matches substrings of server names only; run for about 70 terms.
- **Blocked:** the egress proxy blocked Smithery, Glama, mcp.so, PulseMCP, LobeHub, mcpservers.org, arXiv, gmsh.info, gitlab.onelab.info, gitlab.kitware.com, projects.blender.org, and the FEniCS and CalculiX Discourse forums. Listings on those sites are known only from search snippets.
- **Search budget:** the shared WebSearch budget (200 calls across all agents) ran out partway through. The deal.II, CadQuery and Blender-official web searches never ran; covered through org-repo checks and PyPI instead.
- **Star counts** are from the GitHub pages on 2026-10-06.

**Bottom line.**
- None of these meshing/FE projects publishes an official MCP server: Gmsh, FreeCAD, Salome, Netgen/NGSolve, MeshLab, TetGen/fTetWild, CalculiX/PrePoMax, Elmer, FEniCS, MOOSE, deal.II, OpenFOAM, SU2. The "mcp" filter on each project's GitHub org returned nothing; Gmsh, OpenFOAM and Code_Aster are only partly checked (see the Unverified section).
- Only three servers are hosted by a project's own org, and none of them meshes: PyVista (a toy demo), Kitware's `vtk-mcp` (VTK API knowledge) and CadQuery's contrib server (CAD only).
- Every MCP server that can actually mesh is community-built. Most date from 2026 and most have fewer than 20 stars.
- The most capable meshing MCP servers found drive **commercial** pre-processors (HyperMesh, ANSA). They all come from one community repo, `Cai-aa/CAE-Agent-Hub`.

## 1. Summary table: open-source tools

| Tool | MCP server(s) | Official? | Dates | Meshing capability through MCP | Maturity |
|---|---|---|---|---|---|
| **Gmsh** | `OFFTECH/gmsh-mcp-server`; radia-mcp `mcp-server-gmsh`; `rishabh10gpt/gmesh-mcp`. Gmsh is also used inside mcp-calculix, mcp-fea, su2-mcp, FEP-Agent-Hub and tessalabs freecad-mcp | COMMUNITY | OFFTECH: one commit, 2026-09-23. radia-mcp 2.0.1: 2026-10-03. gmesh-mcp: 2025-03-13→03-17 | **OFFTECH (36 tools):** OCC primitives and booleans, transfinite box, O-H cylinder and 5-block pipe templates, grading, opt-in tetrahedra, minSICN/minDetJac histograms, MSH 2.2 export, gmshToFoam + checkMesh. **radia:** script lint and audit only | OFFTECH: new, 0★. gmesh-mcp: toy, 5★, unclear whether it is really MCP |
| **FreeCAD FEM** | `neka-nat/freecad-mcp`; `gchen19/AnkusDrive`; `tessalabs-space/freecad-mcp`. CAD-only: blwfish, ghbalf/freecad-ai, theosib, contextform, bonninr | COMMUNITY (FreeCAD org has 0 MCP repos) | neka-nat: 2025-03-16→2026-10-06, PyPI 0.1.25 (2026-09-24), FEM tool added 2026-05-08. AnkusDrive: 2026-04-23→10-02, PyPI 0.5.6. tessalabs: 2026-04-17→09-24 | **neka-nat:** `run_fem_analysis` runs CalculiX on an existing analysis; no meshing tool. **AnkusDrive:** 280+ tools; Gmsh/Netgen meshing; `fem_run`/`fem_modal`/`fem_buckling`; CalculiX, Elmer and Code_Aster decks; OpenFOAM/SU2. **tessalabs:** defeaturing, mid-surface, imprint/merge, Gmsh mesh export to UNV/INP/MED/VTK/BDF, named boundary-condition face groups | neka-nat is serious (2.7k★). AnkusDrive 7★, tessalabs 5★ |
| **Salome / Code_Aster** | `gnshb/salome-mcp` | COMMUNITY | 2026-03-03→03-18 | GEOM: primitives, booleans, partitions, groups. SMESH: Netgen 1D-2D-3D, compute, statistics, mesh import/export. Also `execute_salome_code` | Prototype (12★, 9 commits). No Code_Aster MCP found |
| **Netgen/NGSolve** | radia-mcp `radia-ngsolve` (Kindai University) | COMMUNITY | 2026-10-02/03 | NGSolve electromagnetics/multiphysics helpers checked against closed-form solutions; reads `.vol`/`.msh` | Niche academic |
| **PyVista** | `pyvista/pyvista-mcp-server` | **OFFICIAL** (PyVista org); no release | Created 2025-04-16; last human commit 2025-06-17 | None: a single `hello_world` tool that writes HTML | Toy, 7★ |
| **VTK (Kitware)** | `Kitware/vtk-mcp` | **OFFICIAL** (Kitware org); not on PyPI | 2025-07-23→2026-08-12 | None: VTK class lookup, documentation search, `validate_vtk_code` | Experimental, 7★ |
| **ParaView** | `LLNL/paraview_mcp`; `failed33/paraview-mcp`; iowarp `clio-kit` paraview server; FEP-Agent-Hub | COMMUNITY. No Kitware ParaView MCP in Kitware's GitHub org | LLNL: 2025-06-18→2026-04-22, PyPI 0.1.1 (2025-09-26). failed33: 0.2.4 (2026-09-30). clio-kit: 2.2.3 (2026-06-09) | Post-processing only | LLNL is a research prototype (68★, IEEE VIS 2025). failed33 3★ |
| **MeshLab** | `Georges999/MeshLab-mcp` | COMMUNITY | 2025-06-04→06-05 | pymeshlab operations behind FastAPI and a TCP bridge | Toy, 5★ |
| **TetGen / fTetWild / meshio** | None found (wildmeshing org has 0 MCP repos) | — | — | meshio is used internally by viznoir and the CAE-Agent-Hub CalculiX server | — |
| **CalculiX / PrePoMax** | CAE-Agent-Hub `MCP/CalculiX`; `Casys-AI/mcp-calculix`; `benchwire/mcp-fea`; `tianyuzong/strength-studio`; the FreeCAD servers above | COMMUNITY. No PrePoMax MCP found | Hub: 2026-08-08→08-22. mcp-calculix: 2026-07-30→10-05 (JSR 0.8.6). mcp-fea: 2026-07-29→07-31. strength-studio: 2026-09-22→09-29 | **Hub:** parse, edit and run `.inp` decks, read `.dat`. **mcp-calculix:** STEP → Gmsh tetra mesh → ccx, faces picked by bounding box. **mcp-fea:** curvature-adaptive C3D10 mesh. **strength-studio:** Gmsh + CalculiX through 19 MCP tools | Well engineered but very new; 0–1★ apart from the hub |
| **Elmer** | FEP-Agent-Hub (17 Elmer tools); AnkusDrive; tessalabs `.sif` export | COMMUNITY | 2026-08-13→08-15 | Gmsh → ElmerGrid → SIF → solve | Comes with validated benchmarks; new |
| **FEniCS/FEniCSx** | `ekstanley/ccFenics-plugin` (dolfinx-mcp); `haoming-luo/agentfem-mcp` | COMMUNITY | ccFenics: 2026-02-06→03-17. agentfem-mcp: registry 0.1.0 (2026-09-14) | Built-in meshes, Gmsh import, refinement, quality metrics, boundary tags; XDMF/VTK export | Early (5★ / 0★) |
| **MOOSE** | None (idaholab org has 0 MCP repos). The MOOSEnger paper describes an MCP execution backend | — | arXiv 2603.04756 (March 2026) | — | Research only |
| **deal.II** | None (dealii org has 0 MCP repos) | — | — | — | — |
| **OpenFOAM** | `csml-rpi/Foam-Agent` (`foamagent-mcp`); `webworn/openfoam-mcp-server`; `ymg2007/openfoam-mcp`; OFFTECH (Gmsh → gmshToFoam) | COMMUNITY. No Foundation or ESI server found | Foam-Agent: 2025-02-21→2026-09-20. webworn: 2025-07-06→2026-01-18 | **Foam-Agent:** "standard/Gmsh/custom meshes"; tools `plan`, `input_writer`, `run`, `review`, `apply_fixes`, `run_case`, `visualization`. **webworn:** `assess_mesh_quality`, `analyze_stl_geometry` (snappyHexMesh readiness) | Foam-Agent is the most serious (338★). webworn 121★, rates itself "75% Functional" |
| **SU2** | `cmudrc/su2-mcp` | COMMUNITY (Carnegie Mellon Design Research Collective) | 2025-11-29→2026-10-05; PyPI 0.1.0 (2026-03-05) | `generate_mesh_from_step` (via Gmsh), refinement knobs, element counts | Early, 4★ |
| **CadQuery** | `CadQuery/cadquery-contrib/mcp-server` (PyPI `cadquery-mcp`) | **OFFICIAL org, but a contrib repo**; v0.1.0 | Pull request #26 merged 2026-01-16; PyPI 2026-01-18 | None: render, inspect, export (STEP, STL, DXF, 3MF and others) | Experimental |
| **build123d** | `pzfreo/build123d-mcp` | COMMUNITY | 2026-04-30→09-25 (90 releases) | None: CAD, exports STEP/STL | Active, 106★ |
| **Blender** | `ahujasid/blender-mcp` (now PyPI `mcp-for-blender`) | COMMUNITY: "third-party integration and not made by Blender" | 2025-03-07→2026-10-06 | Not applicable | 30.1k★; not a CAE tool |

## 2. Evidence (short quotes)

- **OFFTECH gmsh-mcp-server**
  - It does not handle general geometry: "It does not automatically decompose arbitrary CAD into structured blocks… Arbitrary CAD import, automatic block decomposition, general 3D layer inflation… are not implemented."
  - On conversion: "Foundation OpenFOAM 13 under WSL Ubuntu 24.04 is the initially validated target."
  - github.com/OFFTECH/gmsh-mcp-server, 2026-09-23.
- **radia-mcp / cubit-mesh-export**
  - It claims to be the "First-and-only public Model Context Protocol (MCP) server suite for Gmsh, build123d…", which the servers in section 1 contradict.
  - The Gmsh tools are guidance only: `lint_gmsh_script`, `gmsh_audit_summary`, `*_remediation_plan`.
  - The Cubit server (`mcp-server-cubit`): "`any_step_to_cubit_hex` accepts STEP from any upstream MCP". Exports include `nastran_bdf`. Windows, Python 3.12 and Cubit 2025.12 only.
  - pypi.org/project/radia-mcp (2026-10-03); pypi.org/project/cubit-mesh-export (2.1.4, 2026-10-01).
- **neka-nat/freecad-mcp**
  - `run_fem_analysis` is described as "Run CalculiX on an existing analysis and return summary results."
  - Added by Pull request #54 on 2026-05-08 (from `docs/tools.md` and the git log).
- **LLNL/paraview_mcp**
  - Stability warning: it "relies on synchronization between pvserver and the ParaView client… deprecated in most recent ParaView versions… stability issues may occur."
- **Kitware/vtk-mcp:** "A thin MCP gateway that exposes vtk-knowledge, vtk-index, and vtk-validate as Model Context Protocol tools."
- **pyvista-mcp-server:** "The `hello_world` tool exports an HTML file named `a_basic.html`."
- **Foam-Agent**
  - "Foam-Agent exposes its full CFD workflow as an MCP server" (stdio, or HTTP on port 7860 in Docker).
  - On its FoamBench score: "110 simulation tasks… 100% success rate with Claude Opus 4.6; this is a benchmark result, not a guarantee."
- **mcp-fea** (registry v1.0.0, 2026-07-31): it "rejects bad setups deterministically before any compute is spent, meshes and solves with CalculiX on Modal"; about $0.002 per study.
- **Casys mcp-calculix:** "Gmsh creates the tetrahedral mesh"; it does not "infer loads or materials, approve a design".
- **CAE-Agent-Hub CalculiX server:** "ccx exit code is untrusted"; "Never `meshio.write` — it drops every card and rewrites B31→B31H".
- **salome-mcp:** needs SALOME 9.x; the bridge listens on localhost:1234; meshing uses "Netgen 1d 2d 3d".

## 3. Registry sweep

**Coverage, directory by directory**

- **Official MCP Registry.** Relevant hits:
  - `io.github.blwfish/freecad-mcp` 8.2.3 (2026-10-04)
  - `io.github.failed33/paraview-mcp-server` 0.2.4 (2026-09-30)
  - `io.github.iowarp/paraview-mcp` 2.2.3 (2026-06-09)
  - `io.github.haoming-luo/agentfem` 0.1.0 (2026-09-14)
  - `io.github.TheRoboMaster123/mcp-fea` 1.0.0 (remote, streamable HTTP)
  - `io.github.pzfreo/build123d-mcp` 0.3.90
  - `io.github.vorobjewsen30-max/ansys-mcp-server` 1.0.0 (2026-07-08)
  - `io.github.vegetableno1/dualsphysics` 0.2.0 (SPH solver)
  - `io.github.kobzevvv/moldsim-mcp` (injection-moulding knowledge)
  - `io.github.CSOAI-ORG/meek_cfd_thermal_mcp` (repo could not be cloned)
  - **No entries at all** for: gmsh, openfoam/foam, abaqus, comsol, hypermesh/altair/hyperworks/optistruct/radioss, ANSA, ls-dyna/dyna, nastran/patran/femap, calculix/ccx, salome, code_aster, elmer, fenics, moose, su2, cadquery, netgen/ngsolve, meshlab, pyvista, vtk.
  - Ansys's own PyAnsys MCP servers are **not registered** either.
- **punkpeye/awesome-mcp-servers** (last commit 2026-09-26): only AnkusDrive, agentfem-mcp, viznoir, build123d-mcp, blender-mcp, plus two DEM servers (pfc-mcp, yade-mcp). No CAE category.
- **appcypher** (2026-05-06) and **wong2** (2026-07-13): no CAE or meshing entries.
- **kimimgo/awesome-ai-cae** (2026-09-28): only 3 MCP servers (LLNL ParaView, viznoir, webworn). Its author also maintains viznoir.
- **whut09/Awesome-AI-Engineering-Agents** (2026-10-06): AgentFEM, neka-nat freecad-mcp, ParaView-MCP, viznoir, `yangkunyi/creo-mcp`.
- **`mhr2027r-dotcom/awesome-mcp-servers-cfd`** is a false lead: a fork of the appcypher list with no CFD entries.
- **No "awesome-cae-mcp" list was found** (unverified; search budget ran out).
- **Listings seen only in search snippets:**
  - Glama: OFFTECH gmsh (including a `mesh_quality` tool page), salome-mcp, KasperHonore/Foam-Agent, FEP Agent Hub, MeshLab-mcp, pyvista. FreeCAD variants from mario-2015 and maximedns5 expose `run_fem_analysis`.
  - LobeHub: llnl-paraview_mcp, tessalabs freecad-mcp, ccFenics.
  - mcp.so: freecad-mcp, pyvista-mcp-server, a meshlab tag page.
  - PulseMCP: neka-nat-freecad.
  - mcpservers.org: LLNL ParaView, CAE-Agent-Hub CalculiX, bonninr/freecad_mcp.

**Servers for commercial tools.** All COMMUNITY unless marked otherwise.

| Server | Target tool | Author | Dates | ★ | Transport | Pre-processing tools | Maturity |
|---|---|---|---|---|---|---|---|
| CAE-Agent-Hub `MCP/HyperWorks` v0.10.0 | HyperMesh, HyperView, OptiStruct, Radioss | Cai-aa | 2026-07-16→08-18 | 998 (whole repo) | stdio, plus a token-authenticated bridge inside HyperMesh | STEP/IGES/Parasolid import; `automesh_live_surfaces`; `solid_map_live_solids`; `tetra_mesh_live_solids`; `get_live_mesh_quality` and `repair_live_mesh_quality`; automatic `.hm` checkpoints with rollback; RBE3, welds, solver cards; screened Tcl batch scripts | The most capable HyperMesh MCP found. Windows only; uses a licence |
| CAE-Agent-Hub `MCP/ANSA` v0.5.1 | BETA CAE ANSA | Cai-aa | 2026-09-26 | (hub) | stdio, plus an HMAC-authenticated bridge | 18 tools covering 24 operations: geometry repair, topology paste, perimeter length, surface mesh, TETRA FEM/RAPID/CFD volume mesh, quality criteria and repair, MAT1/*MATERIAL, CLOAD, BOUNDARY, contact pairs, Abaqus/Nastran deck import and export | Validated only on ANSA 25.1.2 under Windows |
| CAE-Agent-Hub `MCP/Abaqus` | Abaqus/CAE | Cai-aa | 2026-05-19→05-25 | (hub) | stdio, plus TCP on port 48152 | `run_python` in the Abaqus kernel, `get_model_info`, `submit_job`, `inspect_odb`, viewport capture | Active |
| `Whfkl/Abaqus-Control-MCP` | Abaqus/CAE (Python 3) | Whfkl | 2026-05-10→09-29 | 200 | stdio, plus TCP | 6 tools that act on the live `mdb` | The upstream of the hub's Abaqus bridge |
| `rutwikg/abaqus-mcp` | Abaqus/Standard | R. Gulakala | PyPI 0.3.0 (2026-09-17) | 2 | stdio | 13 tools: JSON spec → STEP/IGES import + auto-mesh → `.inp` → solve, with automatic deck fixes and mesh-seed refinement | AGPL licence; the Abaqus kernel side runs Python 2.7 |
| CAE-Agent-Hub COMSOL and Ansys servers (Workbench/Mechanical, Fluent, AEDT) | COMSOL, Ansys | Cai-aa | 2026-05-19→09-30 | (hub) | stdio | COMSOL via MPh; in Mechanical, sizing + `mechanical_mesh_and_validate_tool` | Active |
| `vorobjewsen30-max/ansys-mcp-server` | Ansys | "Ansys MCP Server Contributors" | 2026-07-08 | no repo | stdio | Says it has "24 tools" for CFD, FEA and meshing | Cannot be verified |
| `cubit-mesh-export` | Coreform Cubit | K. Sugahara | 0.1.0 (2026-04-03) → 2.1.4 (2026-10-01) | 13 | stdio | Hex meshing that tries auto → sweep → polyhedron → tetmesh with automatic webcut; dry-run batch first; export to `.vol`, `.msh`, `.vtk`, `.bdf` | Academic |
| `yangkunyi/creo-mcp` | PTC Creo | yangkunyi | 2025-07-31 | ? | — | Opens STEP files, runs Creo-side Python | Alpha |

Vendor-official servers that turned up along the way (outside this slice) are all **[OFFICIAL]** and all **absent from the official registry**:
- `ansys/pymapdl-mcp` (`ansys-mapdl-mcp` 0.3.1, 2026-09-28)
- `pymechanical-mcp` (0.2.1)
- `pyfluent-mcp` (0.5.0)
- `pycfx-mcp`

Not MCP servers:
- `dsh-cae-agent` (npm): a DeepSeek Harness plugin that drives Abaqus. **[AI-ASSISTANT-NO-MCP]**
- `@2p1c/comsol-ai`: an agent skill for COMSOL. **[AI-ASSISTANT-NO-MCP]**
- MooseAgent. **[AI-ASSISTANT-NO-MCP]**
- `ghbalf/freecad-ai` (548★) is both an MCP server and an **[MCP-CLIENT]**, but has no FEM tools.

## 4. Use cases and demos

1. **HyperMesh meshing loop** (HyperWorks server): import CAD → automesh at an explicit element size → tetmesh → quality check → smoothing with before/after evidence, rolling back if it fails. Ships verified OptiStruct static and Radioss impact templates. The crash-box and tube-crush templates are explicitly "gated".
2. **ANSA → Abaqus/Nastran deck.** The validation run fixed an aspect-ratio failure at threshold 3: `native_fix` failed, `smooth_shells` passed.
3. **Abaqus self-correcting loop** (rutwikg): patches the deck; when the failure is mesh-related, an outer loop refines the seed and rebuilds the mesh.
4. **STEP plus a plain-English question → Gmsh → CalculiX** (mcp-fea). Self-reported validation: cantilever tip deflection 0.02% error, NAFEMS LE10 1.41%, first frequency 0.08%; 9.3 s with a warm container.
5. **FreeCAD**
   - neka-nat: flange design demo; `run_fem_analysis` returns max von Mises stress and max displacement.
   - tessalabs example prompt: "Mesh the housing at 3 mm, quadratic, and export the mesh as UNV… tag the +Z face as a radiator… export for Elmer."
6. **Salome:** example prompts build fused cylinders with surface groups and a Netgen mesh, and partition a NACA-airfoil wind tunnel.
7. **OpenFOAM:** Foam-Agent `/foam Simulate lid-driven cavity flow at Re=1000`; OFFTECH's structured-duct demo → gmshToFoam → checkMesh.
8. **Cubit:** a build123d helix coil becomes 1,668 hexes and 0 tets in about 30 s.
9. **FEP-Agent-Hub:** 49 of 49 tools exercised. Its transient-heat benchmark matched the analytic value within 0.318 K; beam and Poiseuille-flow results were checked against analytic values.
10. **su2-mcp:** an aircraft STEP file meshed with Gmsh and solved in SU2 (Euler), refining the mesh until results plateau.
11. **ParaView-MCP:** a multimodal model watches the viewport and refines the visualisation (IEEE VIS 2025 paper with a YouTube demo).

## 5. Unverified or conflicting items

- **Primary sites not reachable**, so "no official server" is not fully confirmed for these: Gmsh (ONELAB GitLab), ParaView (Kitware GitLab), Code_Aster (gitlab.com), ESI OpenFOAM (develop.openfoam.com), Blender (projects.blender.org).
- **KasperHonore/Foam-Agent:** listed on Glama as "15 mechanical tools", but the GitHub repo returned 404 on 2026-10-06.
- **Code2MCP-FoamAgent:** a Hugging Face Space; not checked.
- **MOOSEnger** (arXiv 2603.04756): authors' affiliation and code release not verified.
- **Forum threads not readable:** FEniCS Discourse "Introducing MCP for Fenics", and the CalculiX Discourse threads "AI MCP server… B32R" and "AI CAD/CAE co-pilot".
- **`theosib/FreeCAD-MCP-Server`:** a snippet claims FEM support; not verified.
- **Star counts disagree over time:** neka-nat had 1,709★ in a blog post versus 2.7k★ now; CAE-Agent-Hub had 938★ in a snippet versus 998★ now.
- **Claims that conflict with the evidence:**
  - A snippet calls pyvista-mcp-server "actively maintained", but it has had no human commits since 2025-06-17.
  - radia-mcp's "first-and-only" claim is contradicted by the other Gmsh-related servers above.
  - `rishabh10gpt/gmesh-mcp` looks like a web/API tool rather than a server built on the MCP SDK.
- **`meek_cfd_thermal_mcp`** and **`vorobjewsen30-max/ansys-mcp-server`** have no source code that could be inspected.
- **Commercial-tool coverage is incomplete:** no dedicated community MCP found for LS-DYNA, Nastran or Simcenter beyond the servers above, but the search budget ran out before this could be checked properly.

## 6. Sources

- **Registries and lists:**
  - https://registry.modelcontextprotocol.io/v0.1/servers?search=
  - https://github.com/punkpeye/awesome-mcp-servers
  - https://github.com/appcypher/awesome-mcp-servers
  - https://github.com/wong2/awesome-mcp-servers
  - https://github.com/kimimgo/awesome-ai-cae
  - https://github.com/whut09/Awesome-AI-Engineering-Agents
  - https://github.com/mhr2027r-dotcom/awesome-mcp-servers-cfd
- **Servers for commercial tools:**
  - https://github.com/cai-aa/CAE-Agent-Hub (MCP/HyperWorks, MCP/ANSA, MCP/Abaqus, MCP/CalculiX, MCP/COMSOL, MCP/Ansys, MCP/FEP-Agent-Hub)
  - https://github.com/Whfkl/Abaqus-Control-MCP
  - https://github.com/rutwikg/abaqus-mcp
  - https://pypi.org/project/ansys-mcp-server/
  - https://github.com/yangkunyi/creo-mcp
  - https://pypi.org/project/ansys-mapdl-mcp/
  - https://pypi.org/project/ansys-mechanical-mcp/
  - https://pypi.org/project/ansys-fluent-mcp/
- **Gmsh, Cubit, NGSolve:**
  - https://github.com/OFFTECH/gmsh-mcp-server
  - https://github.com/rishabh10gpt/gmesh-mcp
  - https://pypi.org/project/radia-mcp/
  - https://pypi.org/project/cubit-mesh-export/
  - https://github.com/ksugahar/Radia
- **FreeCAD:**
  - https://github.com/neka-nat/freecad-mcp (and its docs/tools.md)
  - https://github.com/gchen19/AnkusDrive
  - https://github.com/tessalabs-space/freecad-mcp
  - https://github.com/blwfish/freecad-mcp
  - https://github.com/ghbalf/freecad-ai
- **Salome:** https://github.com/gnshb/salome-mcp
- **CalculiX:**
  - https://github.com/Casys-AI/mcp-calculix
  - https://github.com/TheRoboMaster123/mcp-fea
  - https://github.com/tianyuzong/strength-studio
- **FEniCS and Elmer:**
  - https://github.com/ekstanley/ccFenics-plugin
  - https://github.com/haoming-luo/agentfem-mcp
  - https://github.com/S2mon123/FEP-Agent-Hub
- **OpenFOAM:**
  - https://github.com/csml-rpi/Foam-Agent
  - https://github.com/webworn/openfoam-mcp-server
  - https://github.com/ymg2007/openfoam-mcp
- **SU2:** https://github.com/cmudrc/su2-mcp
- **CAD:**
  - https://github.com/CadQuery/cadquery-contrib/tree/master/mcp-server
  - https://pypi.org/project/cadquery-mcp/
  - https://github.com/pzfreo/build123d-mcp
- **Visualisation:**
  - https://github.com/LLNL/paraview_mcp
  - https://pypi.org/project/paraview-mcp/
  - https://github.com/failed33/paraview-mcp
  - https://github.com/iowarp/clio-kit
  - https://github.com/kimimgo/viznoir
  - https://github.com/Kitware/vtk-mcp
  - https://github.com/pyvista/pyvista-mcp-server
- **MeshLab:** https://github.com/Georges999/MeshLab-mcp
- **Blender:** https://github.com/ahujasid/blender-mcp
- **Org checks for MCP repos** (all returned none except Kitware and PyVista): https://github.com/orgs/{FreeCAD,SalomePlatform,NGSolve,FEniCS,ElmerCSC,idaholab,dealii,su2code,CadQuery,cnr-isti-vclab,wildmeshing,blender,Kitware,pyvista}/repositories?q=mcp
- **Snippet-only sources:**
  - https://glama.ai/mcp/servers/OFFTECH/gmsh-mcp-server
  - https://glama.ai/mcp/servers/KasperHonore/Foam-Agent
  - https://lobehub.com/mcp/llnl-paraview_mcp
  - https://www.pulsemcp.com/servers/neka-nat-freecad
  - https://mcpservers.org/servers/github-com-cai-aa-cae-agent-hub-tree-main-mcp-calculix
  - https://fenicsproject.discourse.group/t/introducing-mcp-for-fenics/19542
  - https://arxiv.org/abs/2603.04756
