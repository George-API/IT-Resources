# Data Management — Systems Thinking Reference

Data management is a single system with one lifecycle. Every concept—governance, engineering, quality, security, analytics—is a view of the same data flowing through the same pipeline. All topics connect through one lifecycle and four repeating patterns. This document integrates the content across all [data management](README.md) docs into one systems-oriented reference.

```
  ┌────────────────────────────────────────────────────────────┐
  │                  THE DATA LIFECYCLE                        │
  │                                                            │
  │   Ingest ──▶ Store ──▶ Transform ──▶ Serve ──▶ Govern    │
  │      ▲                                           │         │
  │      └───────────── feedback & quality ──────────┘         │
  └────────────────────────────────────────────────────────────┘
```

| Phase | What happens | Key docs |
|-------|-------------|----------|
| **Ingest** | Capture from source systems (batch, streaming, CDC) | [Engineering](concepts/core/engineering.md) |
| **Store** | Land, organize, retain (lake, warehouse, database) | [Data Stores](concepts/core/data_stores.md), [Database](concepts/core/database.md) |
| **Transform** | Clean, model, enrich (Bronze → Silver → Gold) | [Engineering](concepts/core/engineering.md), [Analytics Eng](concepts/core/analytics_eng.md) |
| **Serve** | Deliver to consumers (reports, APIs, ML features) | [Analytics](concepts/core/analytics.md), [Power BI](concepts/platform/powerbi.md) |
| **Govern** | Own, protect, measure, enforce, retire | [Governance](concepts/core/governance.md), [Security](concepts/core/security.md), [Operations](concepts/core/operations.md) |

Govern is not a separate phase—it runs across all others. Quality gates at each boundary. Security controls at every layer. Lineage traced end-to-end. If governance only happens at the end, it's an audit, not governance.

---

## Governing Principles

These 10 rules explain 80% of correct data decisions. When in doubt, check here first.

1. **Profile before building.** Source data surprises cause more rework than bad transformation logic. Spend a day profiling to save weeks of wrong assumptions.
2. **Quality at the source, not downstream.** No transformation fixes fundamentally wrong inputs. Polishing garbage produces polished garbage → *GIGO Pattern*.
3. **Every data decision is a trade-off.** There is no "best" architecture—only the best given your constraints. Different volumes, freshness, compliance, and team skill yield different correct answers → *Constraint Pattern*.
4. **Idempotency is not optional.** Pipelines will be rerun. If a rerun produces different results, the pipeline is broken. Design for MERGE/upsert, partition overwrites, or delete-and-reload.
5. **Data attracts everything.** Once data lives on a platform, applications, services, and more data gravitate toward it → *Gravity Pattern*. Platform decisions are sticky. Factor this into every platform choice.
6. **Schema will change.** Source systems evolve without notice. Design for schema evolution, not schema stability. Detect drift before it causes silent failures.
7. **What you don't observe controls you.** Unmeasured freshness, invisible quality drift, and undocumented lineage compound silently → *Visibility Pattern*. Monitor everything that matters.
8. **Start with batch.** 80% of workloads don't need streaming. Streaming adds complexity, cost, and operational burden. Add it when a real business requirement demands sub-minute latency, not before.
9. **Domain ownership over central bottlenecks.** Federated governance: central team sets standards, domain teams own their data products. Central teams that own all data become bottlenecks.
10. **The consumer defines value.** Work backward from how data will be used. Who needs it, in what format, how fresh, for what decisions? If nobody consumes it, it has no value.

---

## Repeating Patterns

Four structures recur across every data management topic. Recognize one, predict behavior, intervene earlier.

### 1. The GIGO Pattern (Garbage In, Garbage Out)

Quality degrades downstream. Every transformation that processes bad input amplifies the error. Quality must be enforced at boundaries, not patched at the end.

```
  Source (bad) ──▶ Ingest ──▶ Transform ──▶ Serve ──▶ Decision
                                                        │
                    ← error amplifies at each step ──────┘
```

**Instances**:
- Wrong source field meaning → confidently wrong reports → bad business decisions
- Undetected duplicates at ingestion → inflated aggregates → overstated KPIs
- Unvalidated schema assumptions → silent pipeline failures → stale or missing data served as current
- Including test/internal accounts in production data → corrupted analytics → eroded trust

**Countermeasure**: Quality gates at each layer boundary (Bronze → Silver → Gold). Validate at ingestion. Don't push quality checks downstream.

### 2. The Gravity Pattern

Data attracts applications, services, and more data. The longer data lives somewhere, the harder it is to move. Platform decisions compound.

```
  Small dataset lands on Platform A
    → applications connect to it
      → more data lands to join with it
        → dependent services build on it
          → migration cost grows exponentially
            → platform choice is now permanent
```

**Instances**: Data lake platform selection, cloud region choice, warehouse vendor, catalog tool.
**Countermeasure**: Factor gravity into every platform decision. Abstract where practical. Recognize that "temporary" platforms become permanent.

### 3. The Visibility Pattern

What you can't see controls you. Most data failures trace back to something invisible.

| Invisible thing | What breaks | Countermeasure |
|-----------------|-------------|----------------|
| Schema drift | Silent wrong data, pipeline failures | Schema validation at ingestion, drift detection |
| Stale data served as fresh | Decisions made on outdated information | Freshness SLOs, staleness alerts |
| Undocumented business rules | Different teams interpret the same field differently | Explicit data contracts, business glossary |
| Missing lineage | Can't trace errors, can't audit, can't trust | End-to-end lineage tooling |
| Shadow datasets | Ungoverned copies drift from source of truth | Catalog discovery, access controls |
| Silent data loss | Pipeline completes "successfully" but drops records | Row count reconciliation at every boundary |

### 4. The Constraint Pattern

Every data project has constraints that collectively narrow the solution space.

```
  Volume ── Freshness ── Compliance ── Team skill ── Budget ── Legacy
      │          │            │             │           │         │
      └──────────┴────────────┴─────────────┴───────────┴─────────┘
                                    │
                        Collectively determine ──▶ the right architecture
```

Different constraints yield different correct answers. If you change the constraints, the architecture should change. If someone proposes the same architecture regardless of constraints, they're applying a pattern, not solving the problem.

---

## Never Rules & First-Step Rules

### Never Rules

| # | Rule | Because |
|---|------|---------|
| 1 | Never build pipelines before profiling source data | Source surprises cause more rework than anything else |
| 2 | Never trust the source system blindly | "The source is correct" is an assumption, not a fact |
| 3 | Never fix data problems by adding COALESCE downstream | Hides the upstream cause; the real issue continues |
| 4 | Never choose a platform because it's trending | Fit to problem, team, and constraints—not hype |
| 5 | Never assume a column is unique, non-null, or immutable without profiling | Dev works fine; production breaks on real data |
| 6 | Never build streaming when batch would suffice | Streaming adds complexity, cost, and operational burden |
| 7 | Never start without defining what "done" looks like for the consumer | Building without a consumer definition produces unused data |
| 8 | Never report aggregates without checking subgroups | Simpson's paradox: trends reverse when segmented |

### First-Step Rules

| Situation | First step | Why |
|-----------|-----------|-----|
| New data source | Profile it: row counts, nulls, cardinality, distributions | *Principle 1*: profile before building |
| "The numbers are wrong" | Define "wrong" precisely: expected vs actual, where is the gap? | Vague problems get vague fixes |
| Pipeline keeps failing | Check for schema drift at source | *Principle 6*: schema will change |
| Choosing a platform | List constraints (volume, freshness, compliance, team, budget) | *Constraint Pattern*: constraints determine architecture |
| Data looks correct but users don't trust it | Check freshness and lineage—trust erodes from invisible staleness | *Visibility Pattern* |
| Inheriting a dataset | Trace one entity end-to-end from source to consumption | *Visibility Pattern*: understand before building |
| Performance issue | Check the grain—is the data at the right level? Misunderstood grain causes wrong joins and exploded row counts | Most common silent error |

---

## The Data Stack as a Dependency Graph

Each layer depends on the one below it. If a lower layer fails, everything above it breaks.

```
  ┌─────────────────────────────────────────────┐
  │  DECISIONS (executives, analysts, ML)       │  ← depends on Serve
  ├─────────────────────────────────────────────┤
  │  SERVE (dashboards, APIs, features)         │  ← depends on Transform
  ├─────────────────────────────────────────────┤
  │  TRANSFORM (Bronze → Silver → Gold)         │  ← depends on Store
  ├─────────────────────────────────────────────┤
  │  STORE (lake, warehouse, database)          │  ← depends on Ingest
  ├─────────────────────────────────────────────┤
  │  INGEST (batch, CDC, streaming)             │  ← depends on Source
  ├─────────────────────────────────────────────┤
  │  SOURCE SYSTEMS (operational databases,     │
  │   APIs, files, third-party feeds)           │
  └─────────────────────────────────────────────┘

  GOVERN runs vertically across all layers:
  Quality gates │ Security │ Lineage │ Contracts │ Retention
```

**Consequence**: A schema change in a source system can silently break reports that executives use for decisions. The chain is: source schema changes → ingestion still "succeeds" → transformation produces wrong joins → aggregates are silently wrong → dashboard shows bad numbers → business makes wrong decision. This is the *GIGO Pattern* operating through the full stack.

---

## Boundary Distinctions

Confusable concepts with their distinguishing property.

| Pair | Distinguishing property | Decision rule |
|------|------------------------|---------------|
| **ETL vs ELT** | Where transformation happens | If you need flexibility and reprocessing → ELT. If you need pre-filtered, clean loads → ETL |
| **Schema-on-read vs schema-on-write** | When structure is enforced | Raw landing → schema-on-read. Curated serving → schema-on-write. Most architectures use both |
| **Batch vs streaming** | Latency requirement | If sub-minute latency is a real business requirement → streaming. Otherwise → batch (*Principle 8*) |
| **OLTP vs OLAP** | Write-heavy vs read-heavy | Operational transactions → OLTP (normalized). Analytics queries → OLAP (denormalized) |
| **Essential vs accidental complexity** | Source of the complexity | Required by the problem → essential (manage it). Created by your solution → accidental (remove it) |
| **Centralized vs federated governance** | Who owns decisions | Small org or strict compliance → centralized. Large org with domain maturity → federated (*Principle 9*) |
| **Data lake vs data warehouse** | Structure enforcement | Flexible schema, raw data, diverse formats → lake. Enforced schema, curated, fast queries → warehouse. Lakehouse combines both |
| **Data quality vs data integrity** | Scope | Quality: does it represent reality? Integrity: are relationships and constraints maintained? Both required |
| **Complicated vs complex data problem** | Predictability | Analyzable with expertise → complicated (plan it). Emergent behavior → complex (experiment) |

---

## Decision Engine for Data Architecture

```
  1. What does the consumer need?
     Define format, freshness, grain, and volume.
     → If unclear, stop. Clarify before building.

  2. What are the constraints?
     Volume, freshness, compliance, team skill, budget, legacy.
     → Constraints determine architecture, not preferences.

  3. What latency is required?
     Hours/daily → batch
     Minutes     → micro-batch
     Seconds     → streaming
     → Default to batch unless proven otherwise (Principle 8).

  4. Where does the data live?
     → Factor in data gravity. Moving is expensive.
     → Platform where most related data already lives.

  5. Build, buy, or adapt?
     Commodity capability → buy (catalog, BI tool, monitoring)
     Core differentiator  → build (domain-specific transforms)
     → Never build what you can buy for commodity needs.

  6. Centralized or federated?
     Small team, strict compliance → centralized governance
     Multiple domains, mature eng  → federated (central standards, domain ownership)
```

---

## Quality Gates: The Medallion Boundary System

Quality is enforced at layer boundaries, not patched at the end.

```
  Source ──▶ [GATE 1] ──▶ Bronze ──▶ [GATE 2] ──▶ Silver ──▶ [GATE 3] ──▶ Gold
              Schema        Raw         Domain        Clean        KPI
              Format      Landing       Rules       Curated      Sanity
              Freshness               Dedupe                   Reconciliation
```

| Gate | Layer | Checks | On failure |
|------|-------|--------|------------|
| **Gate 1** (Ingestion) | Bronze | Schema validation, format, source availability, freshness | Block: stop pipeline, alert |
| **Gate 2** (Transform) | Silver | Standardization, deduplication, domain rules, referential integrity | Block or warn: depends on severity |
| **Gate 3** (Serving) | Gold | KPI sanity checks, cross-source reconciliation, referential completeness | Block: bad data must not reach consumers |

**Quality dimensions** checked across all gates: completeness, accuracy, consistency, validity, timeliness, uniqueness.

**Data SRE**: Define SLOs (freshness target, completeness target, accuracy target) with error budgets. If the error budget is spent, stop feature work and fix reliability.

---

## Ripple Effect Maps

### Schema Drift Cascade

```
  Source system adds/renames a column without notice
    → ingestion still completes (schema-on-read doesn't reject)
      → transformation joins on old column name → nulls or wrong matches
        → aggregates in Gold are silently wrong
          → dashboard shows incorrect KPIs
            → business makes decision on bad data
              → trust in the data platform erodes
                → stakeholders revert to spreadsheets → shadow datasets
```

*Patterns involved*: GIGO, Visibility.
*Break the cycle at*: Ingestion gate. Schema validation + drift detection + automated alerts on source changes.

### Data Quality Erosion Cascade

```
  Quality checks skipped "temporarily" to meet a deadline
    → bad records enter Silver layer
      → downstream transforms build on bad data
        → quality issues compound across joins and aggregations
          → consumers report "the data doesn't look right"
            → trust erodes → manual workarounds → shadow data
              → more data quality issues (nobody trusts the official source)
```

*Patterns involved*: GIGO, Visibility, feedback loop.
*Break the cycle at*: Never skip quality gates. If deadline pressure is real, reduce scope—don't reduce quality.

### Platform Lock-in Cascade

```
  Team selects platform based on hype, not constraints
    → data lands on platform → gravity builds
      → applications and services integrate
        → more data arrives to join with existing data
          → migration cost exceeds budget
            → stuck on platform regardless of fit
              → workarounds for platform limitations → accidental complexity
```

*Patterns involved*: Gravity, Constraint, Visibility.
*Break the cycle at*: Platform selection. Evaluate against constraints (*Decision Engine step 2*), not trends (*Never Rule 4*).

---

## Cross-Cutting Concept Threads

The same idea appears differently at each lifecycle phase.

| Concept | At Ingest | At Transform | At Serve | At Govern |
|---------|-----------|-------------|----------|-----------|
| **Idempotency** | Rerunnable extraction (watermarks, offsets) | MERGE/upsert, partition overwrites | Cache invalidation | Audit trail consistency |
| **Schema** | Schema validation, drift detection | Schema enforcement, evolution | Contract with consumers | Data dictionary, catalog |
| **Quality** | Source profiling, freshness check | Domain rules, deduplication | KPI sanity, reconciliation | SLOs, error budgets, alerting |
| **Security** | Access control on source connections | Data masking, tokenization | RLS, CLS on serving layer | RBAC, ABAC, audit logging |
| **Lineage** | Source registration, extraction metadata | Transformation logic documented | Column-level lineage to reports | End-to-end traceability |

---

## Key References

For detailed coverage of each topic, see the concept docs:

| Topic | Doc |
|-------|-----|
| Problem solving, mental models, biases | [Critical Thinking](critical_thinking.md) |
| Governance, catalog, lineage, policy | [Governance](concepts/core/governance.md) |
| Pipelines, ETL/ELT, orchestration | [Engineering](concepts/core/engineering.md) |
| Quality, integrity, SRE, observability | [Operations](concepts/core/operations.md) |
| Database design, indexing, transactions | [Database](concepts/core/database.md) |
| OLTP/OLAP, NoSQL, store selection | [Data Stores](concepts/core/data_stores.md) |
| Security, privacy, compliance | [Security](concepts/core/security.md) |
| Analytics, statistics, BI | [Analytics](concepts/core/analytics.md) |
| Semantic layer, data products, dbt | [Analytics Engineering](concepts/core/analytics_eng.md) |
| Azure, Fabric, Databricks, streaming | [Platform docs](concepts/platform/) |

> **Strategy docs**: [Approach](../_strategy/01_approach.md) · [Complexity](../_strategy/02_complexity.md) · [Critical Thinking](../_strategy/03_critical_thinking.md) · [Integrated](../_strategy/04_integrated.md)
