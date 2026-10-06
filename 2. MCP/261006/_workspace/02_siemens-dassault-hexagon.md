# 02. MCP status for Siemens (excluding HyperMesh), Dassault Systèmes, Hexagon and Keysight/ESI, as of 2026-10-06

> 서브 에이전트 원본 산출물 (영문, 감사 추적용 보존). 최종 보고서는 상위 폴더의 보고서 파일 참조.

**Read this first: limits on the evidence**
- The sandbox's egress proxy blocked most vendor and press domains: siemens.com, news/blogs/community/developer/mcp.siemens.com, 3ds.com, blog.3ds.com, hexagon.com, prnewswire, lemagit and others. Vendor quotes below therefore come from **search-engine extracts of those pages, not from reading the pages directly**.
- Read directly: GitHub, the official MCP Registry (registry.modelcontextprotocol.io) and the npm registry.
- The shared WebSearch budget ran out partway through. Because of that, **Hexagon's AI/Nexus and Keysight-ESI AI announcements were checked only negatively**, through GitHub and the MCP Registry.

**Bottom line:** none of these four vendors ships an official MCP server for a meshing or pre-processing product. That covers Simcenter 3D, Femap, NX CAE, STAR-CCM+, Abaqus/CAE, 3DEXPERIENCE SIMULIA, SOLIDWORKS Simulation, MSC Apex/Patran and Visual-Mesh.
- **Siemens does run official MCP servers**, but only for documentation search, its UI design systems, Mendix low-code apps and EDA.
- **Dassault's AURA, LEO and MARIE are agents that work inside the 3DEXPERIENCE platform.** They have no public MCP interface.
- **MCP servers that can mesh in Abaqus exist only as community projects.**

## 1. Summary table

| Product | Vendor | MCP status | Server name + URL | Date / version | What it can do for meshing | Classification |
|---|---|---|---|---|---|---|
| Siemens MCP Server (docs/search) | Siemens | Live and public, no login needed | mcp.siemens.com (endpoint `https://mcp.siemens.com/mcp`, HTTP) | Launch date not found | None: searches docs, products, APIs and web content | OFFICIAL (live) |
| Xcelerator Developer Portal MCP | Siemens | Live (a plugin of the server above) | developer.siemens.com/resources/how-tos/mcp.html | Not verified | None | OFFICIAL |
| Siemens iX / Element MCP | Siemens | Released on npm, runs locally | `@siemens/ix-mcp`; developer.siemens.com/mcps/ix-mcp | Created 2025-11-13; latest 2026-09-22 | None (UI design-system docs) | OFFICIAL |
| Mendix MCP Server module | Siemens (Mendix) | Marketplace module | marketplace.mendix.com, component 240380 | Mendix 10.24.13; MCP Java SDK 0.16.0 | None directly; can expose any Mendix app logic as MCP tools | OFFICIAL |
| Fuse EDA AI Agent | Siemens EDA | Launched | news.siemens.com/en-us/siemens-fuse-eda-ai-agent | 2026-03-16 (NVIDIA GTC) | None (chip design only) | OFFICIAL + MCP-CLIENT |
| Intelligence Center X | Siemens | Product launched; MCP support not confirmed by Siemens | news.siemens.com/…/siemens-intelligence-center-x | 2026-06-01 | None | OFFICIAL product; MCP claim unverified |
| Simcenter portfolio (incl. Simcenter 3D) | Siemens | No MCP found | — | June 2026 release | Copilot only answers from the online help | AI-ASSISTANT-NO-MCP |
| Simcenter Femap | Siemens | Nothing, official or community | — | — | — | none |
| NX / "Designcenter" | Siemens | No official MCP | Community servers only (section 3) | — | Community servers are CAD-only, no CAE or meshing tools | COMMUNITY; Copilot = AI-ASSISTANT-NO-MCP |
| Simcenter STAR-CCM+ | Siemens | No official server; a community docs server plus a tutorial series on Siemens' own forum | github.com/pacoEzq/starccm-javadoc-mcp | ~2026-09-29 | Looks up the Java API, so AI-written macros (including meshing macros) use real calls | COMMUNITY |
| Teamcenter | Siemens | No official server found | github.com/goenninger-b-t/tc-mcp | 2026-03-19 | PLM data only | COMMUNITY |
| Abaqus/CAE | Dassault (SIMULIA) | No official server; more than 20 community servers | Section 3 | 2025-05 to 2026-10 | Varies: mesh seeding and generation, element types, materials, BCs/loads, jobs, ODB results | COMMUNITY |
| 3DEXPERIENCE SIMULIA + AURA/LEO/MARIE | Dassault | No public MCP; press reports Dassault uses MCP/A2A internally | — | Unveiled Feb 2026; available 2026-07-23 (R2026x FD03) | LEO sets up, runs and analyses simulations inside the platform | AI-ASSISTANT-NO-MCP (publicly) |
| SOLIDWORKS (+ Simulation) | Dassault | No official server; community servers exist but none covers Simulation | `io.github.hjbaard/solidworks-mcp` | v0.13.0, 2026-10-06 | None | COMMUNITY; AURA = AI-ASSISTANT-NO-MCP |
| CATIA V5 / CATIA Magic | Dassault | Community only | github.com/daiemon12/catia-v5-mcp-server | ~2026-10-05 | CAD only | COMMUNITY |
| MSC Apex, Patran, Marc/Mentat, MSC Nastran, Simufact, Nexus | Hexagon | Nothing found | — | — | — | none (community: one closed-source Marc assistant; pyNastran-MCP works at deck level) |
| Visual-Environment / Visual-Mesh / VPS (PAM-CRASH) | Keysight (ESI) | Nothing found | — | — | — | none (Keysight's only official MCP is for CyPerf network testing) |

## 2. Details and evidence

### Siemens (HyperMesh excluded)

**Siemens MCP Server (mcp.siemens.com)**
- From the search extract of /docs: *"The Siemens MCP Server includes 4 active plugins with 8 available tools, targeting llm, developer, and agent audiences."*
- The four plugins:
  - `siemens-search`: AI Search, Product Search, Search
  - `siemens-developer-portal`: Developer Search, Developer Discover, Developer Reference
  - `siemens-webcontent`: *"fetching and converting Siemens web content to clean markdown"*
  - `siemens-assets`: asset search
- Access: *"public access (no authentication required)"*. The Claude Desktop setup posts to `https://mcp.siemens.com/mcp`.
- An OpenAPI description is at mcp.siemens.com/openapi.json, and a dev instance runs at dev.mcp.siemens.com.
- Status reads as live; no launch date was found.
- Engineering use: Claude or Copilot can look up Siemens documentation and APIs. The server cannot drive Simcenter.

**developer.siemens.com MCP and how-to**
- The portal MCP *"unlocks AI-powered access to the developer portal… search documentation instantly, discover products and APIs."*
- A how-to page is titled "Setup Agentic AI & MCP".

**iX and Element MCP servers** (UI design systems, not engineering software)
- *"local MCP server that ships with all iX design system documentation, component APIs…"*
- Works with *"VS Code / GitHub Copilot, Cline, Zed"*.
- Requires *"Node.js 20+ and an LLM token from my.siemens.com."*
- The npm package `@siemens/ix-mcp` (MIT) was created 2025-11-13 and last published 2026-09-22. Its source lives on code.siemens.com, not GitHub.
- Siemens' public GitHub org (github.com/siemens) has **0 repos matching "mcp"**.

**Mendix MCP Server module**
- Purpose: *"make Mendix business logic available to agents in an enterprise landscape."*
- Version details: *"based on Mendix 10.24.13 and uses MCP Java SDK 0.16.0 that supports MCP Protocol version 2024-11-05 and 2025-03-26."*
- It comes with a Claude Desktop example.
- This is the officially supported way to publish your own Siemens-stack logic as MCP tools.

**Fuse EDA AI Agent (2026-03-16)**
- *"secure agentic orchestration with Model Context Protocol (MCP) and Agent Skills support, along with an open framework for third-party integration"*
- *"dynamic tool discovery and orchestration across MCP-connected EDA tools."*
- This is the clearest sign that Siemens has adopted MCP company-wide, but it covers EDA only.
- The Questa One Agentic Toolkit (2026-03-02) is related; whether it uses MCP was not verified.

**Intelligence Center X (announced 2026-06-01 at Realize LIVE Americas, Detroit)**
- Official description: *"industrial AI orchestration software"* combining *"the Mendix™ low-code platform with Siemens' Graph Studio and AI Studio software from the Rapidminer® portfolio."*
- Siemens cites customer results of *"95 percent reduction in manual effort and 85 percent faster production issue resolution."*
- On MCP, only a third-party summary says *"Integration uses standard connectors and the Model Context Protocol (MCP)"*. Siemens' own wording on this was not found.

**Simcenter June 2026 release** (first release combining Siemens and Altair tools)
- *"Copilot, an AI-powered support assistant that responds using information from Simcenter's online help and knowledge resources"*
- Also new: PhysicsAI and PhysicsAI Generate.
- No MCP was mentioned, so this is classed AI-ASSISTANT-NO-MCP.

**NX / Designcenter**
- No official MCP server.
- An integrator page (innfactory.ai, undated) says: *"No official MCP server exists from Siemens so far; community servers and documented interfaces are used instead."*
- A third-party site (inteliscience.net) claims *"Siemens renamed NX to Designcenter with the 23 June 2026 release and shipped Designcenter Copilot."* Not confirmed.
- NX CAE meshing (the NXOpen.CAE API) can only be reached through the generic NXOpen script runners in community servers.

**Femap**
- Nothing on GitHub ("femap mcp" gives 0 repos) or in the MCP Registry (0 entries).

**STAR-CCM+**
- Siemens' community forum hosts a six-part series, "Simcenter STAR-CCM+ Macros with AI". Part 2 builds an MCP server for Claude Code that *"exposes two tools: search_api… and get_doc."*
- Repo: pacoEzq/starccm-javadoc-mcp (BSD-3, Node 18+, stdio, 2 stars).
- Its disclaimer: *"Use of Siemens documentation and AI tools remains subject to your applicable Siemens terms."*
- The author's affiliation is not stated, so this is classed COMMUNITY.

### Dassault Systèmes

**No official MCP server or client found.** Vendor search extracts, GitHub, and the MCP Registry for "dassault", "3ds", "catia", "simulia" and "abaqus" were checked; none returned a vendor entry.

**Virtual Companions (AURA, LEO, MARIE)**
- Unveiled at 3DEXPERIENCE World 2026 (Houston, Feb 1–4). Press release of 2026-02-11: "Dassault Systèmes Unveils New Way of Working for Industry with AI-Powered Virtual Companions".
- Availability press release of 2026-07-23: *"availability of its Virtual Companions AURA for program management, LEO for complex engineering and MARIE for deep science"*, *"Enriched with 19 competencies."*
- The SIMULIA blog on R2026x FD03 says LEO:
  - *"helps set up simulations using your company's best practices and knowledge"*
  - covers *"setting up, running and analyzing simulation results"*
  - supports *"automating simulation setup and analysis for real industry test cases."*
- DEVELOP3D (third-party) says the companions are built *"on the Mistral AI foundational model on Dassault Systèmes' Outscale cloud"* and that *"Leo will launch in mid-2026."*
- Classed AI-ASSISTANT-NO-MCP because there is no public MCP endpoint or connector.
- LeMagIT (third-party) reports that Dassault *"began using function calls before heavily adopting MCP, and also uses Agent2Agent"* inside the platform. Unverified.

**Other Dassault items**
- 3DEXPERIENCE Agentic Process Orchestration *"brings workflows, AI agents and human review together in one auditable process model."* No MCP mention surfaced.
- The Mistral partnership was deepened on 2025-11-26: Le Chat Enterprise and AI Studio are now available on OUTSCALE. Not an MCP announcement.

**SOLIDWORKS**
- AURA launched in 2025, and LEO and MARIE are being added.
- getleo.ai (third-party, 2026): *"None of the SolidWorks MCP servers come from Dassault Systemes."*
- Note that getleo.ai's "Leo AI" is an unrelated startup, not Dassault's LEO.

### Hexagon
- No official MCP server found for MSC Apex, Patran, Marc/Mentat, MSC Nastran, Simufact or Nexus.
- MCP Registry searches for "hexagon" and "nastran" returned 0 entries.
- GitHub searches for "hexagon msc mcp" and "apex nastran mcp" returned 0 repos.
- The `hexsupport/hex-mcp` repo belongs to HexagonML, an unrelated MLOps company.

### Keysight (ESI)
- No MCP server found for Visual-Environment, Visual-Mesh or VPS. GitHub "pam-crash mcp" returned 0 repos; the MCP Registry search for "keysight" returned 0 entries.
- Keysight's only official MCP server is `Keysight/cyperf-mcp`: *"exposes Keysight CyPerf network performance and security testing"*. 100 tools, stdio, MIT-licensed © 2025 Keysight, updated 2026-04-16.

## 3. Community / unofficial MCP servers

Star counts and "updated" dates are as shown on GitHub on 2026-10-06; ~ marks dates estimated from GitHub's relative timestamps.

### Abaqus

| Repo | Stars | Updated | Transport | Capabilities |
|---|---|---|---|---|
| Cai-aa/abaqus-mcp | 287 | 2026-04-08 | File-based IPC with a CAE plugin | `execute_script`, `get_model_info` (parts, materials, steps, loads, BCs), `submit_job`, `get_odb_info`, `get_viewport_image`; MIT; Cursor and Claude Desktop |
| Whfkl/Abaqus-Control-MCP | 200 | ~2026-09-29 | stdio, plus local TCP 127.0.0.1:48152 to the CAE plugin | `run_python` on the live model database, `monitor_job_status`, `inspect_odb`, `capture_viewport`; MIT; Codex and Claude Code. Fork pyejy/…-2023 supports Abaqus 2023 (Python 2.7) |
| jianzhichun/abaqus-mcp-server | 105 | 2025-05-26 | GUI automation via pywinauto (Windows) | `execute_script_in_abaqus_gui`, `get_abaqus_gui_message_log`; MIT |
| LonlySteve/abaqus6.14mcp | 104 | 2026-05-04 | MCP (JSON) | Port for Abaqus 6.14 / Python 2.7, plus an MCP server that searches Abaqus Help and a browser viewer |
| Tomsabay/abaqus_agent | 28 | 2026-08-31 | MCP + FastAPI/SSE | Turns a spec or .inp into a solved model, with physics checks on results, a solver-error diagnoser ("Solver Doctor", 30+ failure patterns) and run-to-run diffs; AGPL-3.0; Abaqus 2021+ |
| TutuBarry/abaqus-mcp-pro | 3 | ~2026-09-17 | stdio + TCP | **Covers the most meshing:** `seed_part`, `generate_mesh`, `set_element_type` (C3D8R, S4R, …), `create_material` / `create_section`, `create_bc` / `create_load` / `create_constraint` (tie, coupling, MPC), `submit_job`, `diagnose_job`, `export_odb_to_vtk`; Abaqus 2024+; MIT |
| rutwikg/abaqus-mcp | 2 | ~2026-09-18 | stdio | `build_model`, `build_and_simulate`, `autocorrect_simulation`; repairs input decks and meshes (refines seeds and rebuilds); AGPL-3.0; Abaqus 2022+ |
| MOBAI547800/abaqus-mcp-server and abaqus-codex-mcp | 0 | 2026-07-24 / 2026-07-30 | stdio | 11 and 15 tools: generate scripts, `submit_job`, read logs, extract field/history output to CSV; MIT |

Also found:
- gakialter/abaqus-agent: Abaqus 2026, updated 2026-10-05
- YangCinto/abaqus-mcp-opencode: 11 stars
- x2274870348-hub/codex-abaqus: 9 stars
- Fisfzy/dsh-cae-agent: DeepSeek plugin

### Siemens products

**NX** (all CAD-only; none has CAE or meshing tools):
- DreamEnding/NX_MCP: 107 stars; NX 2506; 16 default tools plus 34 experimental; stdio to loopback JSON-RPC; MIT; validated 2026-09-30.
- cyw2527/nx-auto-mcp: 130+ tools across 15 domains; NX 2412; TCP port 1977; MIT.
- mingfeng6684/nxopen-mcp: 13 stars; searches the NXOpen API docs (an NX 12 index held 97,913 API members); tools `search_api`, `get_class`, `get_member`, `find_builder`.
- Cai-aa/CAD-Agent-Hub: 81 stars; covers NX, CATIA, SOLIDWORKS and Fusion.

**Teamcenter:**
- goenninger-b-t/tc-mcp: 7 stars; 30+ tools covering search, items, BOM and workflow; Teamcenter 2412+, REST (JsonRestServices/SOA); stdio.
- srgio-es/teamcenter-mcp-server: archived.

**Simcenter Amesim:**
- SometingGBBB/Simcenter-Amesim-MCP: 7 stars; updated 2026-08-13; 24+ tools (`create_circuit`, `run_simulation`, …). States it is *"not affiliated with or endorsed by Siemens."*

**Nastran input decks** (works with both Simcenter Nastran and MSC Nastran decks):
- Shaoqigit/pynastran-mcp: 3 stars; reads and writes BDF decks, `check_mesh_quality`, reads OP2 stress/displacement results; stdio, SSE or streamable-HTTP; MIT.

### SOLIDWORKS, CATIA and 3DEXPERIENCE

**SOLIDWORKS:**
- hjbaard/SolidWorks-MCP: listed in the MCP Registry as `io.github.hjbaard/solidworks-mcp`; v0.13.0, 2026-10-06; 84 tools; *"Simulation (FEA) are out of scope"*.
- andrewbartels1/SolidworksMCP-python: 80 stars; 132 tools; simulation is *"Not Yet / Simulated"*.
- eyfel/mcp-server-solidworks and VisionFocus2022/SolidWorksMCP also exist.

**CATIA:**
- daiemon12/catia-v5-mcp-server: 112 stars; 98 tools; no FEM or meshing; V5 R2016+; MIT.
- chenlei-gh/mcp-catia: 44 stars.
- rutwikg/catia-mcp: 161 tools.
- ajhcs/cameo-mcp-bridge: 41 stars; for CATIA Magic (SysML).

**3DEXPERIENCE:**
- VyomGovil/3DExperience_MCP_Integration: 0 stars; updated 2026-08-26.

### Hexagon and Keysight
- WangJ226/_Marc_-_MCP_-Assistant: closed source, 1 star, updated 2026-07-05.
- No community MCP servers for ESI products (Visual-Environment, Visual-Mesh, VPS).

## 4. Use cases and demos (all community; no vendor case study involves MCP and meshing)

1. **jinkeguo/cax-workflow-agent** (updated 2026-07-26) — the closest match to this user's own workflow. *"MCP-powered Codex agent for automated SolidWorks-to-HyperMesh-to-Abaqus workflows."*
   - 32 MCP tools.
   - Drives SOLIDWORKS through its COM automation interface, HyperMesh through Tcl in batch mode, and Abaqus through deck validation, job submission and ODB extraction.
   - Tested with HyperMesh 2025 and Abaqus 2022.
   - Each stage returns *"artifacts, structured checks, warnings, logs."*
2. **TmxjTmxj/microneedle-skin-pullout** (updated 2026-08-31) — an Abaqus/Explicit model of a barbed microneedle pulled out of three-layer skin.
   - Every step, from importing the STEP file through Python scripting, meshing (5,967 skin and 7,974 needle elements) and solving to post-processing, was *"generated and executed by Codex via MCP"*.
   - It used a third-party MCP server called "text-to-cae".
   - Result: peak pull-out force of 0.1818 N.
3. **Patrick-Miles-Z/CAD-CC-Abaqus** (updated 2026-08-30) — DXF → STEP → Abaqus, with meshing, materials, BCs and solving driven from Claude Code through two MCP servers.
4. **Tomsabay/abaqus_agent** — five benchmark cases with checked results: cantilever, plate with hole, modal, explicit impact and blast plate.
5. **Siemens forum STAR-CCM+ series** — grounds AI-written Java macros in the API docs that ship with the user's own installation, so the AI stops inventing API calls. nxopen-mcp does the same for NXOpen.
6. **Vendor-side, not MCP:**
   - LEO in SIMULIA sets up, runs and analyses simulations through natural language inside 3DEXPERIENCE (R2026x FD03).
   - Intelligence Center X customer results: 95% less manual effort and 85% faster issue resolution (not meshing).

## 5. Unverified or conflicting items
- **Intelligence Center X and MCP:** only a third-party summary says it integrates through MCP; Siemens' own wording was not found.
- **"NX renamed Designcenter with the 23 June 2026 release" and "Designcenter Copilot":** third-party claim only (inteliscience.net).
- **myshare-cms.siemens.com/siemens-nx-mcp-server:** a page on a Siemens subdomain whose title is "siemens nx mcp server". It would not load and its content is unknown; it looks like a search-landing page.
- **Dassault's internal use of MCP and A2A:** reported only by LeMagIT (date not captured); not confirmed by Dassault.
- **mcp.siemens.com:** launch date and GA/beta status not stated.
- **Not checked because the search budget ran out:** Hexagon AI/Nexus, any MSC Apex AI features, Keysight-ESI AI announcements, and Siemens + Microsoft Teamcenter integrations involving MCP.
- **Registry caveat:** a vendor missing from the MCP Registry does not prove it has no MCP server; vendors may host them privately.

## 6. Sources
- https://mcp.siemens.com/docs ; https://dev.mcp.siemens.com/docs ; https://developer.siemens.com/resources/how-tos/mcp.html ; https://developer.siemens.com/mcps/ix-mcp/overview.html ; https://developer.siemens.com/mcps/element-mcp/overview.html ; https://ix.siemens.io/docs/home/mcp-server ; https://registry.npmjs.org/@siemens%2fix-mcp ; https://github.com/orgs/siemens/repositories?q=mcp
- https://marketplace.mendix.com/link/component/240380/Mendix/MCP-Server
- https://news.siemens.com/en-us/siemens-fuse-eda-ai-agent/ ; https://www.electronicsweekly.com/news/business/siemens-digital-launches-agentic-toolkit-2026-03/
- https://news.siemens.com/en-gb/siemens-intelligence-center-x/ ; https://www.digitalengineering247.com/article/siemens-launches-industrial-scale-agentic-ai-orchestration-platform ; https://techhq.com/news/siemens-realize-live-2026-intelligence-center-x-digital-twins/
- https://news.siemens.com/en-us/siemens-simcenter-summer-2026/ ; https://roboticsandautomationnews.com/2026/07/29/siemens-unifies-simulation-portfolio-with-ai-powered-simcenter-release/103691/
- https://inteliscience.net/nx-cad-mcp/ ; https://innfactory.ai/en/services/companygpt/integrations/siemens-nx/ ; https://myshare-cms.siemens.com/siemens-nx-mcp-server
- https://community.sw.siemens.com/s/question/0D5Vb00001LiDTCKA3/simcenter-starccm-macros-with-ai-26-build-a-javadoc-search-tool-mcp-server ; https://github.com/pacoEzq/starccm-javadoc-mcp
- https://www.3ds.com/newsroom/press-releases/dassault-systemes-expands-3dexperience-ai-native-agentic-platform-new-virtual-companion-skills-co-engineer-humans ; https://www.3ds.com/assets/invest/2026-02/dassault-systemes-unveils-new-way-of-working-for-industry-with-ai-powered-virtual-companions-pr-final-11-feb-2026_0.pdf ; https://blog.3ds.com/brands/simulia/engineering-workflows-evolve-simulia-virtual-companions-generative-experiences/ ; https://www.3ds.com/products/3dexperience/agentic-process-orchestration ; https://develop3d.com/cad/new-solidworks-ai-agents-added-at-3dexperience-world/ ; https://www.3ds.com/assets/invest/2025-11/dassault-systemes-mistral-ai-deepen-partnership-pr-final-26-nov-2025.pdf ; https://www.lemagit.fr/actualites/366638773/3DExperience-Dassault-Systemes-saute-le-pas-de-lIA-agentique ; https://www.getleo.ai/blog/solidworks-mcp-ai-agent-access
- MCP Registry queries (siemens, solidworks, abaqus, dassault, 3ds, catia, simulia, hexagon, nastran, keysight, teamcenter, simcenter, femap): https://registry.modelcontextprotocol.io/v0/servers?search=…
- https://github.com/Keysight/cyperf-mcp ; https://github.com/hexsupport/hex-mcp
- GitHub repos: Cai-aa/abaqus-mcp, Whfkl/Abaqus-Control-MCP, jianzhichun/abaqus-mcp-server, LonlySteve/abaqus6.14mcp, Tomsabay/abaqus_agent, TutuBarry/abaqus-mcp-pro, rutwikg/abaqus-mcp, MOBAI547800/abaqus-mcp-server, MOBAI547800/abaqus-codex-mcp, jinkeguo/cax-workflow-agent, TmxjTmxj/microneedle-skin-pullout, Patrick-Miles-Z/CAD-CC-Abaqus, DreamEnding/NX_MCP, cyw2527/nx-auto-mcp, mingfeng6684/nxopen-mcp, Cai-aa/CAD-Agent-Hub, goenninger-b-t/tc-mcp, SometingGBBB/Simcenter-Amesim-MCP, Shaoqigit/pynastran-mcp, hjbaard/SolidWorks-MCP, andrewbartels1/SolidworksMCP-python, daiemon12/catia-v5-mcp-server, WangJ226/_Marc_-_MCP_-Assistant (all at https://github.com/<repo>)
