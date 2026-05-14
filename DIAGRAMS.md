# KUru — Diagram Source

Mermaid source for all figures in the SRS. Render with any Mermaid-compatible viewer
(VS Code Mermaid Preview, mermaid.live, etc.).

---

## Figure 1 — System Architecture (`fig:system-architecture`)

> Three-tier architecture with AI layer. Corresponds to §4.1.

```mermaid
flowchart TD
    subgraph CLIENT["Client (Vercel)"]
        FE["Next.js 14 App Router<br/>Tailwind CSS · Shadcn/UI · TypeScript"]
    end

    subgraph BACKEND["Backend (Railway)"]
        direction TB
        API["FastAPI"]
        RAG["RAG Engine"]
        REC["MVP Recommendation Engine<br/>Pipeline B2 semantic curriculum matching"]
        FUTURE_REC["Phase 2 Recommender<br/>Pipeline A + Neo4j B1 + behavioural re-ranking"]
        ING["Data Ingestion Pipeline"]
        API --> RAG
        API --> REC
        API -.-> FUTURE_REC
        API --> ING
    end

    subgraph DATA["Data Layer"]
        direction LR
        SUP[("Supabase<br/>PostgreSQL · pgvector · Auth")]
        NEO[("Neo4j<br/>Knowledge Graph<br/>(Phase 2)")]
    end

    subgraph AI_LAYER["AI — Google"]
        direction LR
        GEN["Gemini 2.5 Flash<br/>Generation"]
        EMB["Gemini text-embedding-001<br/>Embeddings"]
    end

    subgraph SOURCES["External Data Sources"]
        direction LR
        PDF["มคอ.2 PDFs<br/>KU Faculty"]
        ONET["O*NET Dataset<br/>US Dept. of Labor<br/>(Phase 2)"]
        TCAS["TCAS Admission Data<br/>KU Faculty"]
    end

    FE -->|"REST / HTTPS"| API
    FE -->|"Auth"| SUP

    RAG --> SUP
    RAG --> GEN
    RAG --> EMB

    REC --> SUP
    REC --> GEN
    FUTURE_REC -.-> SUP
    FUTURE_REC -.-> NEO
    FUTURE_REC -.-> GEN

    ING --> PDF
    ING -.-> ONET
    ING --> TCAS
    ING -->|"embeddings + structured data"| SUP
    ING -.->|"PLO · Skill · Career graph"| NEO
```

---

## Figure 2 — Use Case Diagram (`fig:use-case-diagram`)

> Actors: Guest and Registered User. Corresponds to §3.3.

```mermaid
flowchart LR
    GUEST(["👤 Guest"])
    RUSER(["👤 Registered User<br/>(extends Guest)"])

    subgraph GUEST_UC["Available to all users"]
        UC01["UC-01<br/>Discover Interests"]
        UC02["UC-02<br/>View Program Recommendations"]
        UC03["UC-03<br/>Explore PLO Spider Chart"]
        UC04["UC-04<br/>Query Curriculum Chatbot"]
        UC05["UC-05<br/>View TCAS Admission Guide"]
        UC06["UC-06<br/>Explore Career Paths<br/>(Phase 2)"]
        UC08["UC-08<br/>Explain Why Recommended"]
        UC09["UC-09<br/>Search Programs Semantically"]
        UC10["UC-10<br/>Pin Programs & Compare<br/>(Phase 2)"]
        UC11["UC-11<br/>View Curriculum Timeline<br/>(Phase 2)"]
        UC12["UC-12<br/>Portfolio Readiness Check<br/>(Phase 2)"]
    end

    subgraph REG_UC["Registered users only"]
        UC07["UC-07<br/>Refine Profile via Implicit Signals<br/>(Phase 2)"]
        DASH["Save Profile & Bookmark Programs<br/>(Phase 2)"]
        TRACK["Track TCAS Deadlines<br/>(Phase 2)"]
        RESET["Reset Interest Profile"]
    end

    GUEST --> UC01
    GUEST --> UC02
    GUEST --> UC03
    GUEST --> UC04
    GUEST --> UC05
    GUEST --> UC06
    GUEST --> UC08
    GUEST --> UC09
    GUEST --> UC10
    GUEST --> UC11
    GUEST --> UC12

    RUSER -->|inherits| GUEST
    RUSER --> UC07
    RUSER --> DASH
    RUSER --> TRACK
    RUSER --> RESET

    UC01 -->|"produces RIASEC vector"| UC02
    UC02 -->|"select program"| UC03
    UC02 -.->|"Phase 2 career link"| UC06
    UC02 -->|"tap why?"| UC08
    UC10 -.->|"Phase 2 batch check"| UC12
```

---

## Figure 3A — MVP Recommendation Pipeline (`fig:mvp-recommendation-pipeline`)

> Evaluated MVP flow. RIASEC is converted into curriculum search intent, then matched against indexed program/PLO/course chunks. This should replace the old recommendation diagram as the primary diagram in §4.2.2.

```mermaid
flowchart TD
    START(["Student completes<br/>RIASEC interest discovery"])

    subgraph PROFILE["1. Build Interest Profile"]
        VEC["6D RIASEC vector<br/>R, I, A, S, E, C"]
        TOP["Top dimensions / Holland code<br/>e.g., Investigative + Realistic"]
        START --> VEC --> TOP
    end

    subgraph INTENT["2. Convert RIASEC to Curriculum Search Intent"]
        THEMES["Map top dimensions to interest themes<br/>analysis, problem solving,<br/>systems, technical design,<br/>experiments, hands-on work"]
        QUERY["Create semantic query<br/>from themes + student answers"]
        TOP --> THEMES --> QUERY
    end

    subgraph RETRIEVAL["3. Pipeline B2: Semantic Curriculum Matching"]
        EMB["Gemini text-embedding-001<br/>embed semantic query"]
        PGV[("Supabase pgvector<br/>program + PLO + course chunks")]
        CHUNKS["Retrieve top matching chunks<br/>with program_id and source metadata"]
        QUERY --> EMB --> PGV --> CHUNKS
    end

    subgraph SCORING["4. Program Scoring"]
        GROUP["Group retrieved chunks<br/>by program"]
        SCORE["Compute curriculum-fit score<br/>top-k similarity average<br/>+ section/source weighting<br/>+ coverage of interest themes"]
        RANK["Rank programs by score"]
        CHUNKS --> GROUP --> SCORE --> RANK
    end

    subgraph EXPLAIN["5. Evidence-Grounded Explanation"]
        EVIDENCE["Select strongest curriculum evidence<br/>PLOs, course descriptions,<br/>projects, labs, admission info if relevant"]
        GEMINI["Gemini 2.5 Flash<br/>generate explanation only from evidence"]
        CARDS["Ranked program cards<br/>with why-this-matches explanation"]
        RANK --> EVIDENCE --> GEMINI --> CARDS
    end

    NOTE["MVP evaluation checks short-term relevance:<br/>advisor/senior-student labels, curriculum evidence,<br/>MRR, NDCG@5, explanation faithfulness.<br/>It does not claim 4-year satisfaction."]
    CARDS --> NOTE
```

---

## Figure 3B — Phase 2 Target-System Recommendation Pipeline (`fig:recommendation-pipeline`)

> Five-stage pipeline with two parallel signals. This is the Phase~2 target-system design, not the evaluated MVP flow.

```mermaid
flowchart TD
    START(["Student completes<br/>Interest Discovery"])

    subgraph STAGE1["Stage 1 — RIASEC Vector Construction"]
        S1["Apply topic-to-RIASEC weights<br/>+ pairwise adjustments<br/>+ dealbreaker filter<br/>+ confidence scaling<br/>→ L2-normalise to unit vector"]
    end

    subgraph STAGE2["Stage 2 — Parallel Signal Computation"]
        direction LR

        subgraph PIPEA["Pipeline A — Career-side"]
            A1["Cosine similarity:<br/>RIASEC vector vs.<br/>O*NET occupation profiles<br/>(Supabase pgvector)"]
            A2["Top 10–15 occupations<br/>form internal career space<br/>(top 7 shown in Career Explorer)"]
            A3["A-score per KU program<br/>via career–program alignment"]
            A1 --> A2 --> A3
        end

        subgraph PIPEB["Pipeline B — Curriculum-side"]
            B1["B1 — Neo4j PLO match<br/>RIASEC → SkillCluster weights<br/>vs. program PLO profiles<br/>(cosine similarity)"]
            B2["B2 — Semantic course match<br/>Interest query vs.<br/>course description embeddings<br/>(Supabase pgvector)"]
            BC["B-score = 0.4 × B1 + 0.6 × B2"]
            B1 --> BC
            B2 --> BC
        end
    end

    subgraph STAGE3["Stage 3 — Score Synthesis"]
        S3["Final score = 0.35 × A-score + 0.65 × B-score<br/>Programs ranked; top-N selected for display"]
    end

    subgraph STAGE4["Stage 4 — Explanation Generation"]
        S4["Gemini 2.5 Flash generates<br/>plain-language explanation per program<br/>grounded in Pipeline A + B evidence<br/>(Thai or English)"]
    end

    subgraph STAGE5["Stage 5 — Behavioural Re-ranking (Registered Users)"]
        S5A{"Sufficient<br/>interaction data?"}
        S5B["Blend: α × pipeline score<br/>+ (1−α) × behavioural fit<br/>α decays 1.0 → 0.2<br/>as interactions accumulate"]
        S5C["Pure pipeline score<br/>(α = 1.0)"]
        S5A -->|"Yes"| S5B
        S5A -->|"No / Guest"| S5C
    end

    RESULT(["Ranked program cards<br/>displayed to student"])

    START --> STAGE1
    STAGE1 --> STAGE2
    PIPEA --> STAGE3
    PIPEB --> STAGE3
    STAGE3 --> STAGE4
    STAGE4 --> STAGE5
    S5B --> RESULT
    S5C --> RESULT
```

---

## Figure 4 — RAG Pipeline Sequence (`fig:rag-pipeline`)

> Query flow for the Curriculum Chatbot. Corresponds to §4.2.1.
> (No current placeholder in the LaTeX — consider adding one to §4.2.1.)

```mermaid
sequenceDiagram
    actor Student
    participant FE as Next.js Frontend
    participant API as FastAPI Backend
    participant EMB as Gemini Embeddings
    participant PGV as Supabase pgvector
    participant LLM as Gemini 2.5 Flash

    Student->>FE: Enter question (Thai or English)
    FE->>API: POST /chat {query, history, program_ids?}
    API->>EMB: embed(query)
    EMB-->>API: query_vector
    API->>PGV: cosine_search(query_vector, top_k=5)
    PGV-->>API: [chunk_1 … chunk_k] with source metadata
    API->>LLM: prompt(query, chunks, history, cite_instruction)
    LLM-->>API: grounded answer + inline citations
    API-->>FE: {answer, citations: [{doc, section, page}]}
    FE-->>Student: Answer with มคอ.2 source badges

    alt No relevant chunks found
        PGV-->>API: empty result
        API-->>FE: "ไม่พบข้อมูลนี้ในเอกสารหลักสูตร"
        FE-->>Student: Fallback message
    end
```

---

## Figure 5 — Interest Elicitation Flow (`fig:elicitation-flow`)

> UC-01 adaptive 11-step process. Export as `assets/diagrams/elicitation-flow.png`.
> Corresponds to §4.2.3.

```mermaid
flowchart TD
    START(["Student opens ค้นหาความสนใจ"])

    subgraph LIKERT["Steps 1–6 — Likert Screens"]
        L["One screen per RIASEC dimension<br/>(R → I → A → S → E → C)<br/>4 statements × 5-point scale<br/>Raw score per dimension: max 20<br/>Progress bar: Step 1–6 of 11"]
    end

    subgraph CONFIDENCE["Step 7 — Confidence Check"]
        C["คุณรู้สึกมั่นใจแค่ไหนกับคำตอบที่เพิ่งเลือก?"]
        C1["มั่นใจมาก → scalar 1.0"]
        C2["ค่อนข้างมั่นใจ → scalar 0.75"]
        C3["ไม่แน่ใจเลย → scalar 0.5"]
        C --> C1 & C2 & C3
        C1 & C2 & C3 --> SCALED["Apply scalar to all 6 dimension scores"]
    end

    DELTA["Compute delta for each dimension pair<br/>|score_A − score_B| for all 15 pairs"]

    AMBIG{"Any pair<br/>with delta < 3?"}

    subgraph PAIRWISE["Step 8 — Adaptive Pairwise (conditional)"]
        P["Show forced-choice questions<br/>for ambiguous pairs only<br/>(max 4–6 pairs)<br/>Each pair offers 'ชอบเท่ากัน' as middle choice<br/>Responses adjust dimension scores"]
    end

    subgraph SCENARIOS["Steps 9–11 — Scenario Questions"]
        S["3 scenarios presented sequentially<br/>Each offers A–F role options<br/>mapped to R/I/A/S/E/C<br/>Student selects one preferred role<br/>Responses adjust dimension scores proportionally"]
    end

    subgraph SUMMARY["Step 11 — Profile Summary + Dealbreaker Filter"]
        SUM["Display:<br/>• Top 2 dominant dimensions<br/>• Bar chart of all 6 scores (out of 20)<br/>• Holland Code label"]
        DB["มีสาขาไหนที่คุณไม่อยากเรียนเลย?<br/>6 dimension chips — tap to exclude<br/>'ไม่มี ข้ามได้เลย' prominently shown"]
        EXCL{"Any dimensions<br/>excluded?"}
        ZERO["Zero out excluded dimensions"]
        KEEP["Continue with current scores"]
        SUM --> DB --> EXCL
        EXCL -->|"Yes"| ZERO
        EXCL -->|"No"| KEEP
    end

    NORM["L2-normalise 6-dimensional vector"]
    OUTPUT(["RIASEC vector output<br/>MVP → Pipeline B2 semantic curriculum matching<br/>Phase 2 → Pipeline A + Neo4j B1"])

    START --> LIKERT
    LIKERT --> CONFIDENCE
    CONFIDENCE --> DELTA
    DELTA --> AMBIG
    AMBIG -->|"Yes"| PAIRWISE
    AMBIG -->|"No"| SCENARIOS
    PAIRWISE --> SCENARIOS
    SCENARIOS --> SUMMARY
    ZERO --> NORM
    KEEP --> NORM
    NORM --> OUTPUT
```

---

## Figure 6 — Domain Model (`fig:domain-model`)

> Core business entities and relationships. Corresponds to §4.1.

```mermaid
classDiagram
    class Student {
        +student_id : uuid
        +language_pref : th|en
        +alpha : float
    }
    class RIASECProfile {
        +R I A S E C : float
        +confidence_scalar : float
        +created_at : timestamp
    }
    class Program {
        +program_id : uuid
        +name_th : string
        +name_en : string
        +degree_level : string
        +mytcas_code : string
    }
    class Faculty {
        +faculty_id : uuid
        +name_th : string
        +name_en : string
    }
    class PLO {
        +plo_id : string
        +description_th : string
        +description_en : string
        +domain : string
    }
    class SkillCluster {
        +cluster_id : string
        +name : string
        +riasec_weights : float[]
    }
    class Career {
        +onet_code : string
        +title_en : string
        +title_th : string
        +riasec_profile : float[]
    }
    class Course {
        +course_code : string
        +title_th : string
        +credits : int
        +year : int
    }
    class TCASRecord {
        +round : int
        +project_name : string
        +quota : int
        +gpax_min : float
        +deadline : date
    }
    class InteractionLog {
        +interaction_type : string
        +weight : float
        +timestamp : timestamp
    }
    class PortfolioCriteria {
        +round : int
        +required_items : json
        +preferred_items : json
    }

    Student "1" --> "1" RIASECProfile : has
    Student "1" --> "*" InteractionLog : generates
    Student "*" --> "*" Program : saves
    InteractionLog "*" --> "1" Program : about

    Program "*" --> "1" Faculty : belongs to
    Program "1" --> "*" Course : contains
    Program "1" --> "*" TCASRecord : has admission via

    Faculty "1" --> "*" PLO : defines
    Faculty "1" --> "*" PortfolioCriteria : publishes

    PLO "*" --> "*" SkillCluster : develops
    Career "*" --> "*" SkillCluster : requires

    RIASECProfile ..> Career : matched against (Pipeline A)
    RIASECProfile ..> SkillCluster : converted to weights (Pipeline B1)
```
