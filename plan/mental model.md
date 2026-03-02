┌─────────────────────────────────────────────┐
│            APPLICATIONS (13)                │  ← What you build
├─────────────────────────────────────────────┤
│         DESIGN PATTERNS (12)                │  ← How you build
├──────────┬──────────┬──────────┬────────────┤
│ PLANNING │REASONING │ MEMORY   │ TOOLS      │  ← Core capabilities
│   (3)    │   (2)    │  (4)     │  (5)       │     (MODULAR)
├──────────┴──────────┴──────────┴────────────┤
│      AGENT CORE LOOP & CONTROL FLOW (1,7)   │  ← Foundation
├─────────────────────────────────────────────┤
│         LLM FOUNDATIONS (11)                │  ← Engine
├─────────────────────────────────────────────┤
│         INFRASTRUCTURE (14)                 │  ← Platform
└─────────────────────────────────────────────┘
         │                           │
    ┌────┴────┐                ┌─────┴─────┐
    │ SAFETY  │                │EVALUATION │
    │  (10)   │                │   (9)     │
    │         │                │           │
    │ CROSS-  │                │ CROSS-    │
    │CUTTING  │                │ CUTTING   │
    └─────────┘                └───────────┘



┌──────────────────────┬────────────┬──────────────────────────────────┐
│ CONCEPT AREA         │ MODULARITY │ WHY                              │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 1. Agent Core Loop   │ ⬛⬛⬛⬜⬜   │ Central; everything plugs into   │
│                      │ LOW        │ this. Hard to swap entirely.     │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 2. Reasoning         │ ⬛⬛⬛⬜⬜   │ Deeply tied to LLM + prompting.  │
│                      │ MEDIUM-LOW │ Can swap strategies but not      │
│                      │            │ decouple from core loop.         │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 3. Planning          │ ⬛⬛⬛⬛⬜   │ Strategies are interchangeable.  │
│                      │ MEDIUM     │ But planning ↔ reasoning are     │
│                      │            │ tightly linked.                  │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 4. Memory            │ ⬛⬛⬛⬛⬜   │ Storage backends are modular.    │
│                      │ HIGH       │ Memory types are composable.     │
│                      │            │ RAG is its own module.           │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 5. Tool Use          │ ⬛⬛⬛⬛⬛   │ MOST modular. Tools are plug &   │
│                      │ VERY HIGH  │ play by design. Add/remove       │
│                      │            │ freely.                          │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 6. Multi-Agent       │ ⬛⬛⬛⬛⬜   │ Agents are modular units.        │
│                      │ HIGH       │ Communication patterns are       │
│                      │            │ swappable. But orchestration     │
│                      │            │ creates coupling.                │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 7. Control Flow      │ ⬛⬛⬛⬜⬜   │ Deeply tied to agent core +      │
│                      │ MEDIUM-LOW │ state management. Patterns are   │
│                      │            │ swappable though.                │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 8. Grounding         │ ⬛⬛⬛⬛⬜   │ Knowledge sources are modular.   │
│                      │ HIGH       │ Can mix and match retrieval      │
│                      │            │ strategies.                      │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 9. Evaluation        │ ⬛⬛⬛⬛⬛   │ Fully modular. Observability is  │
│                      │ VERY HIGH  │ a separate layer. Tools are      │
│                      │            │ interchangeable.                 │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 10. Safety           │ ⬛⬛⬜⬜⬜   │ CROSS-CUTTING concern. Touches   │
│                      │ LOW        │ every other module. Cannot be    │
│                      │            │ isolated.                        │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 11. LLM Foundations  │ ⬛⬛⬛⬛⬜   │ LLM providers are swappable.     │
│                      │ HIGH       │ But capabilities vary, so not    │
│                      │            │ perfectly interchangeable.       │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 12. Design Patterns  │ ⬛⬛⬛⬛⬛   │ Patterns are templates/recipes.  │
│                      │ VERY HIGH  │ Mix and match freely.            │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 13. Applications     │ ⬛⬛⬛⬛⬛   │ Fully modular. Domain-specific   │
│                      │ VERY HIGH  │ implementations are independent. │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 14. Infrastructure   │ ⬛⬛⬛⬛⬜   │ Deployment is modular. But       │
│                      │ HIGH       │ choices affect performance       │
│                      │            │ constraints.                     │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 15. Advanced Topics  │ ⬛⬛⬛⬛⬛   │ Research areas. Mostly           │
│                      │ VERY HIGH  │ independent explorations.        │
├──────────────────────┼────────────┼──────────────────────────────────┤
│ 16. Ecosystem        │ ⬛⬛⬛⬛⬛   │ Tools are fully interchangeable  │
│                      │ VERY HIGH  │ by design.                       │
└──────────────────────┴────────────┴──────────────────────────────────┘
