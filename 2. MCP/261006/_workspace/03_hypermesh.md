# 03. HyperMesh / HyperWorks (Simcenter HyperMesh) and MCP — research findings as of 2026-10-06

> 서브 에이전트 원본 산출물 (영문, 감사 추적용 보존). 최종 보고서는 상위 폴더의 보고서 파일 참조.

**Limits on this research. Read these first.**
- **Blocked sites.** The network proxy blocked direct fetches of help.altair.com, altair.com, community.altair.com, siemens.com, news.siemens.com, blogs.sw.siemens.com, PR Newswire, DE247 and develop3d. Facts from those official pages come from search-engine extracts tied to their URLs, not from reading the full pages.
- **What I read directly.** GitHub pages, including the official Altair GitHub org.
- **Search budget ran out.** The shared WebSearch budget was used up partway through. Part (d) (papers, Altair Technology Conference talks, videos) and the sweep of Zhihu, CSDN, Qiita and Naver/Tistory were **not done**. A follow-up run is needed for those.

---

## 1. Verdict: is there an official MCP for HyperMesh?

**The user's claim holds. As of 2026-10-06, neither Altair nor Siemens offers an official MCP server or MCP client for HyperMesh or HyperWorks.** Since 2026 the product is sold as "Simcenter HyperMesh".

Evidence:
- **Release notes.** The HyperMesh "API and Customization" release notes for 2024, 2025, 2025.1 and 2026 list only Python and Tcl API changes. None mentions MCP, agents or tool servers.
- **Launch announcements.** None of these mention MCP for HyperMesh:
  - the HyperWorks 2026 launch press release (Dec 8, 2025, Troy, MI)
  - the "What's new in Simcenter HyperMesh 2026.1" blog (June 2026)
  - the unified Simcenter release press release (Jul 28, 2026)
  - Realize LIVE Americas 2026 (Jun 1–4, 2026, Detroit)
- **Altair's official GitHub org.** github.com/altairengineering has 26 repos (checked 2026-10-06) and no MCP repo. The only HyperWorks-related repo is `HyperWorksPyAPI`, a VS Code extension for autocomplete.
- **The only official MCP in the former Altair portfolio is Altair Graph Studio**, a knowledge-graph product in the RapidMiner line (announced Oct 28, 2025). It is not a pre-processing tool.
- **HyperMesh's official AI is Altair CoPilot.** It answers questions from the documentation and can generate Python scripts (beta). It neither exposes nor uses MCP.

"No official MCP" does not mean "no MCP". At least five community HyperMesh MCP servers appeared on GitHub between May and Aug 2026 (section 3).

## 2. Altair and Siemens AI features relevant to HyperMesh

| Item | Label | Status and date | Notes |
|---|---|---|---|
| **Altair CoPilot** (in HyperMesh, HyperMesh CFD, HyperMesh NVH, HyperView, HyperGraph, MotionView, HyperLife) | [AI-ASSISTANT-NO-MCP] | Beta in HyperWorks **2024.1**. Current HyperMesh docs title it "Altair® CoPilot" without "Beta". The HyperMesh CFD page still says "(Beta)". **Scripting is still beta.** | Needs an internet connection. Answers only from Altair help, Community knowledge-base articles, how-to videos and eLearning. Answers are narrowed to the active solver profile. The 2024.1 docs list OptiStruct, Radioss, MotionSolve and AcuSolve; **Abaqus is not listed**. Adding `#script` or `/script` to a question returns **Python** code (doc example: "#script create a dialog with OK and Cancel buttons"). The code is shown for the user to run; there is no evidence that CoPilot executes it. |
| **Copilot in Simcenter HyperMesh 2026 / 2026.1** | [AI-ASSISTANT-NO-MCP] | Siemens "What's new in Simcenter" (June 2026) | Same help assistant, now drawing on Simcenter help. It "can also assist users in developing workflows and understanding prerequisite steps". |
| **Simcenter PhysicsAI** (formerly physicsAI; geometric deep learning inside HyperMesh) | [OFFICIAL] GA, not LLM-based, no MCP | HyperWorks 2026 press release (Dec 8, 2025); 2026.1 release (June 2026) | Claims results "up to 1,000x faster" than solvers and models that can be deployed in a browser. 2026.1 adds training on a region of interest only, training on element-level field data, and a 0–1 similarity score. The Jul 28, 2026 press release says PhysicsAI data and models can feed Intelligence Center X. |
| **Retrieve (ShapeAI)** | [OFFICIAL] GA, no MCP | Simcenter HyperMesh 2026.1 | Finds similar parts by shape and loads the best match into the session. The same release adds up to 4x faster deck export and Teamcenter check-in/out from inside HyperMesh. |
| **Python Recording** | [OFFICIAL] GA (not AI) | HyperMesh 2025 | Records GUI actions as Python code. Useful as reference code to ground an LLM. |
| **Rebrand to Simcenter** | [OFFICIAL] | Siemens rebranding blog (dated Jun 16, 2026 in search extracts) | HyperMesh, HyperView, HyperGraph, Inspire and SimLab now carry the "Simcenter" name. A reseller (Simutron) gives Jan 1, 2026 as the effective date. |
| **Altair RapidMiner: Graph Studio MCP integration and Agent Studio** | [OFFICIAL] announced in the Oct 28, 2025 release | altair.com newsroom and PR Newswire | Quote: "With MCP integration, agents directly interact with Graph Studio to query, reason, and make decisions." That implies Graph Studio acts as an MCP server. Agent Studio's MCP role is not stated. Not related to HyperMesh. |
| **Siemens Intelligence Center X** (Mendix + Graph Studio + AI Studio) | [OFFICIAL] announced Jun 1–3, 2026 at Realize LIVE Americas 2026 | Agent-orchestration software | "MCP-based connectivity" appears in material from **CLEVR, a Mendix partner** ([PARTNER] claim). I did not confirm it in Siemens' own release text. It has no link to HyperMesh. |
| **Unified Simcenter release** | [OFFICIAL] | Jul 28, 2026 (Plano, TX) | First release combining Siemens and Altair tools. Mentions AI, GPU and PhysicsAI; no MCP. |
| **Realize LIVE Americas 2025** | Third-party report (DE247) | 2025 | Siemens described the Altair deal in terms of nonlinear solvers, data/AI and HPC. No MCP roadmap. |

**Not found or not verified:** a "HyperWorks AI" product; any AI assistant or MCP in Altair One; whether CoPilot is in Inspire or SimLab. The HyperWorksPyAPI stubs do cover SimLab, Inspire (`hwx`) and `report` modules, so those products have Python APIs.

## 3. Community MCP servers and LLM bridges for HyperMesh (all [COMMUNITY])

| Repo | Activity (2026) | Stars / license | Approach | Capabilities | Maturity |
|---|---|---|---|---|---|
| [times1234/hypermesh-mcp-server](https://github.com/times1234/hypermesh-mcp-server) | May 4–5 ("Initial HyperMesh MCP server") | 4★, 17 commits, no license stated | Python MCP server that generates Tcl. Runs it through `hmbatch.exe` or a socket listener inside the open GUI (`execute_tcl_gui`). | Geometry probe; gear-aware tetra mesh; guarded drag/spin hex; cut-section spin; R-trias surface deviation | Early. The earliest HyperMesh MCP I found. |
| [times1234/hypermesh-mcp](https://github.com/times1234/hypermesh-mcp) | Commits visible May 11 → **Aug 10** (v5.0 "新增联动ansa", i.e. ANSA linkage added) | 10★, 2 forks, about 56 commits, no license | Batch mode plus GUI listener (default 127.0.0.1:47881). Env vars `HYPERMESH_BATCH_EXE` and `HYPERMESH_GUI_EXE`. Config example uses **Altair 2020**. | `locate_hypermesh`, `execute_tcl(_gui)`, `classify_all_solids_from_probe`, `generate_guarded_drag_hex_tcl`, `generate_cutsection_spin_hex_tcl`, `generate_plain_tetra_tcl`, finalize (rename/colour). **Raw meshing Tcl is rejected** unless one of the server's generators produced it (`MCP_SCRIPT_BEGIN/END` markers). | Most developed. Has offline panels. Listed on PulseMCP. Admits limits, e.g. some revolved solids fall back to tetra. |
| [yesooner/hyper-dyna-mcp](https://github.com/yesooner/hyper-dyna-mcp) | Last commit **Jun 12** | **18★**, 3 forks, 110 commits, v2.0.0, **AGPL-3.0** | FastMCP over stdio. Load `hmcustom.tcl` in HyperMesh; it adds an MCP tab and a listener on **port 47883** (socket or IPC) that replies `HYPERMESH_MCP_PONG`. GUI only. | `hm_modeling_action` (create_mesh, create_element, assign_material/property, apply_constraint/load); hex/tet/tria/beam/mass creators; LS-DYNA keyword policy. **Blocked:** `*tetmesh`, surface automesh, K-file export. | README rule: "Do not guess unverified HyperMesh Tcl commands." Only smoke tests. HyperMesh version not stated. |
| [jinkeguo/cax-workflow-agent](https://github.com/jinkeguo/cax-workflow-agent) | Jul 26 (3 commits) | 1★, MIT | Codex plugin with **32 MCP tools**. SolidWorks via COM; HyperMesh via a Tcl/batch adapter; then Abaqus. | Solid-map meshing by explicit ID, **C3D8R**, 3D Jacobian/aspect/min-length checks, Abaqus deck export and validation, datacheck, job submission, ODB post-processing. Diagnoses failures and recovers. **Geometry, mesh, material, contact, load and boundary-condition changes need user approval.** | Tested on **HyperMesh 2025 and Abaqus 2022**. Self-described "early public-release candidate". **Closest match to the user's HyperMesh + Abaqus workflow.** |
| [wangguan1995/hypermesh-mcp](https://github.com/wangguan1995/hypermesh-mcp) | Aug 15 (9 commits, one day) | 1★, no license | `hmbatch.exe` at `C:\Program Files\Altair\2026\hwdesktop\hm\bin\win64\`. Plugin for DeepSeek Harness. | `convert_stp_to_hm` and `generate_mesh` (classify, mesh, save) | Prototype |

**False positives:** `damandeep-hyprbots/hypermesh-mcp` is a document-OCR server unrelated to Altair. `hyper-mcp` / `kranners/hyper-mcp` is a Rust plugin host, also unrelated.

**Bridges that predate MCP (no LLM):**
- [Ch4oooooooLL/HMWorkFlow](https://github.com/Ch4oooooooLL/HMWorkFlow): Tcl plus Python 3.8, parallel `hmbatch` workers, JSON/file/socket hand-off. Baseline HyperMesh 2019, with 2022 support in progress. OptiStruct. Updated Sep 30, 2026.
- [HMFrameLoadKit](https://github.com/Ch4oooooooLL/HMFrameLoadKit): HyperMesh 2019 + OptiStruct. Updated Jul 6, 2026.
- [zbingf/TclPyHyperWorks](https://github.com/zbingf/TclPyHyperWorks): 64★, HyperWorks 13/2017/2021.1, includes a Python runner for `hmbatch`.

**Community MCP for other ex-Altair or Siemens products:**
- `alan4041207/mcp-altair-studio`: Altair AI Studio 2026.1.1 with Claude. Updated Jul 10, 2026.
- `rperdiga/Altair_Graph_MCP_HTTP`: Anzo Graph Server, 43 tools. Updated Jan 12, 2026.
- `SometingGBBB/Simcenter-Amesim-MCP`: README says "Unofficial… Not affiliated with or endorsed by Siemens."

Most of these repos have Chinese-language READMEs and commit messages.

## 4. Feasibility of a custom HyperMesh MCP server

**APIs and versions**
- **Tcl API.** Present for most of HyperMesh's history. Two kinds of command:
  - `*` modify commands, e.g. `*createmark`, `*tetmesh`, `*feoutputwithdata`
  - `hm_*` query commands, e.g. `hm_getvalue`

  Docs: [Tcl reference](https://help.altair.com/hwdesktop/hwd/topics/chapter_heads/tcl_r.htm) and [modify commands](https://2023.help.altair.com/2023/hwdesktop/hwd/topics/chapter_heads/tcl_modify_commands.htm). The Tcl route works on old installs; one community MCP was configured against HyperMesh 2020.
- **Python API.** The belief that it came in 2022.x/2023 is **refuted** for HyperMesh. The release notes call it the "first release of HyperMesh Python API" in **HyperMesh 2024(.0)**. HyperView and HyperGraph got a first-phase Python API in 2023.1. Main objects: `import hm`, `hm.entities as ent`, `hm.Model()`, `hm.Collection(model, ent.Node)`, `CollectionByInteractiveSelection`, `FilterByAttribute`, and the `hwx.gui` toolkit.
  - **2024 limitations:** not every function covered; failures return a status object instead of raising an error.
  - **2025:** Python Recording.
  - **2025.1:** `Collection.get_values()` returns lists or NumPy arrays; `set_items()` renamed `set_values()`; Compose debugger.
  - **2026:** all Model modify/query methods documented; `Model.get(class, query)`; `Assembly` entity class; `hm_info*()`; `hwx.gui` adds WebView, Canvas and Toolbar.
  - **`hm` can only be imported inside HyperMesh's own Python**, not from an external interpreter (Altair community answer). HyperWorksPyAPI gives autocomplete only.
  - Docs: [hm module](https://help.altair.com/hwdesktop/pythonapi/hypermesh/hm.html), [user guide](https://help.altair.com/hwdesktop/pythonapi/hypermesh/hm_user_guide.html), [collections](https://help.altair.com/hwdesktop/pythonapi/hypermesh/hm_user_guide/entity_sel_via_col.html), [examples](https://help.altair.com/hwdesktop/pythonapi/hypermesh/examples/hm_examples.html).
- **Batch mode.**
  - Classic: `hmbatch -tcl <script>`, `-c<cmdfile>`, `-continue`, `-nobg`, `-templex` ([docs](https://help.altair.com/hwdesktop/hwx/topics/getting_started/hm_batch_startup_options_r.htm)). `hmbatch` still ships in 2026.
  - Newer client with Python (community answer): `runhwx.exe -client HyperworksDesktop -plugin HyperworksGeneral -profile HyperworksGeneral -b -f script.py`.
  - Older: `hw.exe -b -tcl`.
- **Controlling a running session from outside.** There is **no official REST, COM or command server**. The options are:
  1. A Tcl `socket -server` listener loaded into the GUI session. All the community servers do this; the "hmbatch as server" community thread shows the same idea.
  2. The legacy Process Manager socket protocol (HWMCommMgr, 2017 docs): commands sent as `"C *readfile …"`, replies `1 …` or `0 …`. Probably not supported in today's client; unverified.
  3. The C/C++ External API (`hm_extapi.h`, `Open_HM_ExtAPI()`), which loads HyperMesh as a library with a limited function set.

**Suggested architecture (agent's assessment, not from a source):**

```
MCP host (Claude Code/Desktop, Copilot, Codex)
  └─ stdio ─ MCP server (Python, official SDK/FastMCP): typed tools, guardrails, audit log, approval gates
       ├─ localhost TCP (127.0.0.1 + token, JSON) ─ in-session adapter
       │     Tcl socket -server + fileevent → whitelisted procs (any version)
       │     Python 2024+ (hm.Model / Collection.get_values) for structured queries
       ├─ headless: hmbatch -tcl … / runhwx -b -f … for deterministic batch jobs
       └─ Abaqus: export .inp via Abaqus profile → abaqus datacheck/job → parse .dat/.msg/ODB
```

**Tools to expose:**
- `get_model_summary`, `list_components`, `quality_report` (Jacobian, warpage, minimum length)
- `import_cad`, `batchmesh`, `tetmesh(params)`
- `assign_property`, `create_sets`, `export_abaqus_inp`, `run_datacheck`
- `execute_tcl`, guarded and requiring approval

**Lessons from the community servers:**
- LLMs invent HyperMesh Tcl commands. Use verified routes and whitelists, and ground the model with Python Recording output and the command journal.
- Probe the geometry before choosing a meshing strategy.
- Require approval for any change to the physics.
- **Licensing:** yesooner is AGPL-3.0 and times1234 has no license, which matters for reuse at an OEM.

## 5. Use cases and demos found

- **[OFFICIAL] CoPilot `#script`.** Generates HyperMesh Python code, for example GUI dialogs (beta, 2024.1 and later).
- **[COMMUNITY] Drivetrain parts (gears, bearings, gearbox) meshed by Claude Code or Codex** (times1234): classify by geometry → hex mesh where the solid can be swept or revolved → otherwise tetra with refinement on gear teeth. ANSA linkage added in Aug 2026.
- **[COMMUNITY] Building an LS-DYNA model in the HyperMesh GUI through verified routes** (yesooner).
- **[COMMUNITY] Closed loop SolidWorks → HyperMesh 2025 (C3D8R) → Abaqus 2022** with datacheck, failure diagnosis and recovery (jinkeguo).
- **[COMMUNITY] STEP to `.hm` conversion plus automatic meshing in batch** with DeepSeek Harness (wangguan1995).
- **Korean-language Altair community threads** on running Python in HyperMesh batch mode (#60901) and on Python Recording in 2025.1 (#63630). Useful to the user, but not about LLMs.
- **Not found (search not completed):** peer-reviewed papers, Altair Technology Conference talks, videos, or Zhihu/CSDN/Qiita/Naver posts about LLMs generating HyperMesh scripts.

## 6. Unverified or conflicting items

- Rebrand effective date: Jan 1, 2026 (reseller) versus the Siemens guide dated Jun 16, 2026. Both come from search extracts.
- "Machine-learning part grouping" in HyperMesh 2026 appears only in a third-party summary; could not be confirmed in the release notes.
- The Intelligence Center X date varies by source (Jun 1, 2, or 3, 2026). Its MCP claim comes only from the partner CLEVR.
- A Siemens community post, "Simcenter STAR-CCM+ Macros with AI (2/6): Build a Javadoc Search Tool (MCP Server)" (~June 2026), is a tutorial, not a product. Its author's affiliation is unverified.
- Whether CoPilot covers the Abaqus profile in 2025/2026 is unknown; the 2024.1 list excludes it.
- The CoPilot beta-to-GA transition date is unknown.
- HyperWorks 2025 and 2025.1 release dates were not verified.
- Community repo facts (tool counts, test claims) are self-reported. times1234's ANSA linkage appears only in a commit message. The last-update date for TclPyHyperWorks conflicts: Jul 8, 2024 on the topic page versus Dec 4, 2023 on the repo page.
- Process Manager socket support in today's HyperMesh client is unverified.

## 7. Sources

**Official Altair and Siemens:**
- https://help.altair.com/hwdesktop/hwx/topics/user_interface/altair_copilot_r.htm
- https://2024.help.altair.com/2024.1/hwdesktop/nvh/topics/user_interface/altair_copilot_r.htm
- https://help.altair.com/hwdesktop/cfd/topics/user_interface/altair_copilot_r.htm
- https://help.altair.com/hwdesktop/hwx/topics/release_notes/rn_2026_hypermesh_r.htm
- https://help.altair.com/hwdesktop/hwx/topics/release_notes/rn_2026_hypermesh_api_r.htm
- https://help.altair.com/hwdesktop/altair_help/topics/release_notes/rn_2024_hypermesh_api_r.htm
- https://2025.help.altair.com/2025/hwdesktop/hwx/topics/release_notes/rn_2025_hypermesh_api_r.htm
- https://2025.help.altair.com/2025.1/hwdesktop/hwx/topics/release_notes/rn_2025_1_hypermesh_api_r.htm
- https://2025.help.altair.com/2025/hwdesktop/pythonapi/hypermesh/hm_recording.html
- https://help.altair.com/hwdesktop/pythonapi/hypermesh.html
- https://help.altair.com/hwdesktop/pythonapi/hypermesh/hm_user_guide/hypermesh_funcs.html
- https://help.altair.com/hwdesktop/hwd/topics/reference/hm/extapi.htm
- https://help.altair.com/hwdesktop/hwd/topics/chapter_heads/extapi-functions.htm
- https://2017.help.altair.com/2017/hwtools/communication_in_process_manager.htm
- https://2017.help.altair.com/2017/hwtools/hwmcommmgr.htm
- https://news.siemens.com/en-us/altair-hyperworks-2026/
- https://blogs.sw.siemens.com/simcenter/whats-new-in-simcenter-hypermesh-2026-1/
- https://blogs.sw.siemens.com/simcenter/whats-new-in-simcenter-physicsai-2026-1/
- https://blogs.sw.siemens.com/simcenter/beyond-a-name-change-your-altair-rebranding-guide/
- https://www.siemens.com/en-us/products/simcenter/latest-version/
- https://www.siemens.com/en-us/products/simcenter/simulation-modeling-visualization/hypermesh/
- https://www.prnewswire.com/news-releases/siemens-accelerates-engineering-simulation-with-a-unified-ai-powered-simcenter-portfolio-302835381.html
- https://www.prnewswire.com/news-releases/siemens-powers-the-next-phase-of-industrial-ai-with-intelligence-center-x-302787786.html
- https://news.siemens.com/en-us/realize-live-americas-2026-recap-day-1/
- https://altair.com/newsroom/news-releases/altair-rapidminer-data-analytics-and-ai-platform-accelerates-enterprise-intelligence-with-expanded-agentic-ai-and-analytics-ecosystem
- https://github.com/orgs/altairengineering/repositories
- https://github.com/altairengineering/HyperWorksPyAPI

**Community and third-party:**
- https://community.altair.com/discussion/20493/accessing-hm-and-hm-entities-externally
- https://community.altair.com/discussion/11335/hmbatch-as-server
- https://community.altair.com/discussion/62568/does-hypermesh-have-a-batch-mode-for-running-python-scripts
- https://community.altair.com/discussion/62393/how-to-run-hypermesh-in-batch-mode
- https://community.altair.com/discussion/64417/how-to-get-altair-copilot-to-work
- https://community.altair.com/discussion/63630/hypermesh-python-recording-기능-사용-방법-2025-1-ver-기준
- https://www.clevr.com/blog/siemens-intelligence-center-x-make-processes-intelligent
- https://simutron.co.za/altair-simcenter-name-change-your-complete-guide-to-the-new-product-names/
- https://www.designfusion.com/post/altair-hypermesh-2026-whats-new
- https://www.digitalengineering247.com/article/siemens-realize-live-americas-kicks-off-in-detroit
- https://www.pulsemcp.com/servers/times1234-hypermesh
- https://community.sw.siemens.com/s/question/0D5Vb00001LiDTCKA3/simcenter-starccm-macros-with-ai-26-build-a-javadoc-search-tool-mcp-server
- https://github.com/topics/hypermesh
- GitHub repos linked in section 3, plus https://github.com/alan4041207/mcp-altair-studio, https://github.com/rperdiga/Altair_Graph_MCP_HTTP and https://github.com/SometingGBBB/Simcenter-Amesim-MCP
