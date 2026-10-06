# Ansys 공식 MCP의 "알파" 의미와 사용 조건

**날짜**: 2026-10-06
**출처**: `3D_FE메시_프로그램_공식MCP_지원현황_및_사례_보고서.md` (261006) 관련 Q&A
**근거**: PyPI 메타데이터, GitHub `ansys/pymechanical-mcp`·`ansys/pymapdl-mcp`의 README와 설치·연결·클라이언트 설정 문서, `ansys/pymechanical` README. 모두 원문을 직접 확인했다.

---

## Q. Ansys 알파단계가 뭐지? 내가 지금 받아서 사용할 수 있는 거야?

---

## A.

### 1. "알파"의 의미: 공개는 됐지만 정식판이 아니라는 표시

PyPI(파이썬 패키지 저장소)에는 개발자가 스스로 매기는 **성숙도 등급(Development Status)**이 있다. 모두 7단계다.

```
1 Planning → 2 Pre-Alpha → [3 Alpha] → 4 Beta → 5 Production/Stable → 6 Mature → 7 Inactive
                              ↑ Ansys MCP 서버들의 현재 등급
```

알파라는 말에는 다음 뜻이 담겨 있다.
- **기능은 동작하고 누구나 받을 수 있다.** 다만 정식(Stable) 제품은 아니다.
- **버전이 0.x다. 도구 이름, 인자, 동작이 예고 없이 바뀔 수 있다.** 예를 들어 Mechanical-MCP는 2026-07-13(0.1.0)부터 2026-09-17(0.2.1)까지 두 달 동안 4번 릴리스됐다.
- **Ansys 설치 프로그램(2026 R1 등)에 들어 있는 기능이 아니다.** PyAnsys 오픈소스 패키지로 따로 배포되고, 제품 릴리스와 일정이 다르다.
- **README에 기술지원 범위가 명시돼 있지 않다.** 오픈소스 프로젝트라 버그 신고와 문의는 GitHub Issues가 기본 창구다. Ansys 정규 기술지원 대상인지는 회사 담당자에게 확인해야 한다.

### 2. 지금 받아서 쓸 수 있나 → 예. 단, Ansys 라이선스가 있어야 한다

- **누구나 무료로 설치할 수 있다.** Apache-2.0 오픈소스이고 PyPI와 GitHub에 공개돼 있다. 신청이나 계약은 필요 없다.
- 하지만 MCP 서버는 LLM과 Ansys를 잇는 **"다리"일 뿐**이다. 실제로 동작하려면 아래 조건이 모두 필요하다.

| 조건 | 내용 |
|---|---|
| **Ansys Mechanical 설치 + 라이선스** | **2024 R2(v242) 이상**이어야 한다(PyMechanical 요구사항). MCP 서버를 설치해도 라이선스는 생기지 않는다 |
| Python | 3.12 ~ 3.14 |
| OS | Windows, Linux |
| MCP 클라이언트 | Claude Code, Claude Desktop, VS Code(Copilot) 등. 공식 문서의 `uvx` 방식을 쓰려면 `uv`와 Git도 필요하다 |

> **첫 확인 사항: 회사에 Ansys Mechanical 2024 R2 이상 라이선스가 있는가.** 없다면 이 MCP만으로는 아무것도 할 수 없다. HyperMesh나 Abaqus 라이선스로는 대신할 수 없다.
> MAPDL용(`ansys-mapdl-mcp`)도 구조는 같다. MAPDL 설치와 라이선스가 필요하다.

### 3. 시작 방법 (공식 문서 기준)

**① 설치**
```bash
pip install ansys-mechanical-mcp            # PyPI 최신 버전
pip install ansys-mechanical-mcp==0.2.1     # 알파이므로 버전 고정 권장 (2026-10-06 기준 최신)
```

**② MCP 클라이언트에 등록**
- Claude Code, 공식 문서 방식. GitHub의 최신 main 브랜치를 그때그때 받아서 실행한다.
  ```bash
  claude mcp add --transport stdio pymechanical-mcp -- \
    uvx --index-strategy unsafe-best-match \
    --from git+https://github.com/ansys/pymechanical-mcp \
    ansys-mechanical-mcp
  ```
- Claude Code, ①에서 버전을 고정해 설치한 경우:
  ```bash
  claude mcp add --transport stdio pymechanical-mcp -- ansys-mechanical-mcp
  ```
- Claude Desktop: 설정 파일의 `mcpServers`에 같은 명령을 넣는다. 설정 파일 위치는 Windows가 `%APPDATA%\Claude\claude_desktop_config.json`이다. 공식 문서 예시는 macOS 경로로 적혀 있다.
- VS Code: `.vscode/mcp.json`에 같은 명령을 `servers` 항목으로 넣는다.

**③ Mechanical 연결** (세 가지 중 하나)
1. **가장 간단한 방법**: 대화창에서 "Mechanical 실행해 줘"라고 하면 `launch_mechanical` 도구가 호출된다. 기본은 GUI 세션이고, `batch=true`로 하면 백그라운드에서 실행된다.
2. **이미 띄운 세션에 연결**: gRPC 포트를 열어 Mechanical을 실행한 뒤 `connect_to_mechanical`로 127.0.0.1:50053에 연결한다.
   ```powershell
   & "C:\Program Files\ANSYS Inc\v261\aisol\bin\winx64\AnsysWBU.exe" -DSApplet -AppModeMech -grpc 50053
   ```
3. **서버 시작 시 자동 연결**: `ansys-mechanical-mcp --connect-on-startup --ip 127.0.0.1 --port 10000`. 이 방식을 쓰면 launch, connect, disconnect 도구는 비활성화된다.

**④ 테스트 프롬프트 예** (공식 예제를 바탕으로 함)
> "이 STEP 파일을 불러와서 2 mm 요소로 메시하고, 한쪽 면은 고정, 반대쪽 면에 10 kN 하중을 걸어 정적 해석한 다음 von Mises 응력 분포를 스크린샷으로 보여줘."

### 4. 주의사항

| 항목 | 내용 |
|---|---|
| **업무 적용 시점** | 알파이므로 PoC(시험) 용도로 쓴다. 버전을 고정하고, 업그레이드 전에는 변경 이력을 확인한다 |
| **데이터 보안** | 클라우드 LLM 클라이언트를 쓰면 모델 정보, 해석 결과, 스크린샷이 LLM 공급자에게 전송된다. **양산 차량 데이터는 사내 규정을 먼저 확인**하고, 승인된 환경이나 엔터프라이즈 계약 환경에서 쓴다 |
| **실행 권한** | `run_python_script`와 `run_python_code`는 Mechanical 안에서 임의 코드를 실행한다. 클라이언트의 도구 실행 승인 기능을 켜 두고, 실행 전에 코드를 확인한다 |
| **사내망** | `uvx --from git+https://github.com/...` 방식은 GitHub에서 직접 내려받으므로 사내 프록시에 막힐 수 있다. PyPI(또는 사내 미러)에서 `pip`으로 설치하는 편이 무난하다 |
| **전송 방식** | 로컬 PC에서는 기본값인 stdio 방식을 권장한다. HTTP 모드(`--transport http`)는 원격 접속용이다 |

### 5. HyperMesh + Abaqus 사용자 관점 요약

- Ansys MCP는 **Ansys로 메시를 만들고 해석하는 사람**을 위한 도구다. 지금의 HyperMesh → Abaqus 흐름을 대체하지 않는다.
- 회사에 Ansys 라이선스가 있다면 **"공식 MCP로 메시·해석을 자동화하면 어떤 느낌인가"를 체험하고 벤치마크하는 용도**로 좋다. 이 경험은 HyperMesh용 사내 MCP를 설계할 때 그대로 참고할 수 있다(본 보고서 §3.3).
