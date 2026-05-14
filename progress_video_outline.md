# KUru Progress Video Outline

Target length: about 5 minutes.

## 0:00-1:00 Proposal Feedback and Response

Main feedback point 1: The committee's main concern was recommendation evaluation.
Response: We agreed that the strongest real outcome, whether a student actually likes or benefits from a program, would take years to know. Because that is not feasible in an MVP, we cut career-path validation from the evaluated scope and changed the recommendation evaluation to short-term proxies: expert/senior-student relevance labels, curriculum evidence, MRR/NDCG@5, explanation faithfulness, and user feedback.

Main feedback point 2: The original proposal scope was too broad for an MVP.
Response: We narrowed the evaluated MVP to RIASEC interest discovery, simplified program recommendation, KUru Advisor RAG, recommendation explanation, semantic program search, static PLO chart, and partial TCAS support. Career explorer, behavioural blending, comparison, timeline, portfolio checker, O*NET, and Neo4j B1 are moved to Phase 2.

Main feedback point 3: The recommendation architecture was too complex for the current project stage.
Response: We now separate MVP from target architecture. MVP recommendation uses Pipeline B2 semantic curriculum matching over Supabase/pgvector. The full hybrid design with O*NET, Neo4j, and behavioural re-ranking remains the future architecture.

Main feedback point 4: Evaluation needed to match the actual AI implementation.
Response: The May 2026 POC uses custom LLM-as-judge regression tests with MLflow because TCAS, fees, aliases, bilingual answers, and missing-data honesty are KU-specific. Final MVP will add RAGAS faithfulness and answer relevancy as standardized metrics.

## 1:00-5:00 Progress Demo

1. Show the running web app or frontend.
   - Start with the main program/search/chat entry point.
   - Avoid spending time on login or sign-up.

2. Demo core AI feature: KUru Advisor RAG.
   - Ask a TCAS or fee question for a known program.
   - Point out retrieved/source-cited answer.
   - Ask a missing-data question and show that the system refuses to invent an answer.

3. Demo semantic program search or program explorer.
   - Use a natural-language Thai or English query.
   - Show that matching programs come from indexed curriculum/program content.

4. Briefly show implementation evidence.
   - Backend RAG module or API route.
   - Ingestion/indexing output or Supabase/pgvector table.
   - MLflow run `v8_structured_tcas_fees` or evaluation result summary.

5. Close with current status.
   - Completed POC: data ingestion, RAG baseline, TCAS/fee grounding, cleaned corpus, Program Explorer, frontend/backend integration, MLflow evaluation.
   - In progress: RIASEC interest discovery, simplified Pipeline B2 recommendation and explanation, final MVP recommendation evaluation using relevance labels and ranking metrics, RAGAS supplementary metrics, usability testing.
   - Adjustment: career-path validation is cut from MVP because long-term program satisfaction cannot be measured during the project.
