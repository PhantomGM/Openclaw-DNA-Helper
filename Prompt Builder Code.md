# Prompt Builder Code

This file contains the logic for translating raw DNA strings into rich, instructional prompts for the LLM. It uses the RAFT framework (Role, Audience, Format, Task).

```python
# prompt_builder.py
# --- DNA DECODER & PROMPT CONSTRUCTOR ---

import re

# --- TRAIT MAPPING (LNC Paired) ---
LNC_PAIRS_MAP = [
    ('B', 'Brave', 'C', 'Cowardly'),
    ('R', 'Reserved', 'O', 'Outspoken'),
    ('L', 'Reckless', 'T', 'Cautious'),
    ('F', 'Confident', 'I', 'Insecure'),
    ('S', 'Stoic', 'X', 'Expressive'),
    ('P', 'Patient', 'M', 'Impatient'),
    ('D', 'Methodical', 'U', 'Impulsive'),
    ('G', 'Organized', 'H', 'Chaotic'),
    ('Y', 'Suspicious', 'W', 'Trusting'),
    ('E', 'Serious', 'A', 'Playful'),
    ('N', 'Introverted', 'V', 'Extroverted'),
    ('K', 'Competitive', 'Q', 'Harmonious'),
    ('Z', 'Tactful', 'B', 'Blunt'),
    ('O', 'Optimistic', 'P', 'Pessimistic'),
    ('C', 'Calm', 'H', 'Hot-headed'),
    ('R', 'Perfectionist', 'L', 'Laid-Back'),
    ('A', 'Authoritative', 'S', 'Submissive'),
    ('D', 'Driven', 'A', 'Apathetic'),
    ('A', 'Adventurous', 'H', 'Hesitant'),
    ('I', 'Diplomatic', 'C', 'Confrontational')
]

# --- TRAIT MAPPING (GNE Unpaired) ---
GNE_TRAITS_MAP = [
    ('H', 'Honest'), ('C', 'Compassionate'), ('K', 'Kind'), ('G', 'Generous'),
    ('L', 'Loyal'), ('J', 'Just'), ('M', 'Merciful'), ('F', 'Forgiving'),
    ('E', 'Empathetic'), ('B', 'Benevolent'), ('U', 'Humble'), ('S', 'Selfless'),
    ('I', 'Integrity'), ('R', 'Responsible'), ('T', 'Tolerant'), ('A', 'Fair'),
    ('D', 'Devoted'), ('V', 'Charitable'), ('Y', 'Accountable'), ('X', 'Virtuous')
]

def decode_alignment(score, kind="LNC"):
    score = int(score)
    if kind == "LNC":
        if score >= 7: return "Lawful"
        elif score <= 3: return "Chaotic"
        else: return "Neutral"
    elif kind == "GNE":
        if score >= 7: return "Good"
        elif score <= 3: return "Evil"
        else: return "Neutral"
    return "Neutral"

def decode_gne_strength(score):
    score = int(score)
    if score >= 7: return "Strongly"
    elif score <= 3: return "Weakly/Opposite"
    else: return "Moderately"

def decode_lnc_segment(segment_str, index):
    if not segment_str or index >= len(LNC_PAIRS_MAP):
        return None
    try:
        score = int(segment_str[0])
        trait_char = segment_str[1]
        intensity = int(segment_str[2])
    except (IndexError, ValueError):
        return None

    p1_char, p1_name, p2_char, p2_name = LNC_PAIRS_MAP[index]
    if trait_char == p1_char: active_trait = p1_name
    elif trait_char == p2_char: active_trait = p2_name
    else: active_trait = "Unknown"

    return {
        "trait": active_trait,
        "score": score,
        "intensity": intensity,
        "alignment": decode_alignment(score, "LNC")
    }

def decode_gne_segment(segment_str, index):
    if not segment_str or index >= len(GNE_TRAITS_MAP):
        return None
    try:
        trait_char = segment_str[0]
        score = int(segment_str[1])
    except (IndexError, ValueError):
        return None

    expected_char, name = GNE_TRAITS_MAP[index]
    if trait_char != expected_char:
        name = next((n for c, n in GNE_TRAITS_MAP if c == trait_char), "Unknown")

    return {
        "trait": name,
        "score": score,
        "strength": decode_gne_strength(score)
    }

def build_npc_prompt(dna_string, user_manual_input="", world_context=""):
    lnc_descriptions = []
    gne_descriptions = []
    
    try:
        match = re.match(r"\((\d+)/(\d+)\)\s+(.*)\s+-\s+(.*)", dna_string.strip())
        if not match: return "Error: DNA string format invalid."

        lnc_avg = int(match.group(1))
        gne_avg = int(match.group(2))
        lnc_str = match.group(3)
        gne_str = match.group(4)
        overall_lnc = decode_alignment(lnc_avg, "LNC")
        overall_gne = decode_alignment(gne_avg, "GNE")
        
        lnc_segments = lnc_str.split(',')
        for i, seg in enumerate(lnc_segments):
            data = decode_lnc_segment(seg, i)
            if data:
                lnc_descriptions.append(f"- **{data['trait']}**: {data['alignment']} expression (Score {data['score']}), Intensity {data['intensity']}/5.")

        gne_segments = gne_str.split(',')
        for i, seg in enumerate(gne_segments):
            data = decode_gne_segment(seg, i)
            if data:
                gne_descriptions.append(f"- **{data['trait']}**: {data['strength']} (Score {data['score']})")

    except Exception as e:
        return f"Error parsing DNA logic: {str(e)}"

    prompt = f"""
# **Professional AI Delegation Brief (RAFT)**

## **1. THE CORE BRIEF**

**Role:** Act as a **Master Storyteller and Game Designer** specializing in complex character psychology for TTRPGs.

**Audience:** A **Game Master** running a campaign who needs a fully realized, unique NPC immediately. They require depth, not tropes.

**Format:** A structured **Markdown Character Profile** with specific sections:
- **Header:** Name, Role, Alignment, Metaphor
- **Profile:** Appearance, Personality, Internal Conflict
- **Backstory:** Origin & History
- **Behavioral Model:** Beliefs, Desires, Intentions (BDI)

**Task:** Decode the provided Personality DNA string and World Context into a unique character profile. Your goal is to **resolve all internal contradictions** in the DNA into narrative tension, creating a believable person rather than a stat block.

---

## **2. CONTEXTUAL LAYERS**

**Situation Context (Inputs):**
1.  **User Manual Overrides (ABSOLUTE PRIORITY):** {user_manual_input if user_manual_input else "None."}
2.  **World Context:** {world_context if world_context else "Standard Setting."}
3.  **DNA Analysis (The Soul):**
    *   **Overall Alignment:** {overall_lnc} {overall_gne} (Derived from averages {lnc_avg}/{gne_avg})
    *   **PAIRED TRAITS (LNC - Structure & Intensity):**
        {chr(10).join(lnc_descriptions)}
    *   **UNPAIRED TRAITS (GNE - Moral Compass):**
        {chr(10).join(gne_descriptions)}

**Output Context (Constraints & Style):**
*   **Tone:** Evocative, precise, and novelist-quality. Avoid dry "game manual" language.
*   **Structure:** Use clear headings and bold text for key traits.
*   **Avoid:** Generic tropes (e.g., "brooding rogue in a corner"), "As an AI" apologies, or explaining the DNA mechanics in the final text.
*   **Mandatory:** 
    *   **Resolve Conflicts:** If the DNA has contradictory traits (e.g., "Brave" but "Cautious"), explain *why*.
    *   **Respect Intensity:** High intensity (4-5) must define them. Low intensity (1-2) must be subtle or hidden.

---

## **3. GENERATION INSTRUCTIONS**

Proceed directly to generating the character profile. Do not ask clarifying questions. Follow the DNA logic strictly unless overridden by User Inputs.
"""
    return prompt

def build_world_prompt(dna_string, user_manual_input="", context_input=""):
    """
    Constructs the LLM prompt for World Generation based on RAFT Framework.
    """
    prompt = f"""
# **Professional AI Delegation Brief (RAFT)**

## **1. THE CORE BRIEF**

**Role:** Act as a **World Decoding AI** and **Expert Worldbuilder**. You specialize in creating coherent, interconnected settings from abstract data.

**Audience:** A **Game Master** who needs a rich, playable campaign setting immediately. They need conflicts, hooks, and distinct regions, not just a list of stats.

**Format:** A structured **Markdown World Bible** with the following sections:
1.  **World Overview** (Tech/Magic/Authority levels & Genre)
2.  **Regional Breakdown** (Distinct areas with unique traits)
3.  **Critical Situations** (Thresholds & Chain Reactions)
4.  **Evolution & Trends** (What is rising/falling?)
5.  **Current State** (The immediate crisis/status quo)
6.  **Future Trajectories** (Optimistic vs Pessimistic paths)
7.  **Story Elements / Campaign Hooks** (Game-ready seeds)

**Task:** Decode the provided World DNA string into a living setting. The DNA contains numeric values for various aspects (Environment, Society, History, etc.). **Interpret these values creatively.** High numbers generally mean high intensity, prevalence, or advancement. Low numbers mean scarcity, primitivism, or weakness.

**Crucial:** If data appears contradictory (e.g., "Arid Environment" but "High Rainfall"), re-interpret it as a **tension** or **anomaly** (e.g., "A magical monsoon in a desert"). **Do not discard data.**

---

## **2. CONTEXTUAL LAYERS**

**Situation Context (Inputs):**
1.  **User Manual Overrides (ABSOLUTE PRIORITY):** {user_manual_input if user_manual_input else "None."}
2.  **Additional Context:** {context_input if context_input else "Standard Fantasy."}
3.  **DNA Code (The Blueprint):**
    ```
    {dna_string}
    ```

**Decoding Keys (Guidance):**
*   **Scales (T/M/A/R):** T=Technology, M=Magic, A=Authority, R=Regions. (1-10 Scale).
*   **Blocks:** ENV=Environment, SOC=Society, CON=Conflict, HIS=History, REG=Region, CRIT=Critical Threshold, CHAIN=Chain Reaction, EVO=Evolution.
*   **Logic:** 
    *   **T (Tech):** 1=Stone Age, 5=Medieval, 9=Sci-Fi/Steampunk.
    *   **M (Magic):** 1=Low/None, 5=Common, 9=Omnipresent/God-like.
    *   **A (Authority):** 1=Anarchy, 5=Feudal/Republic, 9=Totalitarian Hivemind.

---

## **3. GENERATION INSTRUCTIONS**

Proceed directly to generating the World Bible. Do not ask clarifying questions. Follow the DNA logic strictly unless overridden by User Inputs.
"""
    return prompt

def build_faction_prompt(dna_string, user_manual_input="", context_input=""):
    """
    Constructs the LLM prompt for Faction Generation based on RAFT Framework and provided Decoder Logic.
    """
    prompt = f"""
# **Professional AI Delegation Brief (RAFT)**

## **1. THE CORE BRIEF**

**Role:** Act as a **Faction DNA Decoder**, performing your duties with the finesse of a **Master Storyteller**. You specialize in creating secret societies, guilds, and cults that feel ancient and dangerous.

**Audience:** A **Game Master** who needs a fully realized faction profile suitable for direct presentation to players. Not a list of stats, but a living organization.

**Format:** A structured **Faction Profile** with sections:
1.  **Faction Name & Symbol** (Evocative, based on traits)
2.  **Origin & Founding Legend** (In-world lore)
3.  **Doctrine & Beliefs** (Why they exist)
4.  **Tactics & Influence** (How they operate: Bribes? Magic? Force?)
5.  **Leadership** (Personality/Archetype, not just a title)
6.  **Culture & Rituals** (Daily life of a member)
7.  **Public Reputation** (Feared? Unknown? Beloved?)
8.  **Current Conflict** (Internal schism or external threat)
9.  **Hidden Truth** (The dark secret they protect)
10. **Narrative Hooks** (3 adventure seeds)

**Task:** Decode the provided Faction DNA string. Translate the coded traits into a cohesive narrative.
*   **Contradictions are Gold:** If a faction is "Military (T4)" but uses "Theft (M5)" methods, explain it (e.g., "A disgraced legion turned to banditry").
*   **Immersive Tone:** Do not reference the DNA codes (e.g., "T1") in the final output. Write as if describing a real group.

---

## **2. CONTEXTUAL LAYERS**

**Situation Context (Inputs):**
1.  **User Manual Overrides (ABSOLUTE PRIORITY):** {user_manual_input if user_manual_input else "None."}
2.  **World Context:** {context_input if context_input else "Standard Fantasy Setting."}
3.  **DNA Code (The Blueprint):**
    ```
    {dna_string}
    ```

**Decoding Keys (Guidance):**
*   **T (Type):** 1=Cult, 2=Guild, 3=Syndicate, 4=Military, 5=Arcane, 6=Conspiracy, 7=Monster Clan.
*   **G (Goal):** 01=Control, 02=Suppress Knowledge, 03=Protect Relic, 04=Destroy Rival, 05=Spread Belief, 06=Wealth, 07=Awaken Being, 08=Reshape Society, 09=Reclaim Knowledge, 10=Arcane Tech, 11=Seal Horror, 12=Break Monopoly.
*   **M (Methods):** 1=Politics, 2=Law/Bureaucracy, 3=Conversion, 4=Blackmail, 5=Theft/Smuggling, 6=Rituals, 7=Force, 8=Infiltration.
*   **P (Power):** 1=Divine, 2=Wealth, 3=Resources, 4=Secrets, 5=Magic, 6=Crime, 7=Military, 8=Ideology.
*   **S (Structure):** 1=Loose, 2=Secret Council, 3=Charismatic Leader, 4=Hierarchy, 5=Caste, 6=Cells.
*   **O (Origin):** 1=Prophecy, 2=Civil War, 3=Exile, 4=Breakaway, 5=Guard Secret, 6=Underground, 7=Artifact.
*   **L (Leader):** 01=Visionary, 02=Tyrant, 03=Tactician, 04=Prophet, 05=Mask, 06=Caretaker, 07=Opportunist, 08=Ghost, 09=Revolutionary, 10=Administrator.
*   **F (Tone):** 1=Militaristic, 2=Elegant Criminal, 3=Frenzied, 4=Bureaucratic, 5=Apocalyptic, 6=Rational.
*   **D (Conflict):** 1=Rebellion, 2=Leadership Struggle, 3=Resource Crisis, 4=Spy, 5=Exposure Risk, 6=Missing Leader.
*   **A (Alignment):** 1=LG, 2=NG, 3=CG, 4=TN, 5=LN, 6=CN, 7=LE, 8=NE, 9=CE.
*   **SC (Scope):** 1=Local, 2=City, 3=Region, 4=Global, 5=Planar.
*   **MZ (Members):** 1=Elites, 2=Commoners, 3=Outcasts, 4=Fanatics, 5=Monsters, 6=Mixed.
*   **X (Rival):** 1=Faction, 2=Church, 3=Nobles, 4=Heroes, 5=Monsters, 6=Heretics.

---

## **3. GENERATION INSTRUCTIONS**

Proceed directly to generating the Faction Profile. Do not ask clarifying questions. Follow the DNA logic strictly unless overridden by User Inputs.
"""
    return prompt

def build_item_prompt(dna_string, user_manual_input="", context_input=""):
    """
    Constructs the LLM prompt for Magic Item Generation based on RAFT Framework.
    """
    prompt = f"""
# **Professional AI Delegation Brief (RAFT)**

## **1. THE CORE BRIEF**

**Role:** Act as an **Arcane Archivist and Game Designer**. You are decoding the "DNA" of a magical artifact to reveal its true nature, history, and mechanics.

**Audience:** A **Game Master** looking for a memorable loot drop or plot device. They need mechanics, flavor, and "soul," not just +1 stats.

**Format:** A structured **Magic Item Profile** with sections:
1.  **Item Name** (Evocative, based on properties)
2.  **Visual Identity** (Sensory description, material, craftsmanship)
3.  **Magical Effects** (What it does, cost to use, limitations)
4.  **History & Lore** (Origin, previous owners, legends)
5.  **Attunement** (Requirements, bonding process, side effects)
6.  **The Catch/Curse** (Every great item has a cost)
7.  **Evolution** (How it grows or decays over time)
8.  **System-Agnostic Mechanics** (Suggested rules)

**Task:** Decode the provided Item DNA string. 
*   **Contradiction Handling:** If the item is "Cursed (LOR:C99)" but "Revered (LOR:R99)," explain it (e.g., "A sword that kills the wielder but saves the kingdom").
*   **Evolution Logic:**
    *   **STABLE:** Resistant to change.
    *   **RISING/ACCELERATING:** Power grows with use/age.
    *   **DECAYING:** Power fades; a dying artifact.
    *   **VOLATILE/FLUCTUATING:** Unpredictable spikes.

---

## **2. CONTEXTUAL LAYERS**

**Situation Context (Inputs):**
1.  **User Manual Overrides (ABSOLUTE PRIORITY):** {user_manual_input if user_manual_input else "None."}
2.  **World Context:** {context_input if context_input else "Standard Fantasy Setting."}
3.  **DNA Code (The Blueprint):**
    ```
    {dna_string}
    ```

**Decoding Keys (Guidance):**
*   **PHY:** M=Material, S=Size, A=Appearance, D=Durability, C=Craftsmanship, W=Weight, F=Form, T=Texture.
*   **MAG:** E=Effect, P=Potency, D=Duration, C=Control, S=Source, A=Activation, R=Requirement, T=Transformation.
*   **HIS:** O=Origin, C=Creator, A=Age, R=Reputation, L=Legacy, F=Former Owners, D=Destiny, S=Stories.
*   **LOR:** K=Known, F=Forgotten, N=Notorious, C=Curses, R=Revered, E=Enigmatic, I=Impact, S=Secrets.
*   **ATTUNE:** U=User Req, W=Wielder Effect, C=Compatibility, M=Mutation, S=Synergy, V=Vision, P=Price, R=Restriction.

---

## **3. GENERATION INSTRUCTIONS**

Proceed directly to generating the Magic Item Profile. Do not ask clarifying questions. Follow the DNA logic strictly unless overridden by User Inputs.
"""
    return prompt

def build_location_prompt(dna_string, user_manual_input="", context_input=""):
    """
    Constructs the LLM prompt for Location Generation based on RAFT Framework.
    """
    prompt = f"""
# **Professional AI Delegation Brief (RAFT)**

## **1. THE CORE BRIEF**

**Role:** Act as a **Settlement Architect and Worldbuilder**. You decode procedural DNA to create living, breathing places with sensory depth.

**Audience:** A **Game Master** needing a ready-to-run location. They need atmosphere, NPCs, and conflicts, not just a map key.

**Format:** A structured **Settlement Profile** with sections:
1.  **Settlement Name** (Evocative)
2.  **Overview** (Core identity & major contradictions)
3.  **Physical Description** (Sights, smells, architecture, defenses)
4.  **Population** (Attitude, diversity, health, daily life)
5.  **Economy** (Trade, resources, wealth distribution)
6.  **Politics & Law** (Who rules? How corrupt? How free?)
7.  **Notable Locations** (Key POIs like Inns, Shops, Temples)
8.  **Surroundings** (Wilderness, neighbors, threats)
9.  **Trajectory** (Is it rising or falling? Why?)
10. **Hooks & Opportunities** (Adventures waiting to happen)

**Task:** Decode the provided Location DNA string.
*   **Interpret Scales:** 1-33=Low (Crumbling/Poor/Sparse), 34-66=Medium (Average/Stable), 67-99=High (Grand/Rich/Dense).
*   **Core vs Reality:** If Core Size is High but Population Block is Low, explain it (e.g., "A massive ghost town," or "A city of empty palaces").
*   **Evolution:** Use EVO trends (RISING, DECLINING) to describe the *change* happening now.

---

## **2. CONTEXTUAL LAYERS**

**Situation Context (Inputs):**
1.  **User Manual Overrides (ABSOLUTE PRIORITY):** {user_manual_input if user_manual_input else "None."}
2.  **World Context:** {context_input if context_input else "Standard Fantasy Setting."}
3.  **DNA Code (The Blueprint):**
    ```
    {dna_string}
    ```

**Decoding Keys (Guidance):**
*   **STRUCT:** S=Size, D=Density, W=Walls, A=Architecture, F=Fortification, U=Layout, P=Public, Q=Quality.
*   **POP:** S=Size, D=Diversity, A=Attitude, C=Crime, L=Health, H=Happiness, C=Culture, M=Morale.
*   **ECON:** M=Markets, T=Trade, R=Resources, P=Prices, S=Services, G=Guilds, I=Industry, L=Labor.
*   **POL:** G=Gov Type, C=Corruption, R=Religion, F=Freedom, S=Stability, C=Control, L=Law, I=Influence.
*   **POI:** I=Inns, S=Shops, R=Temples, D=Dungeons, H=Homes, A=Academies, M=Military, C=Civic.
*   **PROXI:** Relationship to WILD, TOWN, CITY, RUIN.

---

## **3. GENERATION INSTRUCTIONS**

Proceed directly to generating the Settlement Profile. Do not ask clarifying questions. Follow the DNA logic strictly unless overridden by User Inputs.
"""
    return prompt

def build_quest_prompt(dna_string, user_manual_input="", context_input=""):
    """
    Constructs the LLM prompt for Quest Generation based on RAFT Framework and provided Decoder Logic.
    """
    prompt = f"""
# **Professional AI Delegation Brief (RAFT)**

## **1. THE CORE BRIEF**

**Role:** Act as a **Quest DNA Decoder**, performing your duties with the insight of a **Master Storyteller** and the precision of a **Game Designer**.

**Audience:** A **Game Master** who needs a rich, detailed, and playable adventure scenario. Not a stat block, but a story module.

**Format:** A structured **Quest Profile** with sections:
1.  **Quest Title** (Evocative)
2.  **The Hook** (Visceral introduction)
3.  **Background & Context** (The "Why")
4.  **Core Objectives** (Goals & Hidden goals)
5.  **Obstacles & Challenges** (How to overcome them)
6.  **Adventure Structure & Flow** (Beginning/Middle/End based on EVO data)
7.  **Key Scenes & Encounters** (Specific moments)
8.  **Rewards & Spoils** (Tangible & Intangible)
9.  **Narrative & Tone** (Themes, pacing)
10. **Potential Outcomes** (Success/Failure/Twist)

**Task:** Decode the provided Quest DNA string. 
*   **Values 1-33:** LOW (Easy/Minor)
*   **Values 34-66:** MEDIUM (Moderate)
*   **Values 67-99:** HIGH (Challenging/Significant)

**CRITICAL LOGIC:**
1.  **Reconcile Obstacles vs. Engagement:** If an obstacle (e.g., Combat) is HIGH but Engagement (Combat) is LOW, you MUST describe how players bypass it (stealth, diplomacy, etc.). DO NOT force combat where engagement is low.
2.  **EVO (Evolution):** Use the `RISING`, `DESCENDING`, or `STABLE` patterns to pace the adventure.
3.  **Conflicts:** If Rewards are Low but Difficulty is High, explain *why* (e.g., Moral duty, blackmail, survival).

---

## **2. CONTEXTUAL LAYERS**

**Situation Context (Inputs):**
1.  **User Manual Overrides (ABSOLUTE PRIORITY):** {user_manual_input if user_manual_input else "None."}
2.  **World Context:** {context_input if context_input else "Standard Fantasy Setting."}
3.  **DNA Code (The Blueprint):**
    ```
    {dna_string}
    ```

**Decoding Keys (Guidance):**
*   **GOAL:** C=Clarity, R=Retrieval, E=Elimination, S=Survival, T=Travel, P=Protection, L=Liberation, H=Hidden.
*   **OBS (Obstacles):** C=Combat, M=Magical, E=Environmental, P=Political, T=Time, S=Social, G=Guardians, N=Natural.
*   **REWARD:** M=Monetary, I=Item, K=Knowledge, R=Reputation, S=Skill, P=Power, L=Loyalty, A=Access.
*   **NARR (Narrative):** T=Tension, C=Climax, P=Twists, M=Moral, H=Humor, R=Romance, I=Intrigue, A=Aftermath.
*   **ENGAGE:** Explicit combat/social/explore/puzzle focus.

---

## **3. GENERATION INSTRUCTIONS**

Proceed directly to generating the Quest Profile. Do not ask clarifying questions. Follow the DNA logic strictly unless overridden by User Inputs.
"""
    return prompt
```

See also: [[Prism Engine Code]], [[Generator Init]]
