# extract-official-guide - Architecture

> Back to [README](../README.md)

## Overview

```mermaid
graph TB
    V[Vendor sites<br>Anthropic / OpenAI / Google] --> A[Phase A: Download]
    A --> O[(offical_guide/ archive)]
    O --> B[Phase B: Extract base]
    B --> BF["_base / _base_unlisted / _vendor_*"]
    O --> C[Phase C: Extract model files]
    BF -->|dedup reference| C
    C --> MF["&lt;key&gt;.md"]
    BF --> D[Phase D: Verify and report]
    MF --> D
    D --> I[officialGuideSection layered injection]
```

## Module: Phase A Download

Resolves keys to official URLs, fetches the full markdown, and reconciles it with the existing archive.

```mermaid
graph TB
    subgraph PhaseA[Phase A: Download]
        R[A1 Resolve<br>key → URL] --> F[A2 Fetch full text]
        F --> Q{Archive exists?}
        Q -->|No| W[Write archive<br>first line: Retrieved date and URL]
        Q -->|Yes| DF[A3 diff]
        DF -->|No change or date line only| S[Skip write]
        DF -->|Content differs| ASK[Summarize changed sections and ask]
        ASK -->|Approved| W
        ASK -->|Declined| S
    end
    REG[Source registry] --> R
    R -->|Unregistered| U[Ask user for URL]
    F -->|Live URL replaced| WB[Wayback snapshot]
    WB --> Q
    F -->|Fetch failed| FAIL[Report failure<br>keep old archive]
```

## Module: Phase B Extract Base

Produces the always-injected structural rules, the unlisted-model baseline, and the vendor cross-model layer from distinct sources.

```mermaid
graph TB
    subgraph PhaseB[Phase B: Extract base]
        B1[B1 Structural rules] --> BASE["_base.md<br>Instruction conflicts / Tools"]
        B2[B2 Cross-vendor commons] --> UNL["_base_unlisted.md<br>Acting / Long inputs / Verification / Scope"]
        B2b[B2b Vendor overview] --> VEN["_vendor_&lt;vendor&gt;.md"]
        BASE --> B3{B3 Overlap check<br>same rule in ≥ 3 model files?}
        B3 -->|Yes| UNL
        B3 -->|No| B4[B4 Land<br>diff and ask if exists]
        UNL --> B4
        VEN --> B4
    end
    LOCAL[Harness local contracts] --> B1
    ARCH[(offical_guide/ archive)] --> B2
    ARCH --> B2b
    PREV[Previous model files] --> B3
```

## Module: Phase C Extract Model Files

Reduces each archived guide to the model-specific anti-default corrections, one key at a time.

```mermaid
graph TB
    subgraph PhaseC[Phase C: Extract model files]
        C1["C1 Inventory ## sections"] --> C2[C2 Section selection<br>always-exclude list]
        C2 --> C3[C3 Item filtering<br>deletion-reason table]
        C3 --> C4[C4 Dedup<br>against _base.md only]
        C4 --> C5[C5 Crosscheck shared layer]
        C5 --> C6[C6 Land<br>diff and ask if exists]
    end
    ARCH[(offical_guide/ archive)] --> C1
    BASE["_base.md"] --> C4
    SP["system_prompt.md<br>Behavioral Constraints"] --> C5
    GUIDE["configs/prompts/guide/*.md"] --> C5
    TOOLS[Tool descriptions] --> C5
    C6 --> OUT["&lt;DEST&gt;/&lt;key&gt;.md"]
```

## Module: Phase D Verify

Runs budget, vocabulary, ordering, overlap, build, and injection-branch checks after writing.

```mermaid
graph LR
    subgraph PhaseD[Phase D: Verify]
        V1[Lines and sections] --> V2[Excluded-section leftovers]
        V2 --> V3[Empty sections]
        V3 --> V4[_base overlap count]
        V4 --> V5[Heading vocabulary]
        V5 --> V6[Heading order and Other count]
        V6 --> V7[Ellipsis and full-width chars]
        V7 --> V8[go build and grep -a embed]
        V8 --> V9[Injection branch test]
    end
    V9 --> RPT[Report]
```

## Module: Layered Injection

`officialGuideSection()` picks one file per layer and stacks them.

```mermaid
graph TB
    M[Model name] --> N[claude prefix<br>normalize . to -]
    N --> K{Model file match?<br>longest key wins}
    N --> VQ{Contains vendor name?}
    BASE["_base.md"] --> P[Injected content]
    K -->|Yes| MF["&lt;key&gt;.md"] --> P
    VQ -->|Yes| VF["_vendor_&lt;vendor&gt;.md"] --> P
    K -->|No| BOTH{Neither matched?}
    VQ -->|No| BOTH
    BOTH -->|Yes| UNL["_base_unlisted.md"] --> P
```

## Data Flow

```mermaid
sequenceDiagram
    participant U as User
    participant S as Skill
    participant W as Vendor site
    participant A as offical_guide/
    participant D as DEST
    U->>S: /extract-official-guide [key...]
    S->>W: Fetch full .md
    W-->>S: markdown
    S->>A: diff existing archive
    A-->>S: Change summary
    S->>U: Ask to update archive
    U-->>S: Approve
    S->>A: Write archive
    S->>A: Read full text
    S->>D: Read _base.md and existing outputs
    S->>U: Show output diff and ask
    U-->>S: Approve
    S->>D: Write base / vendor / model files
    S->>D: Run verification
    S-->>U: Download, output, deletion evidence, verification report
```

## State Machine

The overwrite gate shared by archives and outputs.

```mermaid
stateDiagram-v2
    [*] --> Candidate
    Candidate --> Write: Target missing
    Candidate --> Compare: Target exists
    Compare --> Skip: No change or date line only
    Compare --> Pending: Content differs
    Pending --> Write: User approves
    Pending --> ReportOnly: User declines
    Write --> Verify
    Skip --> [*]
    ReportOnly --> [*]
    Verify --> [*]
```
