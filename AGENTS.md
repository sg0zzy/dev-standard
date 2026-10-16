# Instructions for agents maintaining dev-standards

Read [README.md](README.md) and the policy that owns the area being changed. This repository contains documentation, templates and illustrative configuration only. MUST, SHOULD and MAY have the meanings in README.

- Preserve independent project implementation. Do not add a shared executable build system, mandatory parser, reusable central workflow dependency, or runtime service.
- Read [ARCHITECTURE.md](ARCHITECTURE.md) for architecture changes. Keep conditional technology preferences at SHOULD level, preserve justified alternatives, and never require framework adoption or a rewrite solely for standardization. Review data recovery, offline behavior, privacy and platform integration independently of stack choice.
- Keep one authoritative home for each rule; cross-reference it elsewhere. Update impacted examples, templates and [COMPLIANCE.md](COMPLIANCE.md) together.
- Preserve the distinction between build matrix, GitHub publication, and external distribution.
- Review version ordering, concurrent allocation, dirty-source identity, cleanup safety and release recovery whenever those contracts change.
- Inspect, edit and locally validate requested work. Do not push, make remote tags, upload credentials, publish or deploy without explicit authorization under [SECURITY.md](SECURITY.md). The initial creation task also prohibits creating any Git tags or releases.
- Check relative links, YAML examples, placeholder labels and coverage of the acceptance cases. Use temporary validation tools if available; do not introduce a shared implementation to demonstrate compliance.
- Do not claim an application complies without evidence from that application's implementation. Do not modify external repositories as part of standards maintenance unless separately requested.

Project adoption instructions belong in [templates/AGENTS.project.md](templates/AGENTS.project.md); this file governs the standards repository itself.
