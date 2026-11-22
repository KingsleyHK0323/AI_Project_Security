<div align="center">

  <h1>🛡️ AI Project Security</h1>

  <p>
    <b>A modular suite of production-grade AI Agent Skills for application security reviews, threat modeling, and pre-release security gates across backend, frontend, mobile, and desktop systems.</b>
  </p>

  <p>
    <img src="https://img.shields.io/github/license/KingsleyHK0323/AI_Project_Security?style=for-the-badge" alt="License: MIT" />
    <img src="https://img.shields.io/badge/Format-SKILL.md-blue?style=for-the-badge" alt="Format: SKILL.md" />
    <img src="https://img.shields.io/badge/Version-1.0.1-green?style=for-the-badge" alt="Version: 1.0.1" />
    <img src="https://img.shields.io/badge/Controls-126_Verifiable-black?style=for-the-badge" alt="Controls: 126 Verifiable" />
  </p>

  <p>
    <a href="#overview">Overview</a> •
    <a href="#skill-suite">Skill Suite</a> •
    <a href="#key-features">Key Features</a> •
    <a href="#installation">Installation</a> •
    <a href="#usage">Usage</a> •
    <a href="#repository-structure">Structure</a> •
    <a href="#contributing">Contributing</a> •
    <a href="#license">License</a>
  </p>

</div>

<hr />

## Overview

**AI Project Security** is an open-source suite of four interoperable `SKILL.md` specifications designed to equip AI coding assistants with structured, defense-in-depth application security checklists. Spanning **40 security domains** and **126 verifiable controls** aligned with OWASP Top 10, OWASP API Security Top 10, OWASP ASVS, and OWASP MASVS, the suite supports secure feature design, evidence-based code and architecture reviews, and pre-release security gates.

> [!IMPORTANT]
> **Defensive Scope Only:** These skills are built strictly for auditing, hardening, and reviewing systems you own or are authorized to assess. They enforce evidence-based reporting (`Pass`, `Partial`, `Fail`, `N/A`, `Not verified`) and prohibit generating working exploits or attack payloads.

<hr />

## Skill Suite

Select and install the skills that match your application architecture. Companion skills cross-reference each other without duplicating findings:

| Skill Directory | Skill Name (`SKILL.md`) | Target Architecture | Coverage |
|---|---|---|---|
| [`Backend_Security_Checklist/`](Backend_Security_Checklist/SKILL.md) | `backend-security-checklist` | General-purpose server-side services, REST/GraphQL/RPC APIs, background workers, databases, and CI/CD pipelines | 10 domains • 35 controls |
| [`Backend_App_Security_Checklist/`](Backend_App_Security_Checklist/SKILL.md) | `backend-app-security-checklist` | Backend APIs powering native mobile and consumer apps (device binding, anti-replay signing, push notifications, deep links, forced upgrades) | 10 domains • 28 controls |
| [`Frontend_Security_Checklist/`](Frontend_Security_Checklist/SKILL.md) | `frontend-security-checklist` | Browser-based web applications, SPAs, SSR/SSG apps, and PWAs (React, Vue, Angular, Next.js, Nuxt, Svelte, vanilla JS) | 10 domains • 37 controls |
| [`Frontend_App_Security_Checklist/`](Frontend_App_Security_Checklist/SKILL.md) | `frontend-app-security-checklist` | Native mobile (iOS, Android), cross-platform (React Native, Flutter), WebView, and desktop (Electron, Tauri) clients | 10 domains • 26 controls |

<hr />

## Key Features

<table>
  <tr>
    <td width="50%">
      <h3>🎯 Three Operational Modes</h3>
      <p>Adapts seamlessly to <b>Design Guidance</b> (controls per feature), <b>Code &amp; Architecture Review</b> (severity-rated findings table), and <b>Release Gates</b> (domain-grouped task checklists).</p>
    </td>
    <td width="50%">
      <h3>📊 Evidence-Based Auditing</h3>
      <p>Evaluates every applicable control with explicit status tags (<code>Pass</code>, <code>Partial</code>, <code>Fail</code>, <code>N/A</code>, <code>Not verified</code>) and severity ratings (<code>Critical</code> to <code>Info</code>).</p>
    </td>
  </tr>
  <tr>
    <td width="50%">
      <h3>🔗 Full-Stack Cross-Referencing</h3>
      <p>Links client-side defenses (such as route guards, CSRF tokens, and deep links) directly to their required server-side enforcement counterparts.</p>
    </td>
    <td width="50%">
      <h3>🧭 Industry-Standard Alignment</h3>
      <p>Maps all 126 traceable control IDs (e.g., <code>AUTHZ-01</code>, <code>INJ-01</code>, <code>XSS-02</code>, <code>STORE-01</code>) to OWASP ASVS, MASVS, Top 10, and API Security Top 10.</p>
    </td>
  </tr>
</table>

<hr />

## Installation

Clone the repository to obtain all four skills:

```bash
git clone [https://github.com/KingsleyHK0323/AI_Project_Security.git](https://github.com/KingsleyHK0323/AI_Project_Security.git)
cd AI_Project_Security
```

Install one or all skills into your AI assistant environment using each skill's canonical `name`:

<details>
<summary><b>Claude Code (CLI)</b></summary>

Copy each `SKILL.md` into a directory matching its YAML `name` field inside your personal (`~/.claude/skills/`) or project (`.claude/skills/`) skills folder:

```bash
# Install all four skills globally
mkdir -p ~/.claude/skills/backend-security-checklist \
         ~/.claude/skills/backend-app-security-checklist \
         ~/.claude/skills/frontend-security-checklist \
         ~/.claude/skills/frontend-app-security-checklist

cp Backend_Security_Checklist/SKILL.md ~/.claude/skills/backend-security-checklist/SKILL.md
cp Backend_App_Security_Checklist/SKILL.md ~/.claude/skills/backend-app-security-checklist/SKILL.md
cp Frontend_Security_Checklist/SKILL.md ~/.claude/skills/frontend-security-checklist/SKILL.md
cp Frontend_App_Security_Checklist/SKILL.md ~/.claude/skills/frontend-app-security-checklist/SKILL.md
```

</details>

<details>
<summary><b>Claude (Web &amp; Desktop Apps)</b></summary>

Package each skill folder (renamed to match its YAML `name`, containing `SKILL.md`) into a `.zip` file and upload it via **Settings → Capabilities / Skills**. Refer to Anthropic's documentation for current UI steps.

</details>

<details>
<summary><b>Cursor, Gemini, ChatGPT, and Custom Agents</b></summary>

For tools that do not natively load `SKILL.md` frontmatter, copy the Markdown body of the relevant `SKILL.md` file(s) (everything below the closing `---` YAML delimiter) into your project's rules file (e.g., `.cursor/rules`), Gems/Custom Instructions, or system prompt.

</details>

<hr />

## Usage

Once installed, prompt your assistant naturally when writing, reviewing, or shipping code:

```text
Review this FastAPI order endpoint for security issues, focusing on object-level
authorization, mass assignment, and error handling.
```

```text
Generate a pre-release security checklist for our React + Next.js web app
covering CSP headers, token storage, and third-party scripts.
```

```text
Audit our iOS Swift authentication flow and Universal Links configuration
before App Store submission.
```

<details>
<summary><b>Standard Security Review Output Structure</b></summary>

When performing a code or architecture review, each skill produces a structured report containing:

1. **Summary:** Scope, overall security posture, finding counts by severity, and top risks.
2. **Scope and Assumptions:** Reviewed files/configurations, unverified surfaces, and stack assumptions.
3. **Findings Table:** Traceable Control ID, Control Name, Severity (`Critical`, `High`, `Medium`, `Low`, `Info`), Status, Evidence, and Recommendation.
4. **Detailed Remediation:** Concrete minimal code or configuration fixes with verification steps.
5. **Passed Controls & Next Steps:** Verified controls and prioritized action items.

</details>

<hr />

## Repository Structure

```text
.
├── Backend_App_Security_Checklist/
│   └── SKILL.md      # Backend controls for mobile & consumer apps (10 domains, 28 controls)
├── Backend_Security_Checklist/
│   └── SKILL.md      # General-purpose backend & API security baseline (10 domains, 35 controls)
├── Frontend_App_Security_Checklist/
│   └── SKILL.md      # Native mobile, hybrid, WebView & desktop app controls (10 domains, 26 controls)
├── Frontend_Security_Checklist/
│   └── SKILL.md      # Browser-based web frontend & PWA security controls (10 domains, 37 controls)
├── .github/          # Repository workflows and funding configuration
├── README.md         # Suite documentation, comparison matrix, and installation guide
└── LICENSE           # MIT License
```

<hr />

## Contributing

Contributions are welcome, especially updates reflecting new platform security guidance (such as Android Keystore/Credential Manager updates, browser CSP/Trusted Types changes, or OWASP standard revisions).

1. Fork the repository and create a feature branch (`git checkout -b docs/update-owasp-mapping`).
2. Edit the relevant `SKILL.md` (and `README.md` if control counts or structure change), keeping YAML frontmatter valid.
3. Ensure all new controls include a unique ID, clear requirement, verification steps, and defensive-only scope.
4. Open a pull request describing what changed and why.

<hr />

## License

Released under the [MIT License](LICENSE). Copyright (c) 2026 Kingsley ([@KingsleyHK0323](https://github.com/KingsleyHK0323)).