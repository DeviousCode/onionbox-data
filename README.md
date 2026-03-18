# onionbox-data

> Structured Minecraft modding API databases — extracted, normalized, and free to use.

This repository contains SQLite databases built from the source code of popular Minecraft modding platforms. It powers [OnionBox](https://onionbox.dev) — but the data itself belongs to the community.

> **Note:** OnionBox does not provide or sell AI access. Users connect their own AI provider via API key or OAuth — OpenAI, Anthropic Claude, Google Gemini, or a local model. OnionBox is the IDE and context layer. Your AI, your subscription, your control.

---

## Supported Platforms

| Platform | Family | Status |
|----------|--------|--------|
| Paper | bukkit | ✅ Active |
| Spigot | bukkit | ✅ Active |
| Bukkit | bukkit | ✅ Active |
| Fabric | fabric | ✅ Active |
| Forge | forge | ✅ Active |
| NeoForge | neoforge | ✅ Active |

---

## Structure

```
onionbox-data/
├── manifests/
│   ├── paper/
│   ├── fabric/
│   ├── forge/
│   ├── neoforge/
│   └── ...
├── index.json        ← master index of all available platforms and versions
└── LICENSE
```

Each `.sqlite` file follows the naming convention:

```
{platform}-{version}-{hash8}.sqlite
```

The 8-character hash is derived from the source JAR's SHA-256 fingerprint, ensuring each database is uniquely tied to the exact upstream source it was built from.

---

## Database Schema

Each database is a fully self-contained SQLite file. All tables are created fresh per build — no migrations needed at runtime.

---

## Our Tooling

The databases here are produced by a custom two-stage TypeScript pipeline built by [DeviousCode](https://github.com/DeviousCode). The upstream API data belongs to the community — the tooling that makes it useful is ours.

### Stage 1 — Extraction

Platform-specific extractors parse each platform's source JAR (or Javadoc JAR) without requiring a JDK. JARs are decompressed in-memory using `fflate` and routed to the correct extractor:

Each platform has a purpose-built extractor that understands the platform's specific source structure, event patterns, and cancellable conventions. Extractors run without requiring a JDK — source JARs are decompressed and parsed in-memory.

### Stage 2 — Conversion

The converter takes the extracted JSON and builds the SQLite database in a single atomic transaction:

- Stable `symbol_id` values are SHA-256 hashes of `kind|fullName|signature` — consistent across rebuilds
- Every event, method, and command is inserted into both the domain tables and the symbol graph
- `declares_method` relation edges are created between event symbols and their method symbols
- FTS5 indexes are rebuilt after the transaction completes
- Source hash validation ensures only correctly-formatted 64-hex SHA-256 values are accepted
- Failed builds clean up the partial `.sqlite` file automatically

---

## Attribution

The API metadata in this repository is derived from the following open source projects:

| Project | License | Source |
|---------|---------|--------|
| [Paper](https://github.com/PaperMC/Paper) | GPL-3.0 | PaperMC |
| [Spigot](https://hub.spigotmc.org/stash/projects/SPIGOT) | LGPL-3.0 | SpigotMC |
| [Bukkit](https://github.com/Bukkit/Bukkit) | GPL-3.0 | Bukkit |
| [Fabric API](https://github.com/FabricMC/fabric) | Apache-2.0 | FabricMC |
| [Forge](https://github.com/MinecraftForge/MinecraftForge) | LGPL-2.1 | MinecraftForge |
| [NeoForge](https://github.com/neoforged/NeoForge) | LGPL-3.0 | NeoForged |

We do not claim ownership over any Minecraft API, event, or class data. All platform names, class names, and API structures belong to their respective owners.

---

## Updates

Databases are updated as new platform versions are released. Watch the repo or check `index.json` to stay current.

---

## License

[CC0 1.0 Universal](LICENSE) — public domain. No rights reserved.

The extraction and conversion tooling used to produce these databases is proprietary to DeviousCode and is not included in this repository.

---

## About OnionBox

OnionBox is a custom-built IDE for Minecraft plugin and mod developers. Connect your own AI provider via API key or OAuth — OpenAI, Anthropic Claude, Google Gemini, or a local model. OnionBox loads the exact API context your AI needs, version-specific and platform-specific, so you can just prompt and build.

→ [onionbox.dev](https://onionbox.dev)

*Not affiliated with Mojang AB or Microsoft.*
