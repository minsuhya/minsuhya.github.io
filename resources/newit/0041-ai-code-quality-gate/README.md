# AI Code Quality Gate — Resource Pack

YouTube 강의 **["The AI Code Trap: Why Merging Without a Quality Gate Will Break Your Team"](https://www.youtube.com/@gillilab)** | **["AI 코드 그냥 머지했다가 프로덕션 터진 팀들의 공통점"](https://www.youtube.com/@gillilab)** 의 실무 자료.

강의에서 소개한 3레이어 품질 게이트를 그대로 팀 레포에 적용할 수 있는 production-ready 자료입니다.

채널: [GilliLab NEWIT](https://www.youtube.com/@gillilab)

> ⚠️ **면책 조항 / Disclaimer**  
> 본 자료는 교육 목적으로 제공되며, **사용으로 인해 발생하는 모든 결과에 대한 책임은 사용자 본인에게 있습니다.**  
> These Materials are provided for educational purposes. **All responsibility for outcomes arising from use lies solely with the user.**  
> → 전문 보기: [DISCLAIMER.md](DISCLAIMER.md)

---

## 포함 자료 (Contents)

### 영어 자료 / English

| 파일 | 용도 |
|------|------|
| [`.github/pull_request_template.md`](.github/pull_request_template.md) | 25-point AI code review checklist (PR template) |
| [`.github/prompts/llm-prereview-system.txt`](.github/prompts/llm-prereview-system.txt) | LLM pre-review system prompt |
| [`docs/en/team-onboarding-guide.md`](docs/en/team-onboarding-guide.md) | Team onboarding guide (1-page) |
| [`docs/en/do-not-merge-reference.md`](docs/en/do-not-merge-reference.md) | Do-Not-Merge vs Fast-Track-Safe reference |

### 한국어 자료 / Korean

| 파일 | 용도 |
|------|------|
| [`.github/pull_request_template.ko.md`](.github/pull_request_template.ko.md) | 25항목 AI 코드 리뷰 체크리스트 (PR 템플릿) |
| [`.github/prompts/llm-prereview-system.ko.txt`](.github/prompts/llm-prereview-system.ko.txt) | LLM 사전 리뷰 시스템 프롬프트 |
| [`docs/ko/team-onboarding-guide.md`](docs/ko/team-onboarding-guide.md) | 팀 온보딩 가이드 (1페이지) |
| [`docs/ko/do-not-merge-reference.md`](docs/ko/do-not-merge-reference.md) | 머지 금지 vs 패스트트랙 참조 카드 |

### 공통 / Shared (Language-agnostic)

| 파일 | 용도 |
|------|------|
| [`.github/workflows/ai-code-quality-gate.yml`](.github/workflows/ai-code-quality-gate.yml) | Layer 1: Gitleaks + Bandit + Semgrep + Radon + coverage |
| [`.github/workflows/llm-prereview.yml`](.github/workflows/llm-prereview.yml) | Layer 2: LLM 의미론적 사전 리뷰 (Claude API) |
| [`.pre-commit-config.yaml`](.pre-commit-config.yaml) | Pre-commit hooks (Gitleaks + Bandit + 기본 위생) |
| [`LICENSE`](LICENSE) | MIT License |

---

## 빠른 시작 (Quick Start — 5분)

### 1단계: 자료 다운로드

```bash
# 전체 레포 클론
git clone https://github.com/minsuhya/minsuhya.github.io.git
cd minsuhya.github.io/resources/newit/0041-ai-code-quality-gate
```

### 2단계: 프로젝트에 복사

```bash
# .github/ 폴더와 .pre-commit-config.yaml을 프로젝트 루트에 복사
cp -r .github /path/to/YOUR_PROJECT/
cp .pre-commit-config.yaml /path/to/YOUR_PROJECT/

# 한국어 PR 템플릿을 기본으로 사용하려면:
cd /path/to/YOUR_PROJECT/.github/
mv pull_request_template.md pull_request_template.en.md
mv pull_request_template.ko.md pull_request_template.md
```

### 3단계: GitHub Secrets 등록

GitHub 레포 **Settings > Secrets and variables > Actions** 에서:

| Secret 이름 | 값 | 필수 |
|------------|-----|------|
| `ANTHROPIC_API_KEY` | Anthropic API 키 ([console.anthropic.com](https://console.anthropic.com)) | ✅ (LLM 리뷰용) |
| `GITLEAKS_LICENSE` | Gitleaks 라이선스 키 ([gitleaks.io](https://gitleaks.io), 무료) | 조직 레포만 |

### 4단계: Pre-commit 설치 (선택, 강력 권장)

```bash
pip install pre-commit
pre-commit install
# 모든 파일에 대해 한 번 실행
pre-commit run --all-files
```

### 5단계: PR 열어 검증

```bash
git checkout -b test/quality-gate
echo "# test" > test.py
git add . && git commit -m "test quality gate"
git push origin test/quality-gate
# GitHub에서 PR 생성 → CI 자동 실행 → LLM 코멘트 게시 확인
```

---

## 환경 변수 설정 (Configuration)

워크플로우 동작은 **Settings > Variables > Actions** 에서 다음 변수로 커스터마이징할 수 있습니다 (코드 수정 불필요):

| 변수 | 기본값 | 설명 |
|------|--------|------|
| `LLM_PROMPT_LANG` | `en` | LLM 프롬프트 언어 — `en` 또는 `ko` |
| `MAX_DIFF_LINES` | `500` | LLM 리뷰 최대 diff 라인 (초과 시 스킵, 인간 리뷰 권장) |
| `DIFF_CHAR_LIMIT` | `30000` | LLM에 전송할 diff 최대 문자 수 |
| `ANTHROPIC_MODEL` | `claude-sonnet-4-6` | Claude 모델 — `claude-opus-4-7`로 변경 시 품질↑ 비용 1.67배 |
| `COMPLEXITY_LIMIT` | `20` | 순환 복잡도 차단 임계값 |
| `COVERAGE_THRESHOLD` | `70` | 테스트 커버리지 최소치 (%) |

---

## 3레이어 게이트 구조 (Architecture)

```
PR 오픈
  │
  ▼
[ Layer 0 — Optional ] .pre-commit-config.yaml (로컬 사전 차단)
  Gitleaks + Bandit + 위생 검사 (commit 시점에 실행)
  │
  ▼
[ Layer 1 ] ai-code-quality-gate.yml (자동 정적 분석)
  ├─ Gitleaks 시크릿 스캔        (gitleaks/gitleaks-action — 커밋 SHA 고정)
  ├─ Bandit SAST (Python 보안)   (HIGH/HIGH = 차단, tests/ 포함)
  ├─ Semgrep SAST                (AI flag 시 OWASP/security-audit 추가, tests/ 포함)
  ├─ Radon 복잡도                (CC > 20 = 차단)
  └─ pytest-cov                   (커버리지 < 70% = 차단)
  │
  ▼
[ Layer 2 ] llm-prereview.yml (LLM 의미론적 리뷰)
  Claude Sonnet 4.6 API → JSON 구조화 → PR 코멘트 자동 게시
  4차원 분석: 정확성·보안·통합·유지보수성
  CRITICAL → 머지 차단 | HIGH → 시니어 리뷰어 권장
  │
  ▼
[ Layer 3 ] PR 템플릿 (인간 리뷰)
  25항목 체크리스트 — 자동화가 못 잡는 비즈니스 로직·아키텍처 검증
```

---

## 보안 강화 사항 (Security Hardening)

이 리소스 팩에는 GitHub Actions 워크플로우 보안을 위한 다음 조치가 적용되어 있습니다:

| 항목 | 조치 | 이유 |
|------|------|------|
| 서드파티 액션 버전 고정 | `@v4` 대신 40자 커밋 SHA 사용 | 태그 재정의를 통한 공급망 공격 방지 |
| LLM 시스템 프롬프트 격리 | PR head가 아닌 base 브랜치에서 로드 | 공격자가 프롬프트 파일을 교체해 리뷰 결과를 조작하는 것을 방지 |
| LLM 출력 sanitize | PR 코멘트 게시 전 특수문자·멘션 제거 | 악성 링크·`@mention` 삽입 방지 |
| PR body / 브랜치명 안전 처리 | `env:` 블록을 통해 환경변수로 전달 | `run:` 블록 직접 interpolation을 통한 shell injection 방지 |
| 테스트 코드 보안 스캔 포함 | Bandit/Semgrep에서 `tests/` 제외 해제 | `conftest.py` 등 테스트 픽스처의 취약 코드 탐지 |

> 팀 레포에 복사할 때 SHA 값은 그대로 유지하세요. 업그레이드 시 [공식 릴리즈](https://github.com/actions/checkout/releases)에서 새 SHA를 확인하세요.

---

## 스택별 적용 가이드 (Adaptation)

### Python 프로젝트 (기본)
별도 설정 없이 즉시 사용 가능. Bandit, Radon, pytest-cov 자동 적용.

### JavaScript / TypeScript
`ai-code-quality-gate.yml` 하단 주석 처리된 JS 섹션 활성화 — `sast-js`, `coverage-js` 잡 사용.
`.pre-commit-config.yaml`의 ESLint/Prettier 섹션도 활성화.

### GitLab CI
`on:` → `workflow:`, `jobs:` 구조 동일하게 이식.
- [GitLab CI YAML 레퍼런스](https://docs.gitlab.com/ee/ci/yaml/)

### Bitbucket Pipelines
LLM 단계에서 `actions/github-script` 사용 불가 — `curl`로 Bitbucket REST API 직접 호출.
`llm-prereview.yml` 하단 주석에 변환 예시 포함.

---

## 트러블슈팅 (Troubleshooting)

### 워크플로우가 실행되지 않음
- **권한**: Settings > Actions > General > "Read and write permissions" 활성화
- **포크 PR**: 보안상 secrets 접근 불가 — 메인 레포에서 직접 실행 필요

### Gitleaks가 항상 실패함
- 조직 레포는 `GITLEAKS_LICENSE` secret 필요 ([gitleaks.io](https://gitleaks.io)에서 무료 발급)
- 개인 레포는 필요 없음

### LLM 리뷰가 항상 실패함
- `ANTHROPIC_API_KEY` 설정 확인 ([console.anthropic.com](https://console.anthropic.com))
- API 잔액 확인
- 모델명 확인: `claude-sonnet-4-6` 기본값 ([공식 모델 목록](https://platform.claude.com/docs/en/docs/about-claude/models/overview))

### LLM이 PR 코멘트를 게시하지 않음
- 워크플로우 권한: `pull-requests: write`가 활성화됐는지 확인
- diff가 500줄 초과 시 스킵됨 (`MAX_DIFF_LINES` 환경변수로 조정 가능)

### 게이트가 너무 엄격함 / 너무 느슨함
- `COMPLEXITY_LIMIT` (기본 20), `COVERAGE_THRESHOLD` (기본 70%) 조정
- Bandit 룰 제외: `.bandit` 파일 또는 `# nosec` 코멘트
- Semgrep 룰 제외: `.semgrepignore` 파일

### Pre-commit이 너무 느림
- `pre-commit run --files <specific files>` 으로 변경 파일만 검사
- 특정 commit에만 임시 스킵: `git commit --no-verify` (사용 자제)

---

## 참고 자료 (References)

### 공식 문서
- [Anthropic Models Overview](https://platform.claude.com/docs/en/docs/about-claude/models/overview) — 사용 가능한 Claude 모델
- [Gitleaks Action](https://github.com/gitleaks/gitleaks-action) — 공식 시크릿 스캔 액션
- [Semgrep CI Configs](https://semgrep.dev/docs/semgrep-ci/sample-ci-configs) — Semgrep CI 통합
- [Bandit Documentation](https://bandit.readthedocs.io) — Python 보안 분석
- [Radon Documentation](https://radon.readthedocs.io) — 코드 복잡도 측정
- [Pre-commit Framework](https://pre-commit.com) — Pre-commit hook 프레임워크

### 강의 관련
- [GilliLab NEWIT YouTube](https://www.youtube.com/@gillilab)
- [GilliLab Tech Log](https://techlog.gillilab.com)

---

## 라이선스

[MIT License](LICENSE) — 팀 레포에 자유롭게 복사·수정하여 사용하세요.
