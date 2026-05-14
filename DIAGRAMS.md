# KUru - Diagram Source

Mermaid source for SRS diagrams. Render with a Mermaid-compatible viewer
(VS Code Mermaid Preview, mermaid.live, or Mermaid CLI).

---

## Figure 1 - System Architecture (`fig:system-architecture`)

> Three-tier architecture with AI/data layer. Current POC includes ingestion, RAG, and Program Explorer. MVP adds Pipeline B2 recommendation. Phase 2 adds graph/career/behavioral features.

```mermaid
flowchart TD
    subgraph CLIENT["Client (Vercel)"]
        FE["Next.js 14 App Router<br/>Tailwind CSS / Shadcn UI / TypeScript"]
    end

    subgraph BACKEND["Backend (Railway)"]
        direction TB
        API["FastAPI"]
        RAG["RAG Engine<br/>(current POC + MVP)"]
        EXP["Program Explorer<br/>(current POC + MVP)"]
        REC["MVP Recommendation Engine<br/>Pipeline B2 semantic curriculum matching"]
        ING["Data Ingestion Pipeline"]
        FUTURE_REC["Phase 2 Recommender<br/>Pipeline A + Neo4j B1 + behavioral re-ranking"]

        API --> RAG
        API --> EXP
        API --> REC
        API --> ING
        API -.-> FUTURE_REC
    end

    subgraph DATA["Data Layer"]
        direction LR
        SUP[("Supabase<br/>PostgreSQL / pgvector / Auth")]
        NEO[("Neo4j<br/>Knowledge Graph<br/>(Phase 2)")]
    end

    subgraph AI["AI and OCR Services"]
        direction LR
        GEN["Gemini 2.5 Flash Lite<br/>via OpenRouter<br/>generation"]
        EMB["Local multilingual-E5<br/>intfloat/multilingual-e5-base<br/>embeddings"]
        TYPHOON["Typhoon OCR<br/>low-yield PDF pages"]
        FULL_OCR["Full scanned-PDF OCR<br/>configured Gemini/Typhoon path"]
        EXTRACT["Gemini text mode<br/>structured extraction"]
    end

    subgraph SOURCES["External Data Sources"]
        direction LR
        PDF["MKO.2 curriculum PDFs<br/>KU faculty"]
        TCAS["TCAS admission PDFs/data<br/>KU faculty + mytcas gaps"]
        ONET["O*NET Dataset<br/>(Phase 2)"]
    end

    FE -->|"REST / HTTPS"| API
    FE -->|"optional auth"| SUP

    RAG --> SUP
    RAG --> GEN
    RAG --> EMB

    EXP --> SUP
    EXP --> EMB

    REC --> SUP
    REC --> GEN
    REC --> EMB

    ING -->|"PyMuPDF text first"| PDF
    ING --> TCAS
    ING --> TYPHOON
    ING -.-> FULL_OCR
    ING --> EXTRACT
    ING --> EMB
    ING -->|"chunks + metadata + vectors"| SUP

    FUTURE_REC -.-> SUP
    FUTURE_REC -.-> NEO
    FUTURE_REC -.-> GEN
    ING -.->|"PLO / Skill / Career graph"| NEO
    ING -.-> ONET
```

---

## Figure 2 - Use Case Diagram (`fig:use-case-diagram`)

> Use cases separated by MVP, partial MVP, and Phase 2.

```mermaid
flowchart LR
    GUEST(["Guest"])
    RUSER(["Registered User<br/>(extends Guest)"])

    subgraph MVP_UC["Evaluated MVP"]
        UC01["UC-01<br/>Discover Interests"]
        UC02["UC-02<br/>View Program Recommendations<br/>(Pipeline B2 only)"]
        UC04["UC-04<br/>Query KUru Advisor / RAG"]
        UC08["UC-08<br/>Explain Why Recommended"]
        UC09["UC-09<br/>Search Programs Semantically"]
    end

    subgraph PARTIAL_UC["Partial MVP"]
        UC03["UC-03<br/>Explore PLO Chart<br/>(static only)"]
        UC05["UC-05<br/>TCAS Admission Guide<br/>(program detail + chatbot only)"]
    end

    subgraph PHASE2_UC["Phase 2 / Target System"]
        UC06["UC-06<br/>Explore Career Paths"]
        UC07["UC-07<br/>Behavioral Blending"]
        UC10["UC-10<br/>Pin Programs and Compare"]
        UC11["UC-11<br/>Curriculum Timeline"]
        UC12["UC-12<br/>Portfolio Readiness Check"]
        DASH["Saved Profile Dashboard"]
        TRACK["TCAS Deadline Tracker"]
        RESET["Persistent Reset / Saved Profile Controls"]
    end

    GUEST --> UC01
    GUEST --> UC02
    GUEST --> UC03
    GUEST --> UC04
    GUEST --> UC05
    GUEST --> UC08
    GUEST --> UC09
    GUEST -.-> UC06
    GUEST -.-> UC10
    GUEST -.-> UC11
    GUEST -.-> UC12

    RUSER -->|"inherits"| GUEST
    RUSER -.-> UC07
    RUSER -.-> DASH
    RUSER -.-> TRACK
    RUSER -.-> RESET

    UC01 -->|"RIASEC vector"| UC02
    UC02 -->|"select program"| UC03
    UC02 -->|"tap why?"| UC08
    UC02 -.->|"career link in Phase 2"| UC06
    UC10 -.->|"batch portfolio check in Phase 2"| UC12
```

---

## Figure 3A - MVP Recommendation Pipeline (`fig:mvp-recommendation-pipeline`)

> Evaluated MVP flow. RIASEC becomes curriculum search intent, then matches indexed program/PLO/course chunks. No O*NET, Neo4j B1, or behavioral re-ranking in MVP.

```mermaid
flowchart TD
    START(["Student completes<br/>RIASEC interest discovery"])

    subgraph PROFILE["1. Build Interest Profile"]
        VEC["6D RIASEC vector<br/>R / I / A / S / E / C"]
        TOP["Top dimensions / Holland code<br/>example: Investigative + Realistic"]
        START --> VEC --> TOP
    end

    subgraph INTENT["2. Convert RIASEC to Curriculum Search Intent"]
        THEMES["Map dimensions to interest themes<br/>analysis, systems, design,<br/>experiments, hands-on work"]
        QUERY["Create semantic query<br/>from themes + answers"]
        TOP --> THEMES --> QUERY
    end

    subgraph RETRIEVAL["3. Pipeline B2: Semantic Curriculum Matching"]
        EMB["Local multilingual-E5<br/>embed semantic query"]
        PGV[("Supabase pgvector<br/>program + PLO + course chunks")]
        CHUNKS["Retrieve matching chunks<br/>with program_id + source metadata"]
        QUERY --> EMB --> PGV --> CHUNKS
    end

    subgraph SCORING["4. Program Scoring"]
        GROUP["Group chunks by program"]
        SCORE["Compute curriculum-fit score<br/>top-k similarity average<br/>+ section weighting<br/>+ theme coverage"]
        RANK["Rank programs by score"]
        CHUNKS --> GROUP --> SCORE --> RANK
    end

    subgraph EXPLAIN["5. Evidence-Grounded Explanation"]
        EVIDENCE["Select strongest evidence<br/>PLOs, course descriptions,<br/>projects, labs, admission info if relevant"]
        GEMINI["Gemini 2.5 Flash Lite<br/>via OpenRouter<br/>explain only from evidence"]
        CARDS["Ranked program cards<br/>with explanation"]
        RANK --> EVIDENCE --> GEMINI --> CARDS
    end

    NOTE["MVP evaluation: short-term relevance<br/>advisor/senior-student labels,<br/>curriculum evidence, MRR, NDCG@5,<br/>explanation faithfulness.<br/>No 4-year satisfaction claim."]
    CARDS --> NOTE
```

---

## Figure 3B - Phase 2 Target-System Recommendation Pipeline (`fig:recommendation-pipeline`)

> Five-stage target-system design. This is not the evaluated MVP flow.

```mermaid
flowchart TD
    START(["Student completes<br/>Interest Discovery"])

    subgraph STAGE1["Stage 1 - RIASEC Vector Construction"]
        S1["Likert scores + confidence scalar<br/>+ pairwise adjustments<br/>+ scenario adjustments<br/>+ dealbreaker filter<br/>then L2-normalise"]
    end

    subgraph STAGE2["Stage 2 - Parallel Signal Computation"]
        direction LR

        subgraph PIPEA["Pipeline A - Career-side"]
            A1["RIASEC vector vs.<br/>O*NET occupation profiles"]
            A2["Top 10-15 occupations<br/>internal career space"]
            A3["A-score per KU program<br/>via career-program alignment"]
            A1 --> A2 --> A3
        end

        subgraph PIPEB["Pipeline B - Curriculum-side"]
            B1["B1 - Neo4j PLO match<br/>RIASEC to SkillCluster weights<br/>vs. program PLO profiles"]
            B2["B2 - Semantic course match<br/>interest query vs. course chunks<br/>in Supabase pgvector"]
            BC["B-score = 0.4 * B1 + 0.6 * B2"]
            B1 --> BC
            B2 --> BC
        end
    end

    subgraph STAGE3["Stage 3 - Score Synthesis"]
        S3["Final score = 0.35 * A-score + 0.65 * B-score<br/>Programs ranked; top-N displayed"]
    end

    subgraph STAGE4["Stage 4 - Explanation Generation"]
        S4["Gemini 2.5 Flash Lite explains<br/>from Pipeline A + B evidence"]
    end

    subgraph STAGE5["Stage 5 - Behavioral Re-ranking"]
        S5A{"Enough interaction data?"}
        S5B["Blend alpha * pipeline score<br/>+ (1-alpha) * behavioral fit<br/>alpha decays 1.0 to 0.2"]
        S5C["Pure pipeline score<br/>(alpha = 1.0)"]
        S5A -->|"Yes"| S5B
        S5A -->|"No / Guest"| S5C
    end

    RESULT(["Ranked program cards"])

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

## Figure 4 - RAG Pipeline Sequence (`fig:rag-pipeline`)

> Current MVP/POC query flow for the KUru Advisor chatbot.

```mermaid
sequenceDiagram
    actor Student
    participant FE as Next.js Frontend
    participant API as FastAPI Backend
    participant EMB as Local E5 Embedder
    participant PGV as Supabase pgvector
    participant LLM as Gemini 2.5 Flash Lite

    Student->>FE: Enter question (Thai or English)
    FE->>API: POST /chat {query, history, program_id?}
    API->>EMB: embed(query)
    EMB-->>API: query_vector
    API->>PGV: cosine_search(query_vector, top_k=5, min_similarity=0.35)
    PGV-->>API: chunks with source metadata
    API->>LLM: prompt(query, chunks, history, cite_instruction)
    LLM-->>API: grounded answer + inline citations
    API-->>FE: answer + citations(source_file, section_type, similarity)
    FE-->>Student: answer with provenance chips
    Note over FE,Student: Current chips are not clickable;<br/>chunk preview / PDF-page opening is Phase 2

    alt No relevant chunks found
        PGV-->>API: empty result
        API-->>FE: "This information was not found in the curriculum document."
        FE-->>Student: fallback message
    end
```

---

## Figure 5 - Interest Elicitation Flow (`fig:elicitation-flow`)

> UC-01 adaptive 12-step process.

```mermaid
flowchart TD
    START(["Student opens interest discovery"])

    subgraph LIKERT["Steps 1-6 - Likert Screens"]
        L["One screen per RIASEC dimension<br/>R -> I -> A -> S -> E -> C<br/>4 statements x 5-point scale<br/>Raw score per dimension: max 20<br/>Progress bar: Step 1-6 of 12"]
    end

    subgraph CONFIDENCE["Step 7 - Confidence Check"]
        C["How confident are you<br/>in your answers?"]
        C1["Very confident -> scalar 1.0"]
        C2["Somewhat confident -> scalar 0.75"]
        C3["Not sure -> scalar 0.5"]
        C --> C1 & C2 & C3
        C1 & C2 & C3 --> SCALED["Apply scalar to all 6 dimension scores"]
    end

    DELTA["Compute delta for dimension pairs<br/>abs(score_A - score_B)"]
    AMBIG{"Any pair<br/>with delta < 3?"}

    subgraph PAIRWISE["Step 8 - Adaptive Pairwise"]
        P["Show forced-choice questions<br/>for ambiguous pairs only<br/>max 4-6 pairs<br/>responses adjust scores"]
    end

    subgraph SCENARIOS["Steps 9-11 - Scenario Questions"]
        S["3 scenarios<br/>A-F role options mapped to RIASEC<br/>responses adjust scores"]
    end

    subgraph SUMMARY["Step 12 - Profile Summary + Dealbreaker Filter"]
        SUM["Display top 2 dimensions,<br/>score bars, and Holland code"]
        DB["Optional dealbreaker filter<br/>tap dimensions to exclude"]
        EXCL{"Any dimensions excluded?"}
        ZERO["Zero out excluded dimensions"]
        KEEP["Keep current scores"]
        SUM --> DB --> EXCL
        EXCL -->|"Yes"| ZERO
        EXCL -->|"No"| KEEP
    end

    NORM["L2-normalise 6D vector"]
    OUTPUT(["RIASEC vector output<br/>MVP -> Pipeline B2<br/>Phase 2 -> Pipeline A + Neo4j B1"])

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

## Figure 6 - Domain Model (`fig:domain-model`)

> Core business entities. Phase 2 entities and relationships are explicitly labelled.

```mermaid
classDiagram
    class Student {
        +student_id : uuid
        +language_pref : th|en
        +alpha : float (Phase 2)
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
    class SkillCluster {
        <<Phase 2>>
        +cluster_id : string
        +name : string
        +riasec_weights : float[]
    }
    class Career {
        <<Phase 2>>
        +onet_code : string
        +title_en : string
        +title_th : string
        +riasec_profile : float[]
    }
    class InteractionLog {
        <<Phase 2>>
        +interaction_type : string
        +weight : float
        +timestamp : timestamp
    }
    class PortfolioCriteria {
        <<Phase 2>>
        +round : int
        +required_items : json
        +preferred_items : json
    }

    Student "1" --> "1" RIASECProfile : has
    Program "*" --> "1" Faculty : belongs to
    Program "1" --> "*" Course : contains
    Program "1" --> "*" TCASRecord : has admission via
    Faculty "1" --> "*" PLO : defines

    Student "1" --> "*" InteractionLog : generates (Phase 2)
    Student "*" --> "*" Program : saves (Phase 2)
    InteractionLog "*" --> "1" Program : about (Phase 2)
    Faculty "1" --> "*" PortfolioCriteria : publishes (Phase 2)
    PLO "*" --> "*" SkillCluster : develops (Phase 2)
    Career "*" --> "*" SkillCluster : requires (Phase 2)
    RIASECProfile ..> Career : Pipeline A (Phase 2)
    RIASECProfile ..> SkillCluster : Pipeline B1 (Phase 2)
```
