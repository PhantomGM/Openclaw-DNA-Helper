# Priority Roadmap

Implementation order for all proposed improvements.

---

## Phase 1: Foundation (Week 1-2)

### Goal: Make faction system usable

**Task 1.1: Faction Decoder Dictionary**
- [ ] Create semantic mapping for all T codes (Type)
- [ ] Create semantic mapping for all G codes (Governance)  
- [ ] Create semantic mapping for all M codes (Military)
- [ ] Create semantic mapping for all P codes (Philosophy)
- [ ] Create semantic mapping for all S codes (Structure)
- [ ] Create semantic mapping for all O codes (Operations)
- [ ] Create semantic mapping for N values (Numeric strength)
- [ ] Create semantic mapping for all L codes (Leadership)
- [ ] Create semantic mapping for all F codes (Finance)
- [ ] Create semantic mapping for all D codes (Diplomacy)
- [ ] Create semantic mapping for all A codes (Agenda)
- [ ] Create semantic mapping for all SC codes (Secrets)
- [ ] Create semantic mapping for all MZ codes (Magic)
- [ ] Create semantic mapping for all X codes (Special)

**Deliverable:** [[Faction Trait Dictionary]]

**Task 1.2: Faction Output Formatter**
- [ ] Add human-readable output format
- [ ] Create faction summary generator
- [ ] Add power level assessment
- [ ] Add territory/resource estimation

**Deliverable:** Updated [[Faction Generator Code]]

---

## Phase 2: Integration (Week 3-4)

### Goal: Connect content types

**Task 2.1: Foreign Key System**
- [ ] Add ORIGIN field to NPC DNA
- [ ] Add FACTION field to NPC DNA
- [ ] Add PATRON/TARGET fields to Quest DNA
- [ ] Add LOCATION field to Quest DNA
- [ ] Add CREATOR/OWNER fields to Item DNA
- [ ] Add RULER/RULING_FACTION to Location DNA

**Deliverable:** [[Cross-Content Reference Specification]]

**Task 2.2: Cross-Generator API**
- [ ] Create generator coordination module
- [ ] Add parameter passing between generators
- [ ] Build consistency validator
- [ ] Create content bundle generator (NPC + Quest + Location together)

**Deliverable:** [[Generator Integration Module]]

---

## Phase 3: Engagement (Week 5-6)

### Goal: Add narrative depth

**Task 3.1: Hook System**
- [ ] Add HOOK/SECRET/NEED to NPC DNA
- [ ] Add INCITING_INCIDENT/TWIST to Quest DNA
- [ ] Add LEGEND/CURSE_WARNING to Item DNA
- [ ] Add RUMOR/DANGER_SIGN to Location DNA

**Deliverable:** [[Hook System Implementation]]

**Task 3.2: Player Connections**
- [ ] Create PC background matcher
- [ ] Add relevance scoring
- [ ] Generate personalized hooks
- [ ] Build party composition analyzer

**Deliverable:** [[Player Connection Generator]]

---

## Phase 4: Conflict (Week 7-8)

### Goal: Add tension and drama

**Task 4.1: Schism System**
- [ ] Add schism tracking to Faction DNA
- [ ] Create split probability calculator
- [ ] Generate splinter faction names
- [ ] Add schism trigger events

**Deliverable:** [[Schism System Implementation]]

**Task 4.2: Dilemma Generator**
- [ ] Build moral choice framework
- [ ] Add consequence calculator
- [ ] Create hidden option logic
- [ ] Generate alignment impact

**Deliverable:** [[Dilemma System Implementation]]

**Task 4.3: Competing Claims**
- [ ] Add contested resource tracker
- [ ] Create claimant relationship web
- [ ] Generate conflict escalation stages
- [ ] Add resolution paths

**Deliverable:** [[Competing Claims System]]

---

## Phase 5: Enhancement (Week 9-10)

### Goal: Advanced features

**Task 5.1: Prism Narrator**
- [ ] Add DNA interpreter layer
- [ ] Create synthesis engine
- [ ] Build narrative generator
- [ ] Add context-aware descriptions

**Deliverable:** [[Prism Narrator Module]]

**Task 5.2: Campaign Weaver**
- [ ] Link multiple quests into arcs
- [ ] Add faction evolution over time
- [ ] Create recurring NPC system
- [ ] Build location development tracker

**Deliverable:** [[Campaign Weaver System]]

---

## Dependencies

```
Phase 1 (Faction Decoder)
    ↓
Phase 2 (Cross-Content) ←── Can work in parallel after Task 1.1
    ↓
Phase 3 (Narrative Hooks) ←── Needs Phase 2
    ↓
Phase 4 (Conflict) ←── Needs Phase 3
    ↓
Phase 5 (Enhancement) ←── Needs all previous
```

---

## Quick Wins (Can do immediately)

These require no dependencies:

1. **Faction Code Glossary** - Document what T1, G01, etc. mean
2. **Example Outputs** - Generate sample NPCs/Quests with explanations
3. **Generator Usage Guide** - Document how to use each generator
4. **DNA Format Reference** - Quick lookup for each code type

---

## Resource Requirements

| Phase | Estimated Hours | Files Changed | New Files |
|-------|----------------|---------------|-----------|
| 1 | 8-12 | 1 | 1 |
| 2 | 12-16 | 5 | 2 |
| 3 | 10-14 | 5 | 2 |
| 4 | 12-16 | 3 | 3 |
| 5 | 16-20 | 1 | 2 |
| **Total** | **58-78 hours** | **15** | **10** |

---

## Decision Points

Before starting Phase 2, decide:
- [ ] Should generators call each other or just reference IDs?
- [ ] Should cross-links be stored in DNA or separate relationship file?
- [ ] How much validation is needed (strict vs permissive)?

Before starting Phase 3, decide:
- [ ] How personalized should hooks be (generic vs PC-specific)?
- [ ] Should secrets be revealed gradually or all at once?

---

## Success Metrics

**Phase 1 Complete When:**
- Faction DNA generates readable description
- 10 sample factions all make sense

**Phase 2 Complete When:**
- Can generate NPC+Quest+Location bundle
- References are consistent across all three

**Phase 3 Complete When:**
- Every content type has at least one hook
- Player connection suggestions are relevant

**Phase 4 Complete When:**
- Faction schisms generate naturally
- Quest dilemmas have meaningful choices

**Phase 5 Complete When:**
- Full campaign arc can be generated
- Content evolves logically over time

---

## Related Files

- [[Vault Analysis Summary]] - Current state
- [[Content Type Comparison]] - Technical details
- [[Improvement Proposals]] - Specifications