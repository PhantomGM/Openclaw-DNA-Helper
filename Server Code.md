# DNA PCG Server Architect - Implementation Code

This is the `server.py` code running the FastAPI backend for the Prism Engine.

```python
# server.py
# DNA PCG System - API Server
# Uses FastAPI to expose procedural generation engines.

import os
from typing import Optional
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel

# Import our Engines
import generators
import prompt_builder

# --- CONFIGURATION ---
VAULT_PATH = r"C:\Users\nickd\Desktop\Openclaw DNA Helper"
VERSION = "1.0.0"

app = FastAPI(title="Prism Engine API", version=VERSION)

# Enable CORS (for local frontend development)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# --- DATA MODELS ---

class GenerateResponse(BaseModel):
    type: str
    dna: str
    description: str = "Raw DNA string generated successfully."

class DecodeRequest(BaseModel):
    type: str
    dna: str
    manual_override: Optional[str] = ""
    world_context: Optional[str] = "Standard Fantasy Setting."

class DecodeResponse(BaseModel):
    prompt: str
    system_role: str = "Expert Game Master"

class SaveRequest(BaseModel):
    filename: str
    content: str
    folder: Optional[str] = "Generated Content"

class SaveResponse(BaseModel):
    success: bool
    path: str
    message: str

# --- MAPPINGS ---

GENERATOR_MAP = {
    "npc": generators.generate_npc_dna,
    "faction": generators.generate_faction_dna,
    "quest": generators.generate_quest_dna,
    "item": generators.generate_item_dna,
    "location": generators.generate_location_dna,
    "world": generators.WorldDNAGenerator().generate_dna,
    "regional_poi": generators.generate_regional_poi_dna
}

# --- ENDPOINTS ---

@app.get("/health")
def health_check():
    """Server heartbeat."""
    return {"status": "ok", "version": VERSION, "vault_path": VAULT_PATH}

@app.get("/vault/status")
def vault_status():
    """Checks vault accessibility."""
    if os.path.exists(VAULT_PATH):
        try:
            files = os.listdir(VAULT_PATH)
            return {"status": "connected", "exists": True, "file_count": len(files)}
        except Exception as e:
            return {"status": "error", "exists": True, "error": str(e)}
    return {"status": "disconnected", "exists": False, "error": "Path not found"}

@app.post("/generate/{gen_type}", response_model=GenerateResponse)
def generate_dna(gen_type: str):
    """Generates fresh DNA for the requested type."""
    gen_type = gen_type.lower()
    
    if gen_type not in GENERATOR_MAP:
        raise HTTPException(status_code=404, detail=f"Generator '{gen_type}' not found. Available: {list(GENERATOR_MAP.keys())}")
    
    try:
        dna_func = GENERATOR_MAP[gen_type]
        dna_string = dna_func()
        return GenerateResponse(type=gen_type, dna=dna_string)
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/decode", response_model=DecodeResponse)
def decode_dna(request: DecodeRequest):
    """Converts DNA + Context into a Prompt."""
    t = request.type.lower()
    
    try:
        if t == "npc":
            prompt = prompt_builder.build_npc_prompt(request.dna, request.manual_override, request.world_context)
        elif t == "world":
            prompt = prompt_builder.build_world_prompt(request.dna, request.manual_override, context_input=request.world_context)
        elif t == "faction":
            prompt = prompt_builder.build_faction_prompt(request.dna, request.manual_override, context_input=request.world_context)
        elif t == "quest":
            prompt = prompt_builder.build_quest_prompt(request.dna, request.manual_override, context_input=request.world_context)
        elif t == "item":
            prompt = prompt_builder.build_item_prompt(request.dna, request.manual_override, context_input=request.world_context)
        elif t == "location":
            prompt = prompt_builder.build_location_prompt(request.dna, request.manual_override, context_input=request.world_context)
        else:
            # Fallback for generic/unimplemented types
            prompt = f"Decode this {t} DNA: {request.dna}\nContext: {request.world_context}\nOverride: {request.manual_override}"
            
        return DecodeResponse(prompt=prompt)
        
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Decoding error: {str(e)}")

@app.post("/vault/save", response_model=SaveResponse)
def save_to_vault(request: SaveRequest):
    """Saves content to the Obsidian Vault."""
    if not os.path.exists(VAULT_PATH):
        raise HTTPException(status_code=500, detail="Vault path not found on server.")
    
    # 1. Sanitize Filename (Basic)
    safe_filename = "".join(c for c in request.filename if c.isalnum() or c in (' ', '.', '_', '-')).strip()
    if not safe_filename.endswith(".md"):
        safe_filename += ".md"
        
    # 2. Create Directory
    target_dir = os.path.join(VAULT_PATH, request.folder)
    try:
        os.makedirs(target_dir, exist_ok=True)
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Could not create directory: {str(e)}")
        
    # 3. Write File
    full_path = os.path.join(target_dir, safe_filename)
    try:
        with open(full_path, "w", encoding="utf-8") as f:
            f.write(request.content)
        return SaveResponse(success=True, path=full_path, message="File saved successfully.")
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Write error: {str(e)}")

if __name__ == "__main__":
    import uvicorn
    # Host 0.0.0.0 allows local network access, Port 8000 is standard
    print("Starting Prism Engine Server...")
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

See also: [[Server Architecture]], [[Project Progress Summary]]
