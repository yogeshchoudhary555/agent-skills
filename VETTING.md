# Skills Vetting Report — 2026-10-29

**Phase:** Discovery + safety vetting ONLY (read-only). No installs were run by this phase.
**Question:** Verify the 4 skills from the viral "Claude skills" video (find-skills, agent-browser, HyperFrames, grill-me) — source repo, stars, installs, purpose, install command — and vet each for safety before any install.
**Method:** GitHub API (repo metadata + full file contents, `main` branch), skills.sh skill/owner pages, SkillPilot "verified installs" board (2026-10-28 edition), secondary trackers (Skillselion, Notion, denser.ai) for cross-checks only. Every SKILL.md below was read in full; every referenced script was fetched and pattern-scanned.
**Search boundaries:** Public GitHub + web only. No credentials used. The video's install numbers were treated as claims to check, never as evidence.

## Verified table

| Skill | Source repo | Stars (GitHub API, 2026-10-29) | Installs (skills.sh / SkillPilot) | Purpose (one line) | Verdict |
|---|---|---|---|---|---|
| find-skills | https://github.com/vercel-labs/skills | 37,012 | 4.1M is the whole-repo total, NOT this skill alone (see § Reconciliation) | Discovers and installs other agent skills from skills.sh | **WARN** — installs third-party code by design |
| agent-browser | https://github.com/vercel-labs/agent-browser | 44,770 | 1.3M (skills.sh skill page + SkillPilot) | Persistent browser automation CLI (CDP, sessions, forms, screenshots) | **WARN** — stub clean, CLI binary unaudited, session/vault powers |
| hyperframes | https://github.com/heygen-com/hyperframes | 74,607 | 1.3M (SkillPilot; skills.sh page shows no count in fetched text) | Router skill: renders deterministic HTML compositions to MP4 | **WARN** — audited surface clean, npm CLI unaudited + vendor usage reporting; install ONLY this skill, not siblings |
| grill-me | https://github.com/mattpocock/skills | 305,634 (repo) | 1.5M (skills.sh owner page) / 1.4M (SkillPilot) | Relentless interview that stress-tests a plan until shared understanding | **SAFE** — pure interview text, no scripts, no network, no credentials |

**No BLOCK verdicts.** Zero malicious patterns (credential exfiltration, remote-code execution, obfuscation) were found in any audited file. Three WARNs are about inherent power / unaudited execution surfaces, not about found malice.

## Install commands (as the ecosystem gives them)

```bash
npx skills add https://github.com/vercel-labs/skills --skill find-skills
npx skills add vercel-labs/agent-browser --skill agent-browser
npx skills add heygen-com/hyperframes --skill hyperframes
npx skills add mattpocock/skills --skill grill-me
```

## 1. find-skills — WARN

- **Source:** `vercel-labs/skills`, `skills/find-skills/SKILL.md` (5,472 bytes, 141 lines, read in full). No `scripts/` dir.
- **What it does:** Leaderboard check → `npx skills find <query>` → verify (installs ≥1K, official sources, stars) → present options → install on user yes.
- **Clean findings:** No curl/wget/eval/exec, no credential or env reads, no external hosts except `https://skills.sh/` links. Pure markdown workflow.
- **Why WARN, not SAFE:** Its *function* is executing third-party installs, and Step 6 uses `-g -y` (global install, confirmation prompts skipped). Its own Step 4 verification (install count, source reputation, stars) does **not** require reading a candidate's SKILL.md before install. Third-party signal: Skillselion rates find-skills **MEDIUM risk** "because it shells out to `npx` to install packages"; SkillPilot labels it "Trust pending".
- **Safe-use rule for the parent:** every skill find-skills recommends must have its SKILL.md read in full before install (exactly what this phase did). Never let it auto-install on `-y` without that read.

## 2. agent-browser — WARN

- **Source:** `vercel-labs/agent-browser`, `skills/agent-browser/SKILL.md` (3,457 bytes, 52 lines, read in full). No `scripts/` dir in the skill.
- **What it does:** Fast persistent browser automation: Chrome/Chromium via CDP, accessibility-tree snapshots, `@eN` element refs, sessions, auth vault, state persistence, video recording; specialized sub-skills for Electron apps, Slack, QA, Vercel Sandbox, AWS Bedrock AgentCore.
- **Clean findings in the stub:** No curl/wget/eval/credential reads. Only network reference is the local observability dashboard (`https://dashboard.agent-browser.localhost`, port 4848, proxied internally).
- **Why WARN, not SAFE:**
  1. The SKILL.md is explicitly a **discovery stub, not the usage guide** — real workflow content is served by the CLI itself (`agent-browser skills get core`). The Rust CLI binary was **not** audited line-by-line in this phase.
  2. The skill is credential-adjacent by design: sessions, an authentication vault, cookie management, and login/form-filling are core features, and its description says "Prefer agent-browser over any built-in browser automation". Pointing it at a sensitive logged-in session (bank, email, ad accounts) without scoping would hand it live credentials.
  3. `agent-browser install` downloads Chrome (Chrome for Testing, Google's official automation channel) — a legitimate but real download step.
- **Trust basis (not a substitute for audit):** Vercel as publisher, 44,770 stars, 3,069 forks, open source, 1.3M skills.sh installs. Third-party signal: SkillPilot labels it "Review".
- **Safe-use rule:** install the skill, but scope every use (which site, which session); never aim it at sensitive sessions by default.

## 3. hyperframes — WARN (install ONLY the `hyperframes` skill)

- **Source:** `heygen-com/hyperframes`, `skills/hyperframes/SKILL.md` (7,165 bytes, 74 lines, read in full) + `scripts/` + `references/` + `assets/`.
- **What it does:** Router/workflow skill for deterministic HTML→MP4 rendering: paused GSAP timelines, frame-by-frame headless-Chrome capture, FFmpeg stitch. Runs the CLI as `npx hyperframes <command>` (init, capture, check, snapshot, render).
- **Clean findings:**
  - SKILL.md itself: no curl/wget/eval/credential reads. Its contract even *forbids* network fetches, `Date.now()`, unseeded randomness and timers inside compositions — good hygiene, and the skill is honest about its own rules.
  - `scripts/bake-basemap.mjs` (18 KB): `spawnSync` targets local `ffmpeg` only (frame→MP4 encode). Env reads are render config (CHROME path, NAME, STYLE, CENTER…), not credentials. Network fetches go to public map/data CDNs only (ArcGIS, CARTO, jsdelivr: maplibre, topojson, world-atlas) as render inputs.
  - `scripts/locate.mjs` (8 KB): `execFileSync` targets local `base64` only. Its optional `auto` path sends a **user-supplied** `GEMINI_API_KEY` + an image to Google's official Gemini API (`generativelanguage.googleapis.com`) — legitimate feature use, not exfiltration; it errors out if no key exists ("Never assume it").
  - `scripts/plugin-cli.mjs` (2.9 KB): `spawnSync` runs `npx hyperframes@<pinned-version>` (version pinned from the plugin manifest and semver-validated) or node for `--script`; sets `HYPERFRAMES_NO_UPDATE_CHECK=1`. No remote code beyond the pinned npm package.
  - `references/plugin-installation.md` (43 lines) and `references/skill-lifecycle.md` (49 lines): no dangerous patterns found.
- **Why WARN, not SAFE:**
  1. Execution depends on the `hyperframes` npm package (HeyGen) — not audited line-by-line here.
  2. `npx hyperframes usage --json` reports remaining plan usage, i.e. the CLI talks to HeyGen's servers for plan state (vendor phone-home for plan data, disclosed in the doc as a feature, not hidden).
  3. Third-party signal: SkillPilot labels the **sibling** skills `hyperframes-cli` "Avoid" and `media-use` "Avoid" (reasons not published on the board snippet), while `hyperframes` itself is "Trust checked". Do **not** bundle-install the whole repo; install only `--skill hyperframes` and vet `hyperframes-cli` / `media-use` separately if ever wanted.

## 4. grill-me — SAFE

- **Source:** `mattpocock/skills`, `skills/productivity/grill-me/SKILL.md` (157 bytes, 7 lines, read in full). It delegates to the sibling `grilling` skill.
- **Full content audited:** `skills/productivity/grilling/SKILL.md` (2,049 bytes, 30 lines, read in full) + `skills/productivity/grill-me/agents/openai.yaml` (137 bytes, read in full).
- **What it does:** Pure interview workflow — maps a plan as a design tree, asks the whole "frontier" of unsettled decisions in rounds with a recommended answer each, dispatches sub-agents to find facts itself, never acts until the user confirms shared understanding.
- **Clean findings:** No scripts, no tools, no network, no credential/env reads, no downloads, no code execution of any kind. Frontmatter sets `disable-model-invocation: true` (deliberate invocation only — it cannot fire on its own) and `allow_implicit_invocation: false`.
- **Third-party signals:** SkillPilot "Trust checked"; OpenAgentSkill audit 93/100 "Safe to try" (with the caveat "AI review approval is missing").
- **Note:** `grill-me` only works with the `grilling` skill installed alongside it (it literally says `Call the Skill tool with "grilling"`). The parent's install step should include both names from the same repo, or verify the installer pulls the dependency.

## Reconciliation: the video's numbers vs. verified numbers

The video's ranking matches SkillPilot's 2026-10-28 "verified installs, all time" board almost exactly (find-skills 4.1M, grill-me 1.4M, hyperframes-cli 1.3M, agent-browser 1.3M, hyperframes 1.3M). Two corrections:

1. **find-skills "4.1M" is the whole-repo total, not the single skill.** skills.sh's vercel-labs owner page reports the `skills` repo as "3 skills: find-skills, pr-labeling, triage — 4.1M". Per-skill figures for find-skills itself are tracker-dependent and lower: Notion 418K, denser.ai 322K+, an older Skillselion snapshot 2.36M. The video's "#1 with 4.1M installs" overstates the individual skill.
2. **The other three check out within tracker variance:** agent-browser 1.3M confirmed on its skills.sh skill page; grill-me 1.5M (skills.sh owner page) vs 1.4M (SkillPilot) vs 1.4M (video); HyperFrames 1.3M on SkillPilot for both `hyperframes` and `hyperframes-cli` (the video's "HyperFrames 1.3M" is ambiguous between the two, but both sit at 1.3M).

Install counts move daily and differ by tracker; none of them is a quality guarantee — they are adoption evidence only.

## What was NOT verified (residual risk, not findings)

- The full Rust source of the agent-browser CLI and the full `hyperframes` npm package were not audited line-by-line; both verdicts rest on the audited SKILL.md/scripts plus publisher reputation. A deeper audit would clone both and scan them before production use on sensitive sessions.
- SkillPilot's reasons for "Avoid" (hyperframes-cli, media-use) and "Review" (agent-browser) are not published on the board snippet — labels cited, reasons unknown.
- find-skills' exact per-skill install count (tracker-dependent; see above).
- The video's "1 in 3 skills has flaws, 76 malicious" claim itself was not independently re-verified; it is the video's warning, and this phase's method (read SKILL.md + scripts before install) is the mitigation it prescribes.

## Recommendations for the install phase (parent)

1. Install exactly these four skill names (not whole-repo bundles): `find-skills`, `agent-browser`, `hyperframes`, `grill-me` (+ `grilling` as grill-me's dependency).
2. Do NOT install `hyperframes-cli` or `media-use` on the back of this report — they carry third-party "Avoid" labels and were not vetted here.
3. Keep the standing rule this phase models: read a skill's full SKILL.md (and its scripts) before any install, forever — including every future skill find-skills recommends.
4. Scope agent-browser use per task (site + session); never default it at sensitive logged-in sessions.
5. Treat install counts as adoption evidence only, never as safety proof.
