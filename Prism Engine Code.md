# Prism Engine Code

This is the main orchestrator script for the DNA system.

```python
# prism_engine.py
# --- THE PRISM ENGINE: ORCHESTRATOR ---
# Usage: python prism_engine.py --type npc --manual "Traits..." --context "World info..."

import argparse
import sys
import generators
import prompt_builder

# Map CLI types to generator functions
GENERATOR_MAP = {
    "npc": generators.generate_npc_dna,
    "faction": generators.generate_faction_dna,
    "quest": generators.generate_quest_dna,
    "item": generators.generate_item_dna,
    "location": generators.generate_location_dna,
    "world": generators.WorldDNAGenerator().generate_dna, # Class-based
    "regional_poi": generators.generate_regional_poi_dna
}

def load_context(context_input):
    """Loads context from a string or a file path."""
    if not context_input:
        return "Standard Fantasy Setting."
    
    # Check if it looks like a file path
    if context_input.endswith(".txt") or context_input.endswith(".md"):
        try:
            with open(context_input, 'r', encoding='utf-8') as f:
                return f.read()
        except FileNotFoundError:
            print(f"Warning: Context file '{context_input}' not found. Using as raw string.")
            return context_input
    return context_input

def run_engine(gen_type, manual_override, context_raw):
    print(f"--- 1. INITIALIZING PRISM ENGINE ({gen_type.upper()}) ---")
    
    # 1. Select Generator
    if gen_type not in GENERATOR_MAP:
        print(f"Error: Unknown generator type '{gen_type}'. Available: {list(GENERATOR_MAP.keys())}")
        return

    generator_func = GENERATOR_MAP[gen_type]
    
    # 2. Generate DNA
    print("--- 2. GENERATING DNA ---")
    raw_dna = generator_func()
    print(f"DNA String: {raw_dna}\n")

    # 3. Load Context
    world_context = load_context(context_raw)

    # 4. Build Prompt (Currently specialized for NPC, others need expansion)
    print("--- 3. BUILDING PROMPT ---")
    
    if gen_type == "npc":
        prompt = prompt_builder.build_npc_prompt(
            dna_string=raw_dna,
            user_manual_input=manual_override,
            world_context=world_context
        )
    elif gen_type == "world":
        prompt = prompt_builder.build_world_prompt(
            dna_string=raw_dna,
            user_manual_input=manual_override,
            context_input=world_context
        )
    elif gen_type == "quest":
        prompt = prompt_builder.build_quest_prompt(
            dna_string=raw_dna,
            user_manual_input=manual_override,
            context_input=world_context
        )
    elif gen_type == "faction":
        prompt = prompt_builder.build_faction_prompt(
            dna_string=raw_dna,
            user_manual_input=manual_override,
            context_input=world_context
        )
    elif gen_type == "item":
        prompt = prompt_builder.build_item_prompt(
            dna_string=raw_dna,
            user_manual_input=manual_override,
            context_input=world_context
        )
    elif gen_type == "location":
        prompt = prompt_builder.build_location_prompt(
            dna_string=raw_dna,
            user_manual_input=manual_override,
            context_input=world_context
        )
    else:
        # Fallback for non-NPC types until we build specific decoders
        prompt = f"""
# SYSTEM ROLE: The {gen_type.title()} Forge
Act as an expert Game Master. Decode the following DNA string into a {gen_type} description.

# INPUTS
1. USER OVERRIDE: {manual_override}
2. WORLD CONTEXT: {world_context}
3. DNA: {raw_dna}

# TASK
Interpret the DNA values creatively. Higher numbers = greater intensity/power.
Resolve any contradictions in the DNA.
"""

    print("--- 4. FINAL PROMPT OUTPUT ---")
    print(prompt)
    return prompt

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Prism Engine: TTRPG Procedural Generation")
    parser.add_argument("--type", type=str, required=True, help="Type of content (npc, faction, quest, etc.)")
    parser.add_argument("--manual", type=str, default="", help="User manual overrides")
    parser.add_argument("--context", type=str, default="", help="World context string or file path")

    args = parser.parse_args()
    
    run_engine(args.type, args.manual, args.context)
```

See also: [[Prompt Builder Code]], [[Generator Init]]
