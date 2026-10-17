# Agent Skills Collection

A curated collection of modular, production-grade **agent skills** designed for autonomous and pair-programming AI coding assistants (Google Antigravity, Claude Code, Cursor, Codex, and compatible agent frameworks).

Each skill encapsulates domain-specific operational workflows, architectural guidelines, deterministic quality gates, and formal verification methodologies via standardized `SKILL.md` instruction files.

---

## Available Skills

| Skill | Category | Description |
|---|---|---|
| **[`bend`](./bend/SKILL.md)** | Formal Systems & Compilers | Programming language combining Lean-like formal proofs, C-level speeds, and massive parallel execution on CPU/CUDA. |
| **[`bend-formalizer`](./bend-formalizer/SKILL.md)** | Formal Verification | End-to-end mathematical verification workflow: algorithm rewriting in Bend, invariant specification in `LAWS.bend`, and mechanical proofs in `PROOF.bend`. |
| **[`dod`](./dod/SKILL.md)** | High-Performance Systems | Data-Oriented Design for performance-critical applications: cache-conscious memory layouts, contiguous buffers, ECS, and vectorization. |
| **[`dop`](./dop/SKILL.md)** | Data Engineering | Data-Oriented Programming: pure transformations over immutable records, explicit schema validation, and separation of data from logic. |
| **[`hpc`](./hpc/SKILL.md)** | High-Performance Computing | Principles for scientific and analytical Python pipelines: SIMD vectorization, multiprocessing, memory locality, and fast I/O with NumPy/pandas. |
| **[`okf`](./okf/SKILL.md)** | Knowledge Representation | Open Knowledge Format (OKF): minimal, human-readable, and agent-parseable markdown/YAML standard for version-controlled domain knowledge. |
| **[`pr-reviewer`](./pr-reviewer/SKILL.md)** | Code Governance & CI | Automated pull request review assistant: git diff analysis, commit inspection, risk assessment, and structured review generation. |
| **[`quality-test`](./quality-test/SKILL.md)** | Software Quality & Testing | Comprehensive quality testing guide covering 21 objective dimensions (coverage, mutation, SAST, lint/typecheck, complexity, coupling, and CI quality gates). |
| **[`simulate-user-testing`](./simulate-user-testing/SKILL.md)** | QA & Usability | Realistic first-time user simulation framework to uncover onboarding friction, edge cases, UX bottlenecks, and unhandled errors. |
| **[`top`](./top/SKILL.md)** | Formal Methods & Types | Theorem-Oriented Programming applying the Curry–Howard isomorphism: requirements as theorems, types as propositions, implementations as proofs. |

---

## How to Use

### 1. Installation into your Workspace

Clone or add this repository as a submodule into your project's agent skills directory:

```bash
# Option A: Clone directly into workspace customizations
git clone https://github.com/k3ybladewielder/skills.git .agents/skills

# Option B: Add as a Git Submodule
git submodule add https://github.com/k3ybladewielder/skills.git .agents/skills

# Option C: Global configuration (e.g. Antigravity IDE global config)
git clone https://github.com/k3ybladewielder/skills.git ~/.gemini/config/skills
```

### 2. How Agents Discover and Load Skills

Skills adhere to the standard agent skill directory layout:
```text
.agents/skills/
└── <skill-name>/
    ├── SKILL.md       # Primary specification with YAML frontmatter
    ├── tests/         # Subskills, test guidelines, or checklists (optional)
    ├── scripts/       # Helper automation scripts (optional)
    └── references/    # In-depth architectural references (optional)
```

Compatible agents automatically index `SKILL.md` files upon initialization by reading their YAML metadata (`name`, `description`, `keywords`).

### 3. Prompting Agents to Activate a Skill

You can invoke skills explicitly or let your assistant activate them contextually based on the task:

- **Explicit Prompting:**
  ```text
  "Please review this pull request following the guidelines in /pr-reviewer."
  "Audit the complexity and test coverage of src/ using /quality-test."
  "Prove the correctness of our sorting algorithm using /bend."
  ```

- **Autonomous Agent Pairing:**
  When paired with agentic environments, the agent reads the skill's `SKILL.md` file using its file inspection tool before planning and executing changes.

---

## 📚 How to Cite

If you utilize this collection of agent skills or their formal verification and software quality methodologies in academic research, technical reports, or production toolchains, please cite this repository as follows:

### BibTeX

```bibtex
@software{guimaraes_skills_2026,
  author       = {Alysson Guimar{\~a}es},
  title        = {{Agent Skills Collection: Modular Engineering, Formal Verification, and Quality Testing Workflows for AI Agents}},
  month        = oct,
  year         = {2026},
  publisher    = {GitHub},
  version      = {1.0.0},
  url          = {https://github.com/k3ybladewielder/skills}
}
```

### APA Format

> Guimarães, A. (2026). *Agent Skills Collection: Modular Engineering, Formal Verification, and Quality Testing Workflows for AI Agents* (Version 1.0.0) [Computer software]. GitHub. https://github.com/k3ybladewielder/skills

---

## 📄 License

This project is licensed under the [Apache License 2.0](./LICENSE).
