<p align="center">
  <img src="https://play-lh.googleusercontent.com/hoRZ568WPBkAGlEZiBH2XjFjeojcZHIfoeV4k3k5FX2WECK26gRHDXp2O4Bf1s14A-OLjYwUBr0uLosYHqrPeAQ=w240-h480-rw" alt="KalimantanBio" width="180" height="180" />
</p>

<h1 align="center">KalimantanBio</h1>

<p align="center">
  <strong>Biodiversity Knowledge Platform</strong>
</p>

<p align="center">
  Discover the biodiversity of Kalimantan.<br />
  Explore species, relationships, taxonomy, comparisons, and scientific knowledge.
</p>

<p align="center">
  <strong>One platform. Five independent modules.</strong>
</p>

<p align="center">
  <a href="https://kalimantanbio.com/repository/">
    <img src="https://img.shields.io/badge/Existing%20Platform-kalimantanbio.com-238636?style=for-the-badge&logo=github" alt="Existing Platform" />
  </a>
  <a href="https://gusti-alfarisy.github.io/blog/2026/pbl-fp-2026/#kalimantanbio-biodiversity-knowledge-platform">
    <img src="https://img.shields.io/badge/Project%20Spec-gusti--alfarisy.github.io-1f6feb?style=for-the-badge&logo=book" alt="Project Specification" />
  </a>
  <a href="https://github.com/BorneoBiodiverse">
    <img src="https://img.shields.io/badge/Organization-BorneoBiodiverse-8b5cf6?style=for-the-badge&logo=github" alt="Organization" />
  </a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Rust-Axum-000000?style=flat-square&logo=rust&logoColor=white" alt="Axum" />
  <img src="https://img.shields.io/badge/Python-Django-092E20?style=flat-square&logo=django&logoColor=white" alt="Django" />
  <img src="https://img.shields.io/badge/Functional%20Programming-Principles-8b5cf6?style=flat-square&logo=lambda&logoColor=white" alt="Functional Programming" />
  <img src="https://img.shields.io/badge/Architecture-Loosely%20Coupled-238636?style=flat-square" alt="Loosely Coupled" />
  <img src="https://img.shields.io/badge/Development-Concurrent-f97583?style=flat-square" alt="Concurrent Development" />
</p>

---

## 🌴 Existing KalimantanBio Platform

The broader KalimantanBio ecosystem and production context:

[https://kalimantanbio.com/repository/](https://kalimantanbio.com/repository/)

<p align="center">
  <img src="https://kalimantanbio.com/media/species/Agathis_borneensis_Warburg_01_accept.jpg" alt="Kauri Borneo - Agathis borneensis" width="300" />
</p>

<p align="center">
  <em>Kauri Borneo (Agathis borneensis) — Endangered conifer endemic to Borneo</em>
</p>

The existing platform provides a comprehensive biodiversity information system including:

- **Species database** with 100+ documented records
- **Taxonomic classification** and conservation status (IUCN)
- **Spatial distribution mapping** across Kalimantan
- **Forest coverage visualization** via GIS
- **Repository and dataset services** for research
- **Species identification tools**
- **Research permit management** with institutional partners

The five Functional Programming modules extend this ecosystem with specialized intelligent and analytical capabilities for biodiversity knowledge exploration.

---

## 🌿 Explore the Five Modules

```mermaid
flowchart TB
    KB["KalimantanBio<br/>Biodiversity Knowledge Platform"]

    KB --> M1["Intelligent Species Search"]
    KB --> M2["Species Relationship Explorer"]
    KB --> M3["Taxonomy & Classification Explorer"]
    KB --> M4["Comparative Species Explorer"]
    KB --> M5["Biodiversity Knowledge & Citation Explorer"]

    style KB fill:#1f6feb,color:#fff
    style M1 fill:#238636,color:#fff
    style M2 fill:#238636,color:#fff
    style M3 fill:#238636,color:#fff
    style M4 fill:#238636,color:#fff
    style M5 fill:#238636,color:#fff
```

The modules share a common project identity and biodiversity domain, but are designed for independent development with minimal technical coupling.

---

## 📦 One Platform. Five Independent Modules.

| Module | Focus | Capabilities |
|--------|-------|--------------|
| **🔍 Intelligent Species Search** | Species discovery | Natural-language queries, multi-attribute filtering, relevance ranking, related-query recommendations |
| **🕸️ Species Relationship Explorer** | Relationship exploration | Related species discovery, relationship scoring, explanations, interactive networks |
| **🌳 Taxonomy & Classification Explorer** | Taxonomic understanding | Interactive taxonomic tree, taxon-based explorer, coverage analysis, endemic taxa, gap analysis |
| **⚖️ Comparative Species Explorer** | Species comparison | Multi-species comparison, shared/unique attributes, similarity scoring, distinguishing characteristics |
| **📚 Biodiversity Knowledge & Citation Explorer** | Scientific knowledge | Species-to-publication links, topic/location exploration, research coverage, understudied species, citation export |

Each module is a first-class component with:
- Clearly defined responsibility
- Independent development, testing, and documentation
- Separate repository and team
- Minimal cross-module dependencies
- Independent implementation lifecycle

---

## 🔗 How the Modules Fit Together

### Conceptual Relationship (Not Technical Dependency)

```mermaid
flowchart TB
    subgraph DISCOVER["DISCOVER"]
        M1["🔍 Intelligent Species Search"]
    end

    subgraph EXPLORE["EXPLORE"]
        M2["🕸️ Species Relationship Explorer"]
        M3["🌳 Taxonomy & Classification Explorer"]
    end

    subgraph UNDERSTAND["UNDERSTAND"]
        M4["⚖️ Comparative Species Explorer"]
        M5["📚 Biodiversity Knowledge & Citation Explorer"]
    end

    M1 -.-> M2
    M1 -.-> M3
    M2 -.-> M4
    M3 -.-> M4
    M2 -.-> M5
    M3 -.-> M5
    M4 -.-> M5

    style DISCOVER fill:#f6f8fa,stroke:#d0d7de
    style EXPLORE fill:#f6f8fa,stroke:#d0d7de
    style UNDERSTAND fill:#f6f8fa,stroke:#d0d7de
```

> **Dashed lines indicate conceptual complementarity, not technical dependencies.**  
> A user may benefit from using multiple modules, but no module requires another to function.

### Development Model

```text
Team 1 ──→ 🔍 Intelligent Species Search
Team 2 ──→ 🕸️ Species Relationship Explorer
Team 3 ──→ 🌳 Taxonomy & Classification Explorer
Team 4 ──→ ⚖️ Comparative Species Explorer
Team 5 ──→ 📚 Biodiversity Knowledge & Citation Explorer
```

Concurrent development with minimal coordination bottlenecks.

---

## ⚡ Functional Programming

The modules apply Functional Programming principles where appropriate:

| Principle | Application |
|-----------|-------------|
| **Pure functions** | Data transformation and analysis logic |
| **Immutability** | Predictable state management |
| **Function composition** | Reusable analytical pipelines |
| **Higher-order functions** | Flexible filtering, ranking, transformation |
| **Declarative transformations** | Biodiversity data processing |

Functional Programming is part of the engineering approach, not the entire product identity.

---

## 🛠️ Technology Stack

The project specification recommends:

| Technology | Role | Badge |
|------------|------|-------|
| **Axum** | Rust web framework for performant APIs | ![Axum](https://img.shields.io/badge/Axum-Rust%20Web%20Framework-000000?style=flat-square&logo=rust&logoColor=white) |
| **Django** | Python framework for interfaces & light computation | ![Django](https://img.shields.io/badge/Django-Python%20Framework-092E20?style=flat-square&logo=django&logoColor=white) |

Actual technology choices per module are determined by each team and documented in their respective repositories.

---

## 🏗️ Architecture Overview

```
KalimantanBio
├── 🔍 Intelligent Species Search
├── 🕸️ Species Relationship Explorer
├── 🌳 Taxonomy & Classification Explorer
├── ⚖️ Comparative Species Explorer
└── 📚 Biodiversity Knowledge & Citation Explorer
```

Each repository contains its own:
- README with setup and usage instructions
- Architecture and implementation documentation
- Tests and development workflow
- Independent release cycle

---

## 🔗 Project Links

| Resource | Link |
|----------|------|
| **Project Specification** | <https://gusti-alfarisy.github.io/blog/2026/pbl-fp-2026/#kalimantanbio-biodiversity-knowledge-platform> |
| **Existing Platform** | <https://kalimantanbio.com/repository/> |
| **GitHub Organization** | <https://github.com/BorneoBiodiverse> |
| **Project Management** | Trello (primary task management) |

---

## 📋 Project Management

**Trello** is the primary task-management system for planning, assignments, progress tracking, and team coordination.

GitHub is used for:
- Source code and version control
- Branches, pull requests, and code review
- Releases and technical documentation
- Repository collaboration

---

## 👥 Team & Contributors

### Project Leader

| | |
|---|---|
| <a href="https://github.com/sleepymor"><img src="https://avatars.githubusercontent.com/u/87416908?v=4" width="80" height="80" style="border-radius: 50%;" /></a> | **[@sleepymor](https://github.com/sleepymor)** — Project Leader & Organization Owner |
| | KalimantanBio: Biodiversity Knowledge Platform |

### Organization Members

<!-- ORG_MEMBERS_START -->

<table><tbody><tr><td align="center" width="25%"><a href="https://github.com/Alizah14"><img src="https://avatars.githubusercontent.com/u/233382592?v=4" width="60" height="60" style="border-radius: 50%;" /><br /><sub><b>@Alizah14</b></sub></a></td><td align="center" width="25%"><a href="https://github.com/atechforce"><img src="https://avatars.githubusercontent.com/u/208674950?v=4" width="60" height="60" style="border-radius: 50%;" /><br /><sub><b>@atechforce</b></sub></a></td><td align="center" width="25%"><a href="https://github.com/Bieraaa"><img src="https://avatars.githubusercontent.com/u/202978180?v=4" width="60" height="60" style="border-radius: 50%;" /><br /><sub><b>@Bieraaa</b></sub></a></td><td align="center" width="25%"><a href="https://github.com/Jayennn"><img src="https://avatars.githubusercontent.com/u/108043334?v=4" width="60" height="60" style="border-radius: 50%;" /><br /><sub><b>@Jayennn</b></sub></a></td></tr><tr><td align="center" width="25%"><a href="https://github.com/Kiana0130"><img src="https://avatars.githubusercontent.com/u/188063291?v=4" width="60" height="60" style="border-radius: 50%;" /><br /><sub><b>@Kiana0130</b></sub></a></td><td align="center" width="25%"><a href="https://github.com/nirvanaguys"><img src="https://avatars.githubusercontent.com/u/289380515?v=4" width="60" height="60" style="border-radius: 50%;" /><br /><sub><b>@nirvanaguys</b></sub></a></td><td align="center" width="25%"><a href="https://github.com/NotHydra"><img src="https://avatars.githubusercontent.com/u/86897187?v=4" width="60" height="60" style="border-radius: 50%;" /><br /><sub><b>@NotHydra</b></sub></a></td><td align="center" width="25%"><a href="https://github.com/peroyoo"><img src="https://avatars.githubusercontent.com/u/324713925?v=4" width="60" height="60" style="border-radius: 50%;" /><br /><sub><b>@peroyoo</b></sub></a></td></tr><tr><td align="center" width="25%"><a href="https://github.com/sleepymor"><img src="https://avatars.githubusercontent.com/u/87416908?v=4" width="60" height="60" style="border-radius: 50%;" /><br /><sub><b>@sleepymor</b></sub></a></td><td align="center" width="25%"><a href="https://github.com/Thisyath"><img src="https://avatars.githubusercontent.com/u/204345564?v=4" width="60" height="60" style="border-radius: 50%;" /><br /><sub><b>@Thisyath</b></sub></a></td><td></td><td></td></tr></tbody></table>

*Last updated: 2026-09-05 12:37 UTC*
<!-- ORG_MEMBERS_END -->

*Last updated: $(date -u +"%Y-%m-%d %H:%M UTC")*

### Module Teams

Five teams developing concurrently across five independent modules:

| Team | Module | Focus |
|------|--------|-------|
| Team 1 | 🔍 Intelligent Species Search | Natural-language species discovery & filtering |
| Team 2 | 🕸️ Species Relationship Explorer | Species relationships & network exploration |
| Team 3 | 🌳 Taxonomy & Classification Explorer | Taxonomic structure & classification |
| Team 4 | ⚖️ Comparative Species Explorer | Multi-species comparison & similarity |
| Team 5 | 📚 Biodiversity Knowledge & Citation Explorer | Scientific literature & citation exploration |

> **Note:** Organization members are private. Team members contribute through their respective module repositories.

---

## 🤝 Contributing

Each module repository defines its own contribution guidelines. Please refer to the individual repository READMEs for setup, development workflow, and contribution processes.

---

## 📄 License

Individual modules may have their own licenses. Check each repository for details.

---

<p align="center">
  <em>KalimantanBio — Making biodiversity knowledge easier to discover, explore, understand, compare, and reference.</em>
</p>

<p align="center">
  <img src="https://kalimantanbio.com/media/species/Img_Raja_Udang_Kalung-biru_accept.jpg" alt="Raja Udang Kalung-biru" width="200" />
  <br />
  <em>Raja Udang Kalung-biru (Alcedo euryzona) — Critically Endangered</em>
</p>
