# Agent Skills — Jerry's installed skills

> Adopted 2026-10-29 from the agent-skills ecosystem (skills.sh). Every skill below was
> safety-vetted — full SKILL.md plus referenced scripts read — BEFORE install.
> Full evidence: [VETTING.md](VETTING.md).
>
> Standing rule: read a skill's full SKILL.md (and its scripts) before installing it,
> forever — including every future skill that find-skills recommends. Install counts are
> adoption evidence only, never safety proof.

## Installed skills

| Skill | Source repo | Stars | Installs | Purpose | Verdict |
|---|---|---|---|---|---|
| find-skills | https://github.com/vercel-labs/skills | 37,012 | 4.1M is the whole-repo total, NOT this skill alone | Discovers and installs other agent skills from skills.sh | WARN — installs third-party code by design |
| agent-browser | https://github.com/vercel-labs/agent-browser | 44,770 | 1.3M | Persistent browser automation CLI (CDP, sessions, forms, screenshots) | WARN — stub clean, Rust CLI unaudited, session/vault powers |
| hyperframes | https://github.com/heygen-com/hyperframes | 74,607 | 1.3M | Router skill: deterministic HTML → MP4 rendering | WARN — install ONLY this skill, NOT hyperframes-cli / media-use |
| grill-me (+ grilling) | https://github.com/mattpocock/skills | 305,634 (repo) | 1.5M / 1.4M | Relentless interview that stress-tests a plan until shared understanding | SAFE — pure text, no scripts, no network, no credentials |

No BLOCK verdicts. Zero malicious patterns (credential exfiltration, remote-code
execution, obfuscation) were found in any audited file. The three WARNs are about
inherent power / unaudited execution surfaces, not about found malice.

## Install commands

```bash
npx skills add https://github.com/vercel-labs/skills --skill find-skills
npx skills add vercel-labs/agent-browser --skill agent-browser
npx skills add heygen-com/hyperframes --skill hyperframes
npx skills add mattpocock/skills --skill grill-me
npx skills add mattpocock/skills --skill grilling
```

Installed with `-y --copy` into the agent skills directory
(`~/.agents/skills/` on Jerry's machine).

## Safe-use rules

1. Read a candidate skill's full SKILL.md before install — never let find-skills
   auto-install on `-y` without that read.
2. Scope every agent-browser use (which site, which session); never aim it at
   sensitive logged-in sessions (bank, email, ad accounts) by default.
3. Never install `hyperframes-cli` or `media-use` on the back of this vetting —
   they carry third-party "Avoid" labels and were not vetted here.
4. grill-me only works with the `grilling` skill installed alongside it.
5. The video's numbers, corrected: find-skills "4.1M" is the vercel-labs/skills
   repo total (3 skills), not the single skill. The other three check out within
   tracker variance.
