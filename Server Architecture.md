# DNA PCG System: Backend Architectural Specification

## 1. Architecture Overview

The **Prism Server** acts as a stateless procedural generation engine layered over a file-based persistence system. It exposes the core "Prism Engine" logic via a RESTful API, allowing a frontend to request content generation, validate DNA strings, and commit finalized content to the user's local Obsidian Vault.

### Key Components
1.  **FastAPI Application:** The high-performance web server.
2.  **Generator Interface:** A router that directs requests to the `generators/` package.
3.  **DNA Decoder:** Middleware that uses `prompt_builder.py` to translate raw DNA into LLM-ready prompts.
4.  **Vault Manager:** A utility layer for reading/writing Markdown files to the configured Vault path.

## 2. API Design

### Base URL: `/api/v1`

### A. Generator Endpoints
These endpoints handle the creation of new raw data.

*   `POST /generate/{type}`
    *   **Description:** Generates a fresh DNA string for a specific content type.
    *   **Parameters:** `type` (npc, faction, quest, item, location, world).
    *   **Response:** `{ "dna": "T5-G10...", "type": "faction", "raw_data": { ... } }`

*   `POST /decode`
    *   **Description:** Converts a DNA string + User Context into a "God Prompt" for the LLM.
    *   **Body:** `{ "dna": "...", "type": "...", "manual_override": "...", "world_context": "..." }`
    *   **Response:** `{ "prompt": "# Professional AI Delegation Brief...", "dna_analysis": { ... } }`

### B. Persistence Endpoints (Vault Integration)
These endpoints interact with the user's file system.

*   `GET /vault/status`
    *   **Description:** Checks if the Vault path is accessible and writable.
    *   **Response:** `{ "status": "connected", "path": "C:/Users/nickd/...", "file_count": 42 }`

*   `POST /vault/save`
    *   **Description:** Saves a generated profile to the Vault as a Markdown file.
    *   **Body:** `{ "filename": "The Ledger of Flame.md", "content": "# The Ledger of Flame...", "folder": "Factions" }`
    *   **Response:** `{ "success": true, "path": "..." }`

### C. System Endpoints
*   `GET /health`
    *   **Description:** Server heartbeat.
    *   **Response:** `{ "status": "ok", "version": "1.0.0" }`

## 3. Integration Logic

### Modular Dispatch
The server will not contain generation logic directly. Instead, it will import the `prism_engine` map:

```python
# pseudo-code
from generators import generate_npc_dna, generate_faction_dna

GENERATOR_MAP = {
    "npc": generate_npc_dna,
    "faction": generate_faction_dna,
    # ...
}
```

### State Management
*   **Session State:** None. The server does not remember previous generations. The Frontend is responsible for holding the "Draft" state.
*   **Persistence:** Directly to `.md` files. This ensures "Data Sovereignty"—if the server dies, your data is still just text files in your folder.

## 4. Implementation Plan

### Phase 1: Core Server (The Skeleton)
*   Initialize `server.py` with FastAPI.
*   Implement `GET /health` and `POST /generate/{type}`.
*   Verify it can call the existing `generators/` package.

### Phase 2: The Decoder (The Brain)
*   Implement `POST /decode`.
*   Connect it to `prompt_builder.py`.
*   Ensure it handles the "RAFT" format correctly.

### Phase 3: The Vault (The Memory)
*   Implement `POST /vault/save`.
*   Add logic to sanitize filenames and create folders if they don't exist.

## 5. Verification Protocol
*   **Logic Check:** Ensure `dna` string format matches the specific regex/structure of each generator type.
*   **Source Check:** Use Pydantic models for request validation.
*   **Specificity Check:** Ensure error messages (e.g., "Invalid DNA format") are descriptive.

See also: [[Server Code]], [[Project Progress Summary]]
