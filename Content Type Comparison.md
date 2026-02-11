# Content Type Comparison

Detailed breakdown of each generator's DNA structure and capabilities.

---

## NPC Generator

**File:** [[NPC Generator Code]]

### DNA Structure
```
(LNC/GNE) [score][trait][intensity],... - [trait][score],...
```

### Components
- **LNC (Law-Chaos):** 20 trait pairs (B/C, R/O, L/T, etc.)
- **GNE (Good-Evil):** 19 single traits (H, C, K, G, L, etc.)
- **Scores:** 1-9 for each trait
- **Intensity:** 1-5 modifier
- **Averages:** Overall LNC/GNE positioning

### Strengths
- Dual-axis alignment system (D&D compatible)
- Intensity allows nuance
- Balanced random distribution

### Weaknesses
- Trait meanings not documented
- No personality "archetype" output
- Missing background/hook generation
- No faction/culture connection

---

## Quest Generator

**File:** [[Quest Generator Code]]

### DNA Structure
```
QUEST{v1.0[difficulty/complexity/stakes]}<engagement,obstacle>#{type}
GOAL{category:score,chain}
OBS{category:score,chain}
EVO{stages}
CONFLICT{source}
REWARD{category}
HOOK{type}
FAILURE{consequence}
```

### Components
- **GOAL:** 8 categories (DISC, DELIV, ESCORT, DESTROY, etc.)
- **OBS:** 8 categories (TRAP, ENEMY, TIME, PUZZLE, etc.)
- **EVO:** 4-stage evolution tracking
- **CONFLICT:** 10 sources (INTERNAL, MORAL, ECONOMIC, etc.)
- **REWARD:** 8 types (GOLD, ARTIFACT, LAND, etc.)
- **HOOK:** 8 types (REQUESTED, OVERHEARD, DISCOVERED, etc.)
- **FAILURE:** 8 consequence types

### Strengths
- Most complex generator
- Multi-dimensional quest design
- Engagement vs Obstacle tension tracking
- Evolution stages for campaign arcs
- Failure consequences built-in

### Weaknesses
- Engagement vs Obstacle logic not implemented (just stored)
- No quest-to-quest dependency tracking
- No NPC patron/target integration
- Location-based quest generation missing

---

## Faction Generator

**File:** [[Faction Generator Code]] ⚠️ **PRIORITY FIX**

### DNA Structure
```
T#-G##-M#-P#-S#-O#-N##-L##-F#-D#-A#-SC#-MZ#-X#
```

### Components (Codes Only)
- **T:** 7 traits (T1-T7) - Type?
- **G:** 12 traits (G01-G12) - Government?
- **M:** 8 traits (M1-M8) - Military?
- **P:** 8 traits (P1-P8) - Philosophy?
- **S:** 6 traits (S1-S6) - Structure?
- **O:** 7 traits (O1-O7) - Operations?
- **N:** 6 traits (N74, N78, etc.) - Numeric values?
- **L:** 10 traits (L01-L10) - Leadership?
- **F:** 6 traits (F1-F6) - Finance?
- **D:** 6 traits (D1-D6) - Diplomacy?
- **A:** 9 traits (A1-A9) - Agenda?
- **SC:** 5 traits (SC1-SC5) - Secret?
- **MZ:** 6 traits (MZ1-MZ6) - Magic?
- **X:** 6 traits (X1-X6) - Extra?

### Critical Problem
❌ **No semantic decoder exists** - Codes are meaningless without lookup table
❌ No relationship system (allies/enemies)
❌ No hierarchy tracking
❌ No territory/resources

### Required Fix
See: [[Improvement Proposals#Faction Decoder System]]

---

## Item Generator

**File:** [[Item Generator Code]]

### DNA Structure
```
ITEM{v1.0[power/complexity/rarity]}<AP,MR,RE>#{type}
PHY{material,size,age,condition,weight,form}
MAG{power,element,decay,charge,sentience,art,realm,type}
HIS{origin,creator,age,owner,relic,legend,fallen,status}
LOR{knowledge,function,nature,creator,realm,era,item,significance}
ATTUNE{use,wear,cast,master,spell,vision,power,ritual}
CHAIN{use_chain,magic_chain,attunement_chain}
EVO{power,magic}
```

### Components
- **PHY:** 8 physical properties (MSADCWFT)
- **MAG:** 8 magical properties (PEDCSART)
- **HIS:** 8 historical properties (OCARLFDS)
- **LOR:** 8 lore properties (KFNCREIS)
- **ATTUNE:** 8 attunement requirements (UWCMSVPR)
- **EVO:** Power and magic evolution states

### Strengths
- Detailed multi-aspect system
- Attunement requirements add depth
- Physical/magical/historical separation
- Evolution tracking for cursed/blessed items

### Weaknesses
- No curse vs reverence handling (both map to value ranges)
- No item-to-item relationship (sets, counterparts)
- No creator NPC linkage
- No location found/last seen tracking

---

## Location Generator

**File:** [[Location Generator Code]]

### DNA Structure
```
SETTLEMENT{v1.1[size/pop/importance]}<SP,PI,SI>#{type}
STRUCT{shelter,defense,water,air,food,utilities,power,quality}
POP{size,density,age,class,law,health,culture,mood}
ECON{materials,trade,resources,production,service,growth,innovation,loans}
POL{government,control,resources,freedom,security,corruption,law,ideology}
POI{inn,shop,residence,dungeon,historical,authority,military,crafting}
ARCH{material,roof,construction,decoration}
CULTURE{customs,law,punishment,cuisine}
CHAIN{pop, econ, pol}
PROXI{wilderness,town,city,ruin}
EVO{society,politics,infrastructure}
```

### Components
- **STRUCT:** Infrastructure quality (SDWAFUPQ)
- **POP:** Demographics (SDACLHCM)
- **ECON:** Economy (MTRPSGIL)
- **POL:** Politics (GCRFSCLI)
- **POI:** Points of interest (ISRDHAMC)
- **ARCH:** Architecture (MAT, ROOF, CONST, DECOR)
- **CULTURE:** Social norms
- **PROXI:** Regional relationships
- **EVO:** 3-track evolution

### Strengths
- Most detailed settlement generator
- Proximity system for regional webs
- 3-track evolution (separate society/politics/infrastructure)
- POI generation for immediate adventure hooks

### Weaknesses
- Core vs Reality tension not implemented
- No ruling faction/noble house linkage
- No quest generator integration
- No trade route connections to other locations

---

## Cross-Generator Comparison

| Feature | NPC | Quest | Faction | Item | Location |
|---------|-----|-------|---------|------|----------|
| Evolution | ❌ | ✅ 4-stage | ❌ | ✅ 2-track | ✅ 3-track |
| Chaining | ❌ | ✅ | ❌ | ✅ | ✅ |
| Scoring | 1-9 | 1-9 | 1-9 | 10-99 | 1-9/10-99 |
| Multi-line | ❌ | ✅ | ❌ | ✅ | ✅ |
| Hook System | ❌ | ✅ | ❌ | ❌ | ❌ |
| Cross-links | ❌ | ❌ | ❌ | ❌ | ❌ |

---

## See Also

- [[Vault Analysis Summary]] - Overview
- [[Improvement Proposals]] - Detailed fixes
- [[Priority Roadmap]] - Implementation order