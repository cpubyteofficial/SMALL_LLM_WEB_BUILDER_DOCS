# Token-Driven Web Builder LLM (<4B Parameters) — Complete Documentation

## 1. Overview & Architecture

Small language models (**0.8B – 4B parameters**) fail reliably when asked to write 2,500–4,000 tokens of raw HTML, responsive CSS, and `<canvas>` JavaScript from scratch. Common failure modes include unclosed tags, broken flexbox/grid layouts, unreadable color contrast, and mid-file truncation.

This architecture solves that by splitting generation into **two layers**:

```text
┌─────────────────────────┐      ~150–350 tokens       ┌───────────────────────────────┐
│  <4B LLM (Qwen3.5 / Q3) │ ─────────────────────────► │  Deterministic JS Compiler    │
│  Intent, Copy, Colors,  │   Strict JSON / Page DSL   │  Valid HTML5, Responsive CSS, │
│  Canvas & Section Order │   (GBNF-constrained)       │  WCAG Contrast & <canvas> FX  │
└─────────────────────────┘                            └───────────────────────────────┘
             ▲                                                         │
             │        "Make it sunset orange with wave canvas"         ▼
             └─────────────────── (~40-token Theme Patch) ──► Live Full HTML Output
```

### Why This Works on <4B Models
1. **10× fewer output tokens:** The LLM outputs ~200 structured tokens instead of ~3,000 raw code tokens.
2. **Zero syntax errors:** Output is locked by a **JSON Schema / GBNF Grammar** during decoding.
3. **Instant live restyling:** Changing colors, fonts, or `<canvas>` animations only updates the `theme` / `canvas` block—never breaking page layout.
4. **Fast, cheap fine-tuning:** Short sequences train in 15–30 minutes on a free T4 GPU or pure `llama.cpp`.

---

## 2. Recommended Base Models (<4B)

| Model | Params | `Q4_K_M` Size | Best Use Case |
| :--- | :--- | :--- | :--- |
| **`Qwen3.5-4B`** | 4.0B | ~2.6 GB | Best copywriting, layout planning, and complex multi-section pages |
| **`Qwen2.5-Coder-3B`** | 3.1B | ~1.9 GB | Fast structured JSON/tag adherence on laptops |
| **`Qwen3-1.7B`** | 1.7B | ~1.1 GB | Ultra-fast CPU / edge / browser-adjacent generation |
| **`Qwen3.5-0.8B`** / **`SmolLM3-3B`** | 0.8B–3B | 0.6–1.9 GB | Minimal hardware with strict GBNF grammar enabled |

> **Quantization Recommendation:** For models under 4B, **`Q4_K_M`** is already small (~1–2.6 GB) and preserves near-full intelligence. Only drop to `Q2_K` / `IQ2_M` if targeting ultra-low-RAM hardware (<1 GB).

---

## 3. Website Blueprint Schema (`v1.0`)

### 3.1 Full Page Blueprint (JSON Format)

```json
{
  "mode": "create",
  "meta": {
    "title": "NovaFlow AI — Build Pages Faster",
    "description": "Lightweight AI web builder."
  },
  "theme": {
    "bg": "#0b1020",
    "surface": "#141b33",
    "primary": "#38bdf8",
    "accent": "#818cf8",
    "text": "#f8fafc",
    "muted": "#94a3b8",
    "font": "sans",
    "radius": "14px"
  },
  "canvas": {
    "type": "particles",
    "speed": 1.0,
    "density": 55,
    "opacity": 0.45
  },
  "sections": [
    {
      "type": "navbar",
      "brand": "NovaFlow",
      "links": ["About", "Features", "Testimonials", "Contact"],
      "cta": "Get Started"
    },
    {
      "type": "hero",
      "badge": "v2.0 Live",
      "title": "Launch Clean Landing Pages in Seconds",
      "subtitle": "Describe your business, pick your mood, and export production-ready HTML.",
      "primary_cta": "Start Building",
      "secondary_cta": "View Demo"
    },
    {
      "type": "about",
      "heading": "Built for Speed & Clarity",
      "body": "We replace bloated templates with lightweight semantic components and real-time canvas visuals."
    },
    {
      "type": "features",
      "heading": "Everything You Need",
      "items": [
        { "title": "Instant Themes", "desc": "Swap palettes and typography in one command." },
        { "title": "Live Canvas", "desc": "GPU-friendly animated backgrounds out of the box." },
        { "title": "Clean Export", "desc": "Zero external dependencies; single HTML file output." }
      ]
    },
    {
      "type": "testimonial",
      "heading": "Loved by Builders",
      "items": [
        {
          "quote": "Our startup landing page went live in 10 minutes with a clean 99 Lighthouse score.",
          "author": "Riya Sharma",
          "role": "Founder, PixelCraft"
        },
        {
          "quote": "Changing color schemes on client calls takes one prompt now.",
          "author": "Arjun Mehta",
          "role": "Freelance Designer"
        }
      ]
    },
    {
      "type": "cta",
      "heading": "Ready to Ship Your Front Page?",
      "subtitle": "Export clean HTML5 with one click.",
      "button": "Create My Page"
    },
    {
      "type": "footer",
      "brand": "NovaFlow",
      "copyright": "© 2026 NovaFlow. All rights reserved."
    }
  ]
}
```

### 3.2 Patch / Edit Format (For Fast User Modifications)
When the user already has a page and says *"Change background to cream, primary to forest green, and switch canvas to waves"*, the LLM only outputs a minimal patch:

```json
{
  "mode": "patch",
  "theme": {
    "bg": "#faf7f2",
    "surface": "#ffffff",
    "primary": "#15803d",
    "accent": "#16a34a",
    "text": "#1c1917",
    "muted": "#57534e"
  },
  "canvas": {
    "type": "waves",
    "speed": 0.9,
    "density": 40,
    "opacity": 0.35
  }
}
```

### 3.3 Supported Enumerations
* **`theme.font`**: `"sans"` | `"serif"` | `"mono"`
* **`canvas.type`**: `"particles"` | `"waves"` | `"grid_glow"` | `"blobs"` | `"none"`
* **`sections[].type`**: `"navbar"` | `"hero"` | `"about"` | `"features"` | `"testimonial"` | `"pricing"` | `"cta"` | `"footer"`

### 3.4 Optional Compact XML Token DSL (Alternative to JSON)
If you prefer ultra-short custom markup instead of JSON:

```xml
<SITE mode="create" title="NovaFlow">
  <THEME bg="#0b1020" surface="#141b33" primary="#38bdf8" accent="#818cf8" text="#f8fafc" muted="#94a3b8" font="sans" radius="14px" />
  <CANVAS type="particles" speed="1.0" density="55" opacity="0.45" />
  <NAVBAR brand="NovaFlow" links="About|Features|Testimonials" cta="Get Started" />
  <HERO badge="v2.0" title="Launch Clean Pages" subtitle="Fast, responsive HTML." primary_cta="Start" secondary_cta="Demo" />
  <ABOUT heading="About Us" body="Lightweight semantic web tools for modern teams." />
  <TESTIMONIAL quote="Shipped in 5 minutes!" author="Riya Sharma" role="Founder" />
  <CTA heading="Ready to Launch?" button="Create Page" />
  <FOOTER brand="NovaFlow" copyright="© 2026 NovaFlow" />
</SITE>
```

---

## 4. `llama.cpp` GBNF Grammar (`site_blueprint.gbnf`)

Save this as `site_blueprint.gbnf` and pass `--grammar-file site_blueprint.gbnf` to `llama-server` or `llama-cli`. This guarantees even a 0.8B model cannot produce invalid JSON, invalid hex colors, or unknown canvas/section types.

```gbnf
root ::= "{" ws "\"mode\"" ws ":" ws mode-val "," ws
         ("\"meta\"" ws ":" ws meta-obj "," ws)?
         ("\"theme\"" ws ":" ws theme-obj "," ws)?
         ("\"canvas\"" ws ":" ws canvas-obj ("," ws)?)?
         ("\"sections\"" ws ":" ws sections-arr)?
         ws "}" ws

mode-val ::= "\"create\"" | "\"patch\""

meta-obj ::= "{" ws "\"title\"" ws ":" ws string "," ws "\"description\"" ws ":" ws string ws "}"

theme-obj ::= "{" ws
  "\"bg\"" ws ":" ws hex-color "," ws
  "\"surface\"" ws ":" ws hex-color "," ws
  "\"primary\"" ws ":" ws hex-color "," ws
  "\"accent\"" ws ":" ws hex-color "," ws
  "\"text\"" ws ":" ws hex-color "," ws
  "\"muted\"" ws ":" ws hex-color
  ("," ws "\"font\"" ws ":" ws font-val)?
  ("," ws "\"radius\"" ws ":" ws string)?
  ws "}"

font-val ::= "\"sans\"" | "\"serif\"" | "\"mono\""

canvas-obj ::= "{" ws
  "\"type\"" ws ":" ws canvas-type "," ws
  "\"speed\"" ws ":" ws number "," ws
  "\"density\"" ws ":" ws integer "," ws
  "\"opacity\"" ws ":" ws number
  ws "}"

canvas-type ::= "\"particles\"" | "\"waves\"" | "\"grid_glow\"" | "\"blobs\"" | "\"none\""

sections-arr ::= "[" ws (section ("," ws section)*)? ws "]"

section ::= navbar-sec | hero-sec | about-sec | features-sec | testimonial-sec | cta-sec | footer-sec

navbar-sec ::= "{" ws "\"type\"" ws ":" ws "\"navbar\"" "," ws "\"brand\"" ws ":" ws string "," ws "\"links\"" ws ":" ws str-arr "," ws "\"cta\"" ws ":" ws string ws "}"

hero-sec ::= "{" ws "\"type\"" ws ":" ws "\"hero\"" "," ws "\"badge\"" ws ":" ws string "," ws "\"title\"" ws ":" ws string "," ws "\"subtitle\"" ws ":" ws string "," ws "\"primary_cta\"" ws ":" ws string ("," ws "\"secondary_cta\"" ws ":" ws string)? ws "}"

about-sec ::= "{" ws "\"type\"" ws ":" ws "\"about\"" "," ws "\"heading\"" ws ":" ws string "," ws "\"body\"" ws ":" ws string ws "}"

features-sec ::= "{" ws "\"type\"" ws ":" ws "\"features\"" "," ws "\"heading\"" ws ":" ws string "," ws "\"items\"" ws ":" ws feature-arr ws "}"
feature-arr ::= "[" ws feature-item ("," ws feature-item)* ws "]"
feature-item ::= "{" ws "\"title\"" ws ":" ws string "," ws "\"desc\"" ws ":" ws string ws "}"

testimonial-sec ::= "{" ws "\"type\"" ws ":" ws "\"testimonial\"" "," ws "\"heading\"" ws ":" ws string "," ws "\"items\"" ws ":" ws test-arr ws "}"
test-arr ::= "[" ws test-item ("," ws test-item)* ws "]"
test-item ::= "{" ws "\"quote\"" ws ":" ws string "," ws "\"author\"" ws ":" ws string "," ws "\"role\"" ws ":" ws string ws "}"

cta-sec ::= "{" ws "\"type\"" ws ":" ws "\"cta\"" "," ws "\"heading\"" ws ":" ws string "," ws "\"subtitle\"" ws ":" ws string "," ws "\"button\"" ws ":" ws string ws "}"

footer-sec ::= "{" ws "\"type\"" ws ":" ws "\"footer\"" "," ws "\"brand\"" ws ":" ws string "," ws "\"copyright\"" ws ":" ws string ws "}"

str-arr ::= "[" ws (string ("," ws string)*)? ws "]"
hex-color ::= "\"#" [0-9a-fA-F]{6} "\""
string ::= "\"" ( [^"\\\x7F\x00-\x1F] | "\\" (["\\/bfnrt] | "u" [0-9a-fA-F]{4}) )* "\""
integer ::= [0-9]+
number ::= [0-9]+ ("." [0-9]+)?
ws ::= [ \t\n\r]*
```

> **Alternative:** With `llama-server`, you can also simply pass `"response_format": { "type": "json_schema", ... }` in the OpenAI-compatible API request instead of writing GBNF manually.

---

## 5. Dataset & Fine-Tuning Pipeline (Zero Unsloth)

Fine-tuning makes a 1.7B–4B model follow your custom section style, color harmony, and patch behavior natively without giant system prompts.

### 5.1 Dataset Format (`train.jsonl`)
Aim for **500–2,000 examples** split into three categories:
* **60% Full Creation (`mode: "create"`)**: Diverse industries (SaaS, cafe, portfolio, fitness, agency, developer tools), varied palettes (dark neon, minimal cream, brutalist monochrome, pastel).
* **30% Incremental Edits (`mode: "patch"`)**: User supplies current theme/sections and asks to change colors, swap canvas animation, or add/reorder sections.
* **10% Copywriting Refinements**: Rewriting hero headlines or testimonials in specific tones.

Example row in `train.jsonl`:
```json
{
  "messages": [
    {
      "role": "system",
      "content": "You are a Web Blueprint AI. Output valid JSON conforming to the Site Blueprint v1.0 schema."
    },
    {
      "role": "user",
      "content": "Create a warm bakery front page with hero, about us, customer testimonials, and soft floating blob canvas."
    },
    {
      "role": "assistant",
      "content": "{\"mode\":\"create\",\"meta\":{\"title\":\"Crumb & Hearth Bakery\",\"description\":\"Artisan sourdough and pastries.\"},\"theme\":{\"bg\":\"#fffbf5\",\"surface\":\"#ffffff\",\"primary\":\"#b45309\",\"accent\":\"#d97706\",\"text\":\"#292524\",\"muted\":\"#78716c\",\"font\":\"serif\",\"radius\":\"16px\"},\"canvas\":{\"type\":\"blobs\",\"speed\":0.6,\"density\":18,\"opacity\":0.25},\"sections\":[{\"type\":\"hero\",\"badge\":\"Fresh Daily\",\"title\":\"Handcrafted Sourdough, Baked Every Dawn\",\"subtitle\":\"Stone-milled flour, slow fermentation, and neighborhood warmth.\",\"primary_cta\":\"Order Morning Box\",\"secondary_cta\":\"See Menu\"},{\"type\":\"about\",\"heading\":\"Our Kitchen\",\"body\":\"Founded in 2021, we bake small-batch loaves using heritage grains and 36-hour cold fermentation.\"},{\"type\":\"testimonial\",\"heading\":\"Morning Regulars\",\"items\":[{\"quote\":\"Best country loaf in the city—still warm when I walk in at 7 AM.\",\"author\":\"Meera Nair\",\"role\":\"Local Customer\"}]},{\"type\":\"cta\",\"heading\":\"Reserve Today's Bake\",\"subtitle\":\"Pickup ready daily from 7 AM to 2 PM.\",\"button\":\"Reserve a Loaf\"}]}"
    }
  ]
}
```

> **Note on Custom Vocabulary Tokens:** You *can* add custom tokens (like `<HERO>`, `<TESTIMONIAL>`) via `tokenizer.add_tokens(...)` + `model.resize_token_embeddings(len(tokenizer))`, but if you do, you **must** include `embed_tokens` and `lm_head` in `modules_to_save` during LoRA training so those new token IDs learn representations. Using standard JSON or standard-token XML tags avoids resizing embeddings and converges faster.

### 5.2 Option A: Pure HuggingFace PEFT Training Script (`train_lora.py`)
Works on a free Kaggle T4 (`16 GB`) or Colab GPU with zero Unsloth:

```python
import torch
from datasets import load_dataset
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
from peft import LoraConfig, get_peft_model, prepare_model_for_kbit_training
from trl import SFTTrainer, SFTConfig

MODEL_ID = "Qwen/Qwen3-1.7B"  # or "Qwen/Qwen3.5-4B"

tokenizer = AutoTokenizer.from_pretrained(MODEL_ID)
if tokenizer.pad_token is None:
    tokenizer.pad_token = tokenizer.eos_token

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True,
)

model = AutoModelForCausalLM.from_pretrained(
    MODEL_ID,
    quantization_config=bnb_config,
    device_map="auto",
)
model = prepare_model_for_kbit_training(model)

lora_config = LoraConfig(
    r=16,
    lora_alpha=32,
    target_modules="all-linear",
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM",
)
model = get_peft_model(model, lora_config)

dataset = load_dataset("json", data_files="train.jsonl", split="train")
def format_fn(batch):
    return {
        "text": [
            tokenizer.apply_chat_template(m, tokenize=False, add_generation_prompt=False)
            for m in batch["messages"]
        ]
    }
dataset = dataset.map(format_fn, batched=True)

trainer = SFTTrainer(
    model=model,
    train_dataset=dataset,
    processing_class=tokenizer,
    args=SFTConfig(
        dataset_text_field="text",
        max_length=2048,
        per_device_train_batch_size=2,
        gradient_accumulation_steps=4,
        num_train_epochs=3,
        learning_rate=2e-4,
        lr_scheduler_type="cosine",
        warmup_ratio=0.05,
        bf16=True,
        logging_steps=10,
        output_dir="web_token_lora",
    ),
)
trainer.train()
model.save_pretrained("web_token_lora/final")
tokenizer.save_pretrained("web_token_lora/final")
```

### 5.3 Merge & Convert to GGUF (`llama.cpp`)

```python
# merge.py — Load base in FP16/BF16 and merge LoRA cleanly
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel

base = AutoModelForCausalLM.from_pretrained("Qwen/Qwen3-1.7B", torch_dtype=torch.bfloat16)
tok = AutoTokenizer.from_pretrained("Qwen/Qwen3-1.7B")
merged = PeftModel.from_pretrained(base, "web_token_lora/final").merge_and_unload()
merged.save_pretrained("web_builder_bf16")
tok.save_pretrained("web_builder_bf16")
```

Convert and quantize with `llama.cpp`:

```bash
# 1. Convert to BF16 GGUF
python3 llama.cpp/convert_hf_to_gguf.py web_builder_bf16 \
  --outfile web-builder-bf16.gguf --outtype bf16

# 2. Recommended for <4B: Quantize to Q4_K_M (fast, high accuracy)
./llama.cpp/build/bin/llama-quantize \
  web-builder-bf16.gguf web-builder-Q4_K_M.gguf Q4_K_M

# 3. Optional (only if you MUST squeeze into ultra-small RAM):
# Calibrate an imatrix from your JSON blueprints + quantize surgically to Q2_K
./llama.cpp/build/bin/llama-imatrix -m web-builder-bf16.gguf -f calib_blueprints.txt -o web.imatrix
./llama.cpp/build/bin/llama-quantize --imatrix web.imatrix \
  --token-embedding-type q6_k --output-tensor-type q8_0 \
  web-builder-bf16.gguf web-builder-Q2_K.gguf Q2_K
```

### 5.4 Option B: Train Directly in Pure C++ (`llama-finetune-lora`)
If you already have a `.gguf` file and don't want Python/PyTorch training:

```bash
./llama.cpp/build/bin/llama-finetune-lora \
  -m qwen3-1.7b-Q4_K_M.gguf \
  -f train.jsonl \
  --assistant-loss-only \
  --lora-rank 16 --lora-alpha 32 \
  -c 2048 -b 256 -ub 128 -ngl 999 \
  -o web_style_lora.gguf
```
*(Verify exact flags with `./llama.cpp/build/bin/llama-finetune-lora --help`.)*

---

## 6. Deterministic HTML5 + CSS + Canvas Compiler (`compiler.js`)

This compiler accepts a `create` blueprint (or merges a `patch` onto the current blueprint) and outputs a complete, standalone, mobile-responsive HTML file with contrast-safe buttons and animated `<canvas>` backgrounds.

```javascript
/**
 * Merges a partial "patch" onto an existing blueprint, or initializes a full "create" blueprint.
 */
export function applyBlueprintUpdate(currentBlueprint, incoming) {
  if (!currentBlueprint || incoming.mode === "create") {
    return {
      mode: "create",
      meta: incoming.meta || { title: "Generated Website", description: "" },
      theme: {
        bg: "#0b1020", surface: "#141b33", primary: "#38bdf8",
        accent: "#818cf8", text: "#f8fafc", muted: "#94a3b8",
        font: "sans", radius: "14px",
        ...(incoming.theme || {})
      },
      canvas: {
        type: "particles", speed: 1.0, density: 50, opacity: 0.4,
        ...(incoming.canvas || {})
      },
      sections: Array.isArray(incoming.sections) ? incoming.sections : []
    };
  }

  // Patch mode: update only changed colors, canvas, or sections
  return {
    ...currentBlueprint,
    meta: incoming.meta ? { ...currentBlueprint.meta, ...incoming.meta } : currentBlueprint.meta,
    theme: incoming.theme ? { ...currentBlueprint.theme, ...incoming.theme } : currentBlueprint.theme,
    canvas: incoming.canvas ? { ...currentBlueprint.canvas, ...incoming.canvas } : currentBlueprint.canvas,
    sections: Array.isArray(incoming.sections) ? incoming.sections : currentBlueprint.sections
  };
}

function esc(str) {
  return String(str ?? "").replace(/[&<>"']/g, m => ({
    "&": "&amp;", "<": "&lt;", ">": "&gt;", "\"": "&quot;", "'": "&#39;"
  }[m]));
}

function contrastText(hex) {
  const clean = String(hex || "#38bdf8").replace("#", "");
  const full = clean.length === 3 ? clean.split("").map(c => c + c).join("") : clean;
  const r = parseInt(full.slice(0, 2), 16) || 0;
  const g = parseInt(full.slice(2, 4), 16) || 0;
  const b = parseInt(full.slice(4, 6), 16) || 0;
  const lum = (0.299 * r + 0.587 * g + 0.114 * b) / 255;
  return lum > 0.55 ? "#0b0f19" : "#ffffff";
}

function fontStack(fontKey) {
  if (fontKey === "serif") return "Georgia, 'Times New Roman', serif";
  if (fontKey === "mono") return "ui-monospace, SFMono-Regular, Menlo, Consolas, monospace";
  return "system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif";
}

function renderSection(s) {
  switch (s.type) {
    case "navbar":
      return `
      <header class="nav-wrap">
        <nav class="nav">
          <a class="brand" href="#">${esc(s.brand)}</a>
          <div class="nav-links">
            ${(s.links || []).map(l => `<a href="#${esc(l.toLowerCase().replace(/\s+/g, "-"))}">${esc(l)}</a>`).join("")}
            ${s.cta ? `<a class="btn btn-sm" href="#cta">${esc(s.cta)}</a>` : ""}
          </div>
        </nav>
      </header>`;

    case "hero":
      return `
      <section class="hero" id="hero">
        ${s.badge ? `<span class="badge">${esc(s.badge)}</span>` : ""}
        <h1>${esc(s.title)}</h1>
        <p class="hero-sub">${esc(s.subtitle)}</p>
        <div class="hero-actions">
          ${s.primary_cta ? `<a class="btn" href="#cta">${esc(s.primary_cta)}</a>` : ""}
          ${s.secondary_cta ? `<a class="btn btn-ghost" href="#about">${esc(s.secondary_cta)}</a>` : ""}
        </div>
      </section>`;

    case "about":
      return `
      <section class="card" id="about">
        <h2>${esc(s.heading || "About Us")}</h2>
        <p class="muted-text">${esc(s.body)}</p>
      </section>`;

    case "features":
      return `
      <section id="features">
        <h2 class="sec-title">${esc(s.heading || "Features")}</h2>
        <div class="grid-3">
          ${(s.items || []).map(it => `
            <div class="card">
              <h3>${esc(it.title)}</h3>
              <p class="muted-text">${esc(it.desc)}</p>
            </div>`).join("")}
        </div>
      </section>`;

    case "testimonial":
      return `
      <section id="testimonials">
        <h2 class="sec-title">${esc(s.heading || "Testimonials")}</h2>
        <div class="grid-2">
          ${(s.items || []).map(it => `
            <div class="card quote-card">
              <blockquote>“${esc(it.quote)}”</blockquote>
              <div class="author">${esc(it.author)} <span>${esc(it.role || "")}</span></div>
            </div>`).join("")}
        </div>
      </section>`;

    case "cta":
      return `
      <section class="card cta-box" id="cta">
        <h2>${esc(s.heading)}</h2>
        ${s.subtitle ? `<p class="muted-text">${esc(s.subtitle)}</p>` : ""}
        <a class="btn" href="#">${esc(s.button || "Get Started")}</a>
      </section>`;

    case "footer":
      return `
      <footer class="footer">
        <strong>${esc(s.brand)}</strong>
        <span class="muted-text">${esc(s.copyright)}</span>
      </footer>`;

    default:
      return "";
  }
}

function buildCanvasScript(canvas, theme) {
  const type = canvas?.type || "none";
  if (type === "none") return "";
  const speed = Number(canvas.speed ?? 1.0);
  const density = Math.max(5, Math.min(200, Number(canvas.density ?? 50)));
  const opacity = Math.max(0.05, Math.min(1, Number(canvas.opacity ?? 0.4)));

  return `
  (function(){
    const c = document.getElementById("bg-canvas");
    const ctx = c.getContext("2d");
    let w, h;
    function resize(){ w = c.width = window.innerWidth; h = c.height = window.innerHeight; }
    resize(); window.addEventListener("resize", resize);
    const primary = ${JSON.stringify(theme.primary)};
    const accent = ${JSON.stringify(theme.accent || theme.primary)};
    const mode = ${JSON.stringify(type)};
    const speed = ${speed};
    const count = ${density};
    ctx.globalAlpha = ${opacity};

    if (mode === "particles" || mode === "blobs") {
      const pts = Array.from({length: count}, (_, i) => ({
        x: Math.random() * w, y: Math.random() * h,
        vx: (Math.random() - 0.5) * speed, vy: (Math.random() - 0.5) * speed,
        r: mode === "blobs" ? 40 + Math.random() * 80 : 1.5 + Math.random() * 2.5,
        col: i % 2 === 0 ? primary : accent
      }));
      (function draw(){
        ctx.clearRect(0, 0, w, h);
        ctx.globalAlpha = mode === "blobs" ? ${opacity * 0.45} : ${opacity};
        for (const p of pts) {
          p.x = (p.x + p.vx + w) % w;
          p.y = (p.y + p.vy + h) % h;
          ctx.fillStyle = p.col;
          ctx.beginPath();
          ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
          ctx.fill();
        }
        requestAnimationFrame(draw);
      })();
    } else if (mode === "waves") {
      let t = 0;
      (function draw(){
        ctx.clearRect(0, 0, w, h);
        ctx.globalAlpha = ${opacity};
        t += 0.015 * speed;
        for (let k = 0; k < 3; k++) {
          ctx.beginPath();
          ctx.strokeStyle = k % 2 === 0 ? primary : accent;
          ctx.lineWidth = 2;
          for (let x = 0; x <= w; x += 12) {
            const y = h * (0.35 + k * 0.16) + Math.sin(x * 0.005 + t + k) * (35 + k * 14);
            if (x === 0) ctx.moveTo(x, y); else ctx.lineTo(x, y);
          }
          ctx.stroke();
        }
        requestAnimationFrame(draw);
      })();
    } else if (mode === "grid_glow") {
      let t = 0;
      (function draw(){
        ctx.clearRect(0, 0, w, h);
        ctx.globalAlpha = ${opacity * 0.6};
        ctx.strokeStyle = primary;
        ctx.lineWidth = 1;
        t = (t + 0.5 * speed) % 48;
        for (let x = 0; x < w; x += 48) { ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, h); ctx.stroke(); }
        for (let y = t; y < h; y += 48) { ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(w, y); ctx.stroke(); }
        requestAnimationFrame(draw);
      })();
    }
  })();`;
}

export function compileBlueprintToHTML(bp) {
  const t = bp.theme;
  const btnText = contrastText(t.primary);
  const bodyHTML = (bp.sections || []).map(renderSection).join("\n");

  return `<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>${esc(bp.meta?.title || "Generated Site")}</title>
  <meta name="description" content="${esc(bp.meta?.description || "")}" />
  <style>
    :root {
      --bg: ${esc(t.bg)};
      --surface: ${esc(t.surface)};
      --primary: ${esc(t.primary)};
      --accent: ${esc(t.accent || t.primary)};
      --text: ${esc(t.text)};
      --muted: ${esc(t.muted)};
      --btn-text: ${btnText};
      --radius: ${esc(t.radius || "14px")};
      --font: ${fontStack(t.font)};
    }
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      background: var(--bg); color: var(--text); font-family: var(--font);
      line-height: 1.6; min-height: 100vh; overflow-x: hidden;
    }
    #bg-canvas { position: fixed; inset: 0; z-index: 0; pointer-events: none; }
    .container {
      position: relative; z-index: 1; max-width: 1040px;
      margin: 0 auto; padding: 24px 20px 64px; display: grid; gap: 44px;
    }
    .nav {
      display: flex; justify-content: space-between; align-items: center;
      padding: 14px 20px; background: var(--surface); border-radius: var(--radius);
      border: 1px solid color-mix(in srgb, var(--text) 10%, transparent);
    }
    .brand { font-weight: 800; font-size: 1.1rem; color: var(--text); text-decoration: none; }
    .nav-links { display: flex; gap: 18px; align-items: center; flex-wrap: wrap; }
    .nav-links a { color: var(--muted); text-decoration: none; font-size: 0.92rem; }
    .nav-links a.btn { color: var(--btn-text); }
    .hero { text-align: center; padding: 56px 12px 32px; }
    .badge {
      display: inline-block; padding: 4px 12px; border-radius: 999px;
      font-size: 0.78rem; font-weight: 700; margin-bottom: 16px;
      background: color-mix(in srgb, var(--primary) 18%, transparent);
      color: var(--primary);
    }
    .hero h1 { font-size: clamp(2.1rem, 5vw, 3.6rem); line-height: 1.15; margin-bottom: 16px; }
    .hero-sub { color: var(--muted); font-size: 1.12rem; max-width: 640px; margin: 0 auto 28px; }
    .hero-actions { display: flex; gap: 12px; justify-content: center; flex-wrap: wrap; }
    .btn {
      display: inline-block; background: var(--primary); color: var(--btn-text);
      padding: 12px 24px; border-radius: calc(var(--radius) * 0.7);
      font-weight: 700; text-decoration: none;
    }
    .btn-sm { padding: 7px 14px; font-size: 0.85rem; }
    .btn-ghost {
      background: transparent; color: var(--text);
      border: 1px solid color-mix(in srgb, var(--text) 22%, transparent);
    }
    .card {
      background: var(--surface); padding: 28px; border-radius: var(--radius);
      border: 1px solid color-mix(in srgb, var(--text) 10%, transparent);
    }
    .sec-title { font-size: 1.6rem; margin-bottom: 18px; }
    .muted-text { color: var(--muted); }
    .grid-3 { display: grid; grid-template-columns: repeat(auto-fit, minmax(230px, 1fr)); gap: 18px; }
    .grid-2 { display: grid; grid-template-columns: repeat(auto-fit, minmax(290px, 1fr)); gap: 18px; }
    .quote-card blockquote { font-size: 1.05rem; margin-bottom: 12px; }
    .quote-card .author { font-weight: 700; font-size: 0.9rem; }
    .quote-card .author span { color: var(--muted); font-weight: 400; margin-left: 6px; }
    .cta-box { text-align: center; padding: 40px 24px; display: grid; gap: 14px; justify-items: center; }
    .footer {
      display: flex; justify-content: space-between; flex-wrap: wrap; gap: 12px;
      padding-top: 24px; border-top: 1px solid color-mix(in srgb, var(--text) 10%, transparent);
    }
  </style>
</head>
<body>
  <canvas id="bg-canvas"></canvas>
  <main class="container">${bodyHTML}</main>
  <script>${buildCanvasScript(bp.canvas, t)}<\/script>
</body>
</html>`;
}
```

---

## 7. Running the Full Live Studio Locally

### Step 1: Start `llama-server` with GBNF Grammar Enforcement
```bash
./llama.cpp/build/bin/llama-server \
  -m web-builder-Q4_K_M.gguf \
  --grammar-file site_blueprint.gbnf \
  --port 8080 \
  --ctx-size 4096 \
  -ngl 999
```

### Step 2: Connect Any Frontend UI via `fetch()`
```javascript
async function askWebAI(userPrompt, currentBlueprint) {
  const messages = [
    {
      role: "system",
      content: "Output a valid Site Blueprint v1.0 JSON. Use mode='create' for new sites and mode='patch' for color, canvas, or section edits."
    }
  ];

  if (currentBlueprint) {
    messages.push({
      role: "user",
      content: `Current state: ${JSON.stringify({ theme: currentBlueprint.theme, canvas: currentBlueprint.canvas })}\nRequest: ${userPrompt}`
    });
  } else {
    messages.push({ role: "user", content: userPrompt });
  }

  const res = await fetch("http://127.0.0.1:8080/v1/chat/completions", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      messages,
      temperature: 0.35,
      max_tokens: 700
    })
  });

  const data = await res.json();
  const parsed = JSON.parse(data.choices[0].message.content);
  const nextBlueprint = applyBlueprintUpdate(currentBlueprint, parsed);
  const fullHTML = compileBlueprintToHTML(nextBlueprint);
  return { nextBlueprint, fullHTML };
}
```

---

## 8. Best Practices & Troubleshooting

* **Low temperature (`0.2 – 0.45`):** Small models produce cleaner color contrast and tighter copy at lower temperatures.
* **Color Contrast Safety:** Even if the LLM outputs an unusual hex color for `primary`, `contrastText()` in `compiler.js` automatically switches button label text between `#0b0f19` and `#ffffff` so buttons remain readable.
* **Keep KV Cache at `F16` or `Q8_0`:** Avoid `Q4` KV cache quantization when running structured JSON/GBNF generation on <4B models.
* **Adding New Sections:** To support `<PRICING>` or `<FAQ>`, add the section type to `site_blueprint.gbnf`, add a `case` to `renderSection()` in `compiler.js`, and include ~30 examples of that section in `train.jsonl`.
