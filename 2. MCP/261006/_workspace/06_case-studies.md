# 06. LLM/MCP agents for FE pre-processing, meshing and simulation: use cases, case studies and literature (status 2026-10-06)

> 서브 에이전트 원본 산출물 (영문, 감사 추적용 보존). 최종 보고서는 상위 폴더의 보고서 파일 참조.

**How this was checked, and its limits.**
- Could not open arXiv, SSRN, Springer, NAFEMS or vendor pages directly because the egress proxy blocked them. Facts about papers and vendors come from search-engine extracts of those pages.
- GitHub READMEs and commit pages were read directly.
- The shared WebSearch budget (200 calls per turn) ran out partway through. The Siemens, Dassault, COMSOL and Autodesk event searches, the Anthropic/OpenAI/Microsoft customer-story searches, the LinkedIn/YouTube demo searches and all Korean-language searches were never run. §5 lists these gaps.
- "n/v" means not verified.

## 1. Academic literature

### 1a. FE/structural and meshing (closest to the HyperMesh + Abaqus stack)

| Title | Authors / Affil. | Date | Venue / ID | Tool(s) | MCP? | What was automated | Key results | Link |
|---|---|---|---|---|---|---|---|---|
| A Multi-AI-agent Framework Enabling End-to-end FEA for Solid Mechanics Problems ("AbaqusAgent") | T.R. Sarker, M.J. Zulqernine, L. Yue, S. Pan, C. Wang, S. Lin | Jun 2026 | arXiv 2606.00138 | Abaqus (.inp/.odb) | No (uses Anthropic + OpenAI APIs) | Natural language → .inp file (geometry, material, BCs, loads), then run, an error-review loop and .odb visualization. Six agents: interpreter, architect, input writer, runner, reviewer, visualizer | 50 solid-mechanics problems, **86% success** | github.com/LIRAM-LIN/AbaqusAgent |
| VFEAgent: Multimodal Agent Framework for End-to-End Automated FEA | Peking Univ.; China Agricultural Univ. | May 2026 | arXiv 2605.28978 | Abaqus (Python 3.9 agents, Python 2.7 kernel) | Not reported | Image + text → FEA spec via a ReAct vision-language pipeline, then verification-first code synthesis with self-debugging and fallback | **Schema validity 90.0%**, best against CoT baselines (GPT-4o, GPT-5 preview, Gemini-3-Pro, Qwen-3-Max) | arxiv.org/abs/2605.28978 |
| CAX-Agent: Lightweight Agent Harness for Reliable APDL Automation | C. Lin, Y. Hai, Y. He, R. Wang, H. Qiang, L. Yu | 12 May 2026 | arXiv 2605.15218 | Ansys MAPDL | Not reported | LLM writes APDL. Failures go up a recovery ladder: rule patch → LLM regeneration → more context → human | 50 benchmarks × 3 runs per strategy. Completion **0.9267** (model_only) vs 0.7733 (rule_only) vs 0.6933 (no recovery). Zero-intervention rate 0.84. Rater agreement κ = 0.84 | arxiv.org/abs/2605.15218 |
| FeaGPT: End-to-End agentic-AI for FEA | Y. Qi, R. Xu (Univ. Stuttgart); X. Chu (Exeter/Stuttgart) | 24 Oct 2025 | arXiv 2510.21993 | Gmsh Python API, CalculiX | No | Geometry → mesh → simulation → analysis. The LLM passes mesh density class, refinement zones and tet/hex choice. Faces described as "fixed" or "loaded" become physical groups. BCs are inferred | Turbocharger cases (7-blade compressor, 12-blade turbine at 110,000 rpm). 432 NACA configurations. No overall success rate reported | arxiv.org/abs/2510.21993 |
| **MeshExpert**: Physics-Aware LLM Agent Framework for Automated Industrial FE Meshing | R. Zhao (NUAA), Y. Han, G. Tian, T. Song | 8 Jul 2026 | SSRN 7083227 | Pre-processor API (which one: n/v) | Not reported | Meshing operation sequences. A 20B model fine-tuned on API semantics and expert workflows, then trained with GRPO using a simulation-feedback reward (executability, mesh quality, efficiency). RAG over meshing guidelines | 50 **industrial automotive components**: **82.5% Pass@3**, mean Jacobian **0.78** | papers.ssrn.com/sol3/papers.cfm?abstract_id=7083227 |
| PAMF: LLM-driven framework for automated mesh generation in mechanical simulation and CAE workflows | Pan, Yin, Dou | 2026 | J. Mech. Sci. Technol. 40:4497–4507 | Ansys APDL | No | Two fine-tuned agents: one generates the mesh up front, one predicts error, and the predicted error drives refinement | 100 models: **88.4%** mesh success; 66.8% of meshes under 10% ZZ error; **3.69× faster** | doi.org/10.1007/s12206-026-0334-6 |
| GReFEM: Multimodal LLMs as zero-shot assistants for physics-guided 3D mesh refinement | n/v | Jul 2026 | arXiv 2607.08798 | CAD scene → FE mesh refinement | No | Multimodal LLM picks the regions that need refinement; an "orthoViews" module selects camera views | More precise than a blind baseline (numbers n/v) | arxiv.org/abs/2607.08798 |
| MooseAgent | T. Zhang, Z. Liu, Y. Xin, Y. Jiao (Nuclear Power Institute of China) | Apr 2025 | arXiv 2504.08621 | MOOSE | No | Natural language → input files. Task decomposition, RAG over annotated input cards, iterative fixing. Default models: DeepSeek-V3/R1, GPT-4o-mini | **93%** average success (heat transfer, mechanics, phase field, multiphysics) | github.com/taozhan18/MooseAgent |
| MechAgents | Bo Ni, M.J. Buehler (MIT) | Nov 2023 | arXiv 2311.08166 | FEniCS | No | Agents write, run and self-correct FE code covering BCs, geometry, mesh and constitutive law | Qualitative (elasticity problems) | arxiv.org/abs/2311.08166 |
| ALL-FEM | Listed on Univ. da Coruña portal | Mar 2026 | arXiv 2603.21011 | FEniCS | No | LLMs of 3B–120B fine-tuned on 1,000+ verified scripts. Agents formulate the PDE, then code, debug and visualize | 39 benchmarks (scores n/v) | arxiv.org/abs/2603.21011 |
| Constrained NL interface for variational multiphysics FE in FEniCS | N. Upadhyay, W.F. Reinhart (Penn State) | 10 Jun 2026 | arXiv 2606.10928 | FEniCS, Gmsh | No | The LLM is limited to turning the prompt into JSON and writing Gmsh code only for geometries outside the catalog | n/v | arxiv.org/abs/2606.10928 |
| Automating Structural Analysis Across Multiple Software Platforms Using LLMs | U. Miami, HBC Eng., Hunan U., Lehigh | Apr 2026 | arXiv 2604.09866 | ETABS, SAP2000, OpenSees | No | Reasoning agents write one unified JSON; translator agents turn it into scripts for each code in parallel | 20 frame problems × 10 trials: **above 90% accuracy** on all | arxiv.org/abs/2604.09866 |
| TO-Master | H. Lin … T. Xue (HKUST; China Iron & Steel Research Inst.) | 2 Jul 2026 | arXiv 2607.01812 | FE topology optimization | Tool-orchestrated (MCP n/v) | Generates or accepts meshes (including image-to-mesh), checks mesh and BCs, passes typed solver arguments | Reproduces standard benchmarks (qualitative) | arxiv.org/abs/2607.01812 |
| FEABench | N. Mudur et al. (Google Research, Harvard) | NeurIPS 2024 workshop; Apr 2025 | arXiv 2504.06260 | COMSOL via API | No | Agent writes COMSOL API calls, checks the output and iterates | 15 "Gold" + 200 "Large" problems. Best strategy produces executable API calls **88%** of the time | arxiv.org/abs/2504.06260 |

### 1b. CFD agents and benchmarks (where MCP adoption is furthest along)

| Title | Authors / Affil. | Date | Venue / ID | Tool(s) | MCP? | What was automated | Key results | Link |
|---|---|---|---|---|---|---|---|---|
| Foam-Agent → **Foam-Agent 2.0** | L. Yue, N. Somasekharan, T. Zhang, Y. Cao, Z. Chen, S. Di, S. Pan (RPI) | May 2025 / Sep 2025; journal 2026 | arXiv 2505.04997, 2509.18178; CMAME 461:119271 | OpenFOAM, Gmsh, ParaView/PyVista, Slurm | **Yes.** v2.0 exposes tools `plan`, `input_writer`, `run`, `review`, `apply_fixes`, `run_case`, `visualization`. Setup: `claude mcp add foamagent -- foamagent-mcp`, plus a `/foam` Claude Code skill. Commits: MCP packaging 27 Mar 2026; progress-notification fix for client timeouts 30 Mar 2026 | Meshing (including external .msh), case setup, HPC scripts, run, repair, visualization | Paper: **88.2%** on 110 FoamBench tasks. README (project-reported): Claude Opus 4.6 with 25 loops 100% basic / 100% advanced; Sonnet 4.6 87.88% / 75%; gpt-5.4 45.45% / 75% | github.com/csml-rpi/Foam-Agent |
| CFDLLMBench | Somasekharan … Pan (RPI, NREL) | Sep 2025; Feb 2026 | arXiv 2509.20374; JDMLR 3(13) | OpenFOAM | — | Benchmark: 90 multiple-choice questions, 24 coding tasks, FoamBench (110 basic + 16 advanced) | Best model zero-shot: **14%** on coding, **34%** on FoamBench. Multi-agent setups raise all models | github.com/NREL-Theseus/cfdllmbench |
| CFD-copilot | n/v | 8 Dec 2025 | arXiv 2512.07917 | CFD solver n/v | **Yes** (MCP for post-processing functions) | Fine-tuned LLM → runnable setup; agents run, auto-correct and analyse | NACA0012, 30P-30N. Authors conclude domain adaptation and MCP together improve reliability | arxiv.org/abs/2512.07917 |
| ChatCFD | n/v | Jun 2025 | arXiv 2506.02019 | OpenFOAM; DeepSeek-R1/V3 | No | Multimodal inputs → case → run → repair | **82.1%** execution vs MetaOpenFOAM 6.2% and Foam-Agent 42.3% on ChatCFD's own benchmark. **68.12% physical fidelity**. About $0.208 per case | arxiv.org/abs/2506.02019 |
| MetaOpenFOAM; OpenFOAMGPT (and 2.0) | n/v | Jul 2024; Jan / Apr 2025 | arXiv 2407.21320, 2501.06327, 2504.19338 | OpenFOAM | No | MetaGPT-style pipeline + RAG; RAG | MetaOpenFOAM: 85% pass, $0.22 per case. OpenFOAMGPT with RAG solved all 6 cases | — |
| A Preliminary Assessment of Coding Agents for CFD Workflows | Peking Univ.; AI for Science Inst. | Feb 2026 | arXiv 2602.11689 | OpenFOAM, Gmsh API | Generic coding agent | Prompt config steering the agent to reuse tutorials and repair from logs | Prompt guidance raises completion. **GPT-5.2 is much better at mesh generation** than earlier models | arxiv.org/abs/2602.11689 |
| What Do CAE Simulation Agents Really Need Beyond a Generic Harness? | J. Shi, T. Zhang | Sep 2026 | arXiv 2609.03718 | FoamBench | Generic harness | Ablation study | One generic agent **96.4%** vs the specialized multi-agent system's 88.2%. Execution-feedback repair: 71.8% → 96.4%. Solver tutorials: 80.9% → 96.4%. Scripted reflection adds nothing | arxiv.org/abs/2609.03718 |

### 1c. Post-processing, CAD→FE and surveys

| Title | Authors / Affil. | Date | Venue / ID | Tool(s) | MCP? | What was automated | Key results | Link |
|---|---|---|---|---|---|---|---|---|
| ParaView-MCP | S. Liu, H. Miao, P.-T. Bremer (LLNL) | 11 May 2025 | arXiv 2505.07064; IEEE VIS 2025 short paper | ParaView Python API | **Yes** | Visualization from natural language, with the agent viewing the viewport | Qualitative. The repo warns that a deprecated pvserver sync causes instability | github.com/LLNL/paraview_mcp |
| TOOLCAD | n/v | Apr 2026 | arXiv 2604.07960 | FreeCAD | **Yes** (FreeCAD MCP server) | Text-to-CAD via tool calls, trained with RL | n/v | arxiv.org/abs/2604.07960 |
| Self-Improving CAD Generation Agents with FEA as Feedback | n/v | May 2026 | arXiv 2605.17448 | CadQuery → STEP → FEA | n/v | Agent revises geometry using FEA feedback | n/v | arxiv.org/abs/2605.17448 |
| Survey of AI Methods for Geometry Preparation and Mesh Generation | S. Owen (Sandia) et al. | Dec 2025 | arXiv 2512.23719; IMR 2026 | — | — | Survey, including scripting automation with RL and LLMs | Conclusion: AI **complements** meshing algorithms rather than replacing them | arxiv.org/abs/2512.23719 |

**Adjacent items:**
- **AutoEDA** (arXiv 2508.01012) uses MCP servers to drive Tcl-scripted chip-design (EDA) flows. It is a useful parallel for Tcl-based HyperMesh.
- A Univ. Bayreuth student thesis (2025) generates Z88/Abaqus input from natural language plus STEP files.
- **No LLM or MCP paper driving HyperMesh or ANSA turned up** in the searches that could be run.

## 2. Industrial case studies and demos

| Who | What | When | Tool(s) | MCP? | Outcome | Link |
|---|---|---|---|---|---|---|
| Altair (Siemens) + **Lucid Motors** | Webinar "AI Agents of Change". Agents fetch CAD, apply materials and set BCs. In Lucid's seat-frame workflow, agents used metadata tags to enforce design standards | 25 Sep 2025 | Altair CAE tools | Not stated | Vendor claim: setup time "from hours to minutes". No independent numbers | web.altair.com/agent-powered-simulation-automation |
| **BMW Group + Mistral AI** | Domain-specific "Large Industry Models" trained on over 1 PB of BMW crash-simulation data; thousands of virtual crash runs per week | ~late May 2026 | Crash CAE data | No | Described as a first step; targets faster analysis of crash results, not meshing. No metrics | bmwgroup.com/en/news/general/2026/AI-in-crash-simulations.html |
| BETA CAE (Cadence) | AI Assistant: RAG over the full documentation, Python script generation from plain-language prompts, and an agentic mode that hands off to built-in or user agents, including **on-prem or self-hosted LLMs**. No extra license. The 2026.1 release says "Agentic AI gaining ground in ANSA, EPILYSIS, META…" | Release 29 Jul 2026 | ANSA/META | n/v | Product capability; no customer metrics | beta-cae.com/news/20260729_announcement_suite_2026.1.htm |
| Ansys (Synopsys) Innovation Space | Course "Agentic AI for Ansys Mechanical and MAPDL workflows": Claude Code, Codex, Cursor and Copilot driving Mechanical/MAPDL in GUI and batch mode (model creation → solve → post-processing → report, plus in-house standards) | n/v | Mechanical, MAPDL | n/v | Training offering | innovationspace.ansys.com/product/agentic-ai-for-ansys-mechanical-and-mapdl-workflows-2/ |
| **SimuTech Group** (MingYao Ding, SVP) | Google Antigravity connected to a **live Ansys Mechanical session through the Mechanical MCP server**, using a bracket model: inspects the model, proposes approaches, builds modal and static workflows, explores random vibration and fatigue | n/v (2025–26) | Mechanical | **Yes** | Qualitative demo | simutechgroup.com/resources/tips-and-tricks/using-google-antigravity-with-ansys-mechanical-mcp/ |
| PADT | Blog "Using AI LLMs for Ansys APDL Scripts is Not There Yet". Microsoft Copilot Chat asked to write APDL that extracts and normalizes modal displacements | 16 Mar 2025 | MAPDL | No | "Yes, you can, sort of." APDL is "just different enough to trip the LLM up"; the scripts needed debugging | padtinc.com/2025/03/16/ai-llm-to-write-ansys-apdl-scripts/ |
| Rescale | Live demo of "Rescale Assistant" at NAFEMS World Congress 2025 | 19–22 May 2025 | HPC and data platform | n/v | Demo only | rescale.com/resources/nafems-congress-2025-rescale-sponsored-presentation/ |
| Honda Research Institute + Idiap | "MECHANIC" project: LLMs coordinating end-to-end simulation-based design | Dec 2024–Nov 2027 | — | — | Ongoing research | idiap.ch/en/projects/mechanic |
| rutwikg/**abaqus-mcp** (open source) | 13 tools. A JSON spec (STEP/IGES file or parametric shape) → CAE build and mesh in Python 2.7 → flat .inp → run → parse .sta/.msg/.dat → rule-based deck fixes → re-mesh on negative Jacobian or distortion → .odb results | Commits 30 Aug–17 Sep 2026 | Abaqus 2022 | **Yes** | Author reports it works end-to-end on real jobs. Shows that a converged run can still be physically wrong (§4) | github.com/rutwikg/abaqus-mcp |
| jianzhichun/abaqus-mcp-server | Drives an open Abaqus/CAE window through GUI automation (pywinauto): "Run Script" plus scraping the message log | 24–26 May 2025 | Abaqus/CAE | **Yes** | Prototype. It cannot return script output directly, so it is fragile | github.com/jianzhichun/abaqus-mcp-server |
| codersag/mechanical-mcp | Over PyMechanical gRPC: materials, element size and mesh, roughly ten BC types, solve, results, DOCX reports, arbitrary IronPython | Jun 2026 | Mechanical 2023 R1+ | **Yes** | Tool; 19 GitHub stars | github.com/codersag/mechanical-mcp |
| vorobjewsen30-max/ansys-mcp-server | 30 tools wrapping PyAnsys (Fluent, MAPDL, Mechanical, DPF, **Prime meshing**: generate, refine, quality, mesh transfer). Tool descriptions spell out the workflow order | 4–9 Jul 2026 | PyAnsys | **Yes** (listed in the MCP registry) | Tool | github.com/vorobjewsen30-max/ansys-mcp-server |
| LLK-LL/**K-agent** | Claude Code plugin / Codex skill that builds LS-DYNA .k decks (drop, crash, forming, ALE, SPH). Runs static checks (L0), initialization trials (L1) and full runs with energy, hourglass and mass-scaling gates (L2) | 29 Jul–23 Sep 2026 | LS-DYNA R14.1.1 | Only for literature search | Tool; no metrics | github.com/LLK-LL/K-agent |
| test1card/femis-skill | Agent Skill that acts as a governance layer: mesh-independence checks, V&V, and a contract on what may run headless vs. what needs a human | 28–29 Jun 2026 | Ansys, Abaqus, Nastran, LS-DYNA… | Designed to pair with CAE MCP servers | Tool | github.com/test1card/femis-skill |
| r/fea user | Overlay that screenshots the Ansys window to chat about it, plus UI and API automation | 12 Mar 2026 | Ansys | n/v | Prototype; commenters say Ansys is building its own assistant | reddit.com/r/fea/comments/1rrsn4y/ |

## 3. Korean sources

- **MCP-SIM (KAIST, Dept. of Mechanical Engineering).** Donggeun Park, Hyeonbin Moon, Seunghwa Ryu.
  - Title: "A self-correcting multi-agent LLM framework for language-based physics simulation and explanation".
  - Published as an engrXiv preprint (#4723, 2025) and on nature.com (s44387-025-00057-z; date n/v).
  - Turns vague prompts into FEniCS simulations through plan → act → reflect → revise cycles with shared memory, and writes multilingual explanation reports. **12 of 12** benchmark tasks succeeded.
  - **Naming trap:** "MCP" here means "Memory-Coordinated Physics-aware". It is **not** Model Context Protocol.
  - Prof. Ryu directs KAIST's InnoCORE PRISM-AI center.
- **PAMF** (see §1a) appeared in the *Journal of Mechanical Science and Technology*, the KSME/Springer journal. Authors' affiliations not verified; they do not appear to be Korean.
- **Korean-language searches were not run** because the budget ran out. Suggested follow-up queries:
  - MCP CAE 자동화; LLM 메시 자동화; 하이퍼메시 자동화 AI; Abaqus MCP; 해석 자동화 에이전트; CAE AI 에이전트
  - 현대자동차 CAE 생성형 AI; 마이다스아이티 AI 에이전트; 태성에스엔이 AI; KIMM LLM 해석
  - Proceedings of KSME and COSEIK (한국전산구조공학회), 2025–26

## 4. Cross-cutting lessons

**Typical architecture.**
- An LLM client (Claude Code or Desktop, Cursor, Codex, Antigravity) talks to an MCP server. Local setups use stdio; containers use HTTP (Foam-Agent exposes `--transport http` on port 7860).
- The server reaches the tool in one of four ways, from most to least robust:
  1. Official Python/gRPC APIs: PyMechanical, PyMAPDL, Prime, ParaView Python.
  2. Batch kernel scripts, such as Abaqus `cae -noGUI` run as a separate Python 2.7 subprocess (the pattern in abaqus-mcp and VFEAgent).
  3. Editing plain-text decks directly (.inp, .k, OpenFOAM dictionaries).
  4. GUI automation, the weakest option.
- Around that sit a log parser, a classifier, a repair loop, results extraction, and a knowledge layer (RAG over tutorials, manuals or annotated decks, and skills).
- For HyperMesh: HyperMesh journals every database-changing operation to `command.tcl`. That journal is an obvious source of examples for building a Tcl/Python MCP wrapper (agent's inference; no published HyperMesh work found).

**What works:**
1. **Repair driven by execution feedback is the biggest lever.** Examples: 71.8% → 96.4% (2609.03718), recovery 0.69 → 0.93 (CAX-Agent), and AbaqusAgent's reviewer loop reaching 86%.
2. **Giving the model domain examples** (tutorials, annotated input cards) adds up to about 15 points (80.9% → 96.4%). Scripted self-reflection adds nothing.
3. **A structured spec layer between the LLM and the tool** helps. Examples: a JSON spec with coordinate-free face selectors (abaqus-mcp), JSON translated to several solvers (above 90% accuracy), and the constrained FEniCS interface.
4. **Model strength matters most for geometry and meshing.** Examples: GPT-5.2's jump in mesh generation, and the Foam-Agent README's spread across models (Opus 4.6 at 100% vs others at 37.5–88%).
5. **Fine-tuning plus RL** is the only route so far with industrial-meshing numbers: MeshExpert at 82.5% Pass@3 on automotive parts, and PAMF at 3.69× faster.
6. **One general agent plus MCP tools can match bespoke multi-agent frameworks** (96.4% vs 88.2%).

**Failure modes:**
- Zero-shot code is weak: 14% and 34% on CFDLLMBench, and the APDL quirks PADT ran into.
- **A run that executes is not a correct run.**
  - ChatCFD: 82.1% execution but only 68.12% physical fidelity.
  - abaqus-mcp: a cantilever at 1.85× its Euler buckling load converged to 0.14 mm of deflection under stabilization instead of buckling. The server flags such runs "SUCCEEDED (with caveats)" and refuses to make up moduli or thicknesses.
  - femis-skill: headless thermal-contact settings that solve but give wrong answers.
- Long solver runs hit MCP client timeouts; Foam-Agent had to add progress notifications.
- Mixing Abaqus's Python 2.7 kernel with Python 3 tooling.
- GUI bridges break easily (the ParaView pvserver issue, scraping the Abaqus log).
- License and container constraints: Abaqus cannot ship inside a Docker image, and the license server must be reachable.
- Benchmarks are mostly simple or parametric geometry. Automotive mid-surfacing, welds and connectors, and batch-mesh quality criteria are barely tested.
- On-prem LLM support (BETA CAE) matters for OEM data security.

## 5. Unverified or conflicting items

- **Coverage gaps.** Search ran out before covering: Siemens Realize LIVE, Altair ATC, BETA CAE conference, COMSOL Conference, 3DEXPERIENCE World, Autodesk University, Anthropic/OpenAI/Microsoft customer stories, LinkedIn/YouTube demos, and Korean OEMs and vendors. **No OEM case with quantified MCP-based meshing results was found.**
- **Foam-Agent numbers depend on the benchmark:** 88.2% on FoamBench vs 42.3% on ChatCFD's benchmark. The README's model names are reported as written. The NeurIPS 2025 track for Foam-Agent is n/v.
- **MCP-SIM:** journal name and date n/v (the DOI prefix s44387 suggests npj Artificial Intelligence).
- **MechAgents:** the 2024 journal version (Extreme Mechanics Letters) is from background knowledge only.
- **Authors n/v:** CFD-copilot, ChatCFD, MetaOpenFOAM, OpenFOAMGPT, GReFEM. Affiliations n/v: PAMF.
- **MeshExpert:** which pre-processor API it drives is n/v.
- **FeaGPT on GitHub:** github.com/naividh/FeaGPT (Gemini + FreeCAD) describes itself as "based on" the paper. Whether the paper's authors wrote it is n/v.
- **SimuTech demo:** whether the "Mechanical MCP server" is an official Ansys product is n/v (cross-check with vendor agent). The SimuTech demo date and the Ansys course date are n/v.
- **Autodesk Fusion ↔ Claude via MCP:** seen only in a search extract.
- **Seen but content n/v:** RICOS "Generative CAE" (CES Innovation Awards 2025), a Toyota Motor Europe internship on crash-CAE automation, the CDH CAE User Forum 2026, NAFEMS NWC25 talks nwc25-0007152 and nwc25-0007070, and the GmshNet-8B model.
- **Title only:** "Your Simulation Runs but Solves the Wrong Physics" (2605.09360) and the case-bundle OpenFOAM paper (2609.11941).

## 6. Sources

**Papers and benchmarks**
- arxiv.org/abs/2606.00138 · github.com/LIRAM-LIN/AbaqusAgent
- arxiv.org/abs/2605.28978
- arxiv.org/abs/2605.15218
- arxiv.org/abs/2510.21993 · github.com/naividh/FeaGPT
- papers.ssrn.com/sol3/papers.cfm?abstract_id=7083227
- link.springer.com/article/10.1007/s12206-026-0334-6
- arxiv.org/abs/2607.08798
- arxiv.org/abs/2504.08621 · github.com/taozhan18/MooseAgent
- arxiv.org/abs/2311.08166
- arxiv.org/abs/2603.21011
- arxiv.org/abs/2606.10928
- engrxiv.org/preprint/view/4723 · nature.com/articles/s44387-025-00057-z
- arxiv.org/abs/2604.09866
- arxiv.org/abs/2607.01812
- arxiv.org/abs/2504.06260 · mlanthology.org/neuripsw/2024/mudur2024neuripsw-feabench
- arxiv.org/abs/2505.04997 · arxiv.org/abs/2509.18178 · github.com/csml-rpi/Foam-Agent (and its /commits/main page)
- arxiv.org/abs/2509.20374 · github.com/NREL-Theseus/cfdllmbench
- arxiv.org/abs/2512.07917
- arxiv.org/abs/2506.02019
- arxiv.org/abs/2407.21320 · arxiv.org/abs/2501.06327 · arxiv.org/abs/2504.19338
- arxiv.org/abs/2602.11689
- arxiv.org/abs/2609.03718
- arxiv.org/abs/2609.11941
- arxiv.org/abs/2505.07064 · github.com/LLNL/paraview_mcp
- arxiv.org/abs/2604.07960
- arxiv.org/abs/2605.17448
- arxiv.org/abs/2512.23719
- arxiv.org/abs/2508.01012
- arxiv.org/abs/2605.09360
- konstruktionslehre.uni-bayreuth.de/pool/dokumente/studentische_arbeiten/2025_FEM-Vorverarbeitung_durch_NLP-Agenten-Workflows_PG.pdf

**Industry**
- web.altair.com/agent-powered-simulation-automation · altair.com/resource/ai-agents-of-change-automate-accelerate-simulate-featuring-lucid-motors
- bmwgroup.com/en/news/general/2026/AI-in-crash-simulations.html · repairerdrivennews.com/2026/06/01/bmw-mistral-ai-partner-for-ai-use-in-crash-simulation/
- beta-cae.com/news/20260729_announcement_suite_2026.1.htm · www5.cadence.com/ai_assistant_ansa_meta_ebook.html
- innovationspace.ansys.com/product/agentic-ai-for-ansys-mechanical-and-mapdl-workflows-2/
- simutechgroup.com/resources/tips-and-tricks/using-google-antigravity-with-ansys-mechanical-mcp/
- padtinc.com/2025/03/16/ai-llm-to-write-ansys-apdl-scripts/
- rescale.com/resources/nafems-congress-2025-rescale-sponsored-presentation/
- idiap.ch/en/projects/mechanic
- reddit.com/r/fea/comments/1rrsn4y/ (seen via a mirror)
- 2017.help.altair.com/2017/hm_ref_guide/topics/reference/hm/command_files_r.htm

**Open-source MCP servers and skills**
- github.com/rutwikg/abaqus-mcp
- github.com/jianzhichun/abaqus-mcp-server
- github.com/codersag/mechanical-mcp
- github.com/vorobjewsen30-max/ansys-mcp-server
- github.com/knewnothing-git/ansys-mcp-server
- github.com/LLK-LL/K-agent
- github.com/test1card/femis-skill
