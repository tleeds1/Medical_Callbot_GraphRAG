# Medical_Callbot_GraphRAG
Medical Callbot Assist (GraphRAG, VI-enabled)

## Version 1: Medical Chatbot (GraphRAG, VI-enabled)
Overview
- A medical assistant chatbot that answers in Vietnamese with full diacritics and helps route users to the right department and schedule an appointment.
- Uses a hybrid GraphRAG pipeline: vector retrieval for unstructured knowledge + lightweight graph/CSV knowledge (departments, doctors, schedules, curated diseases) to ground responses and booking.
- Designed to generalize across common triage scenarios; avoids brittle, hard-coded case logic by combining heuristics, embeddings, and structured lookups.

Key Capabilities
- Vietnamese-first answering: forces clear, concise answers in Vietnamese; auto-rewrites non-VI generations.
- Graph + Vector Retrieval: merges graph facts (curated from medical_kg.csv and infor.md) with top‑K text chunks from embeddings.
- Symptom → Department routing: infers suspects (e.g., viêm ruột thừa, tiêu chảy cấp) and maps them to the most suitable department (Ngoại, Tiêu Hóa, Nhi, Sản...).
- Profile-aware booking: extracts basic patient profile (tuổi, giới tính, mang thai) from conversation and uses it to route (e.g., Nhi only when ≤12 tuổi) and to pick suitable doctors/slots.
- Structured booking: reads doctors, rooms, schedules, and channels from CSV to suggest options and can hold a provisional slot.
- Backend-flexible LLM: works with Ollama or Hugging Face pipelines; configurable model/temperature/top‑K.

How It Works (High Level)
1) Ingestion
   - Build sentence embeddings for documents into FAISS under embeddings/.
   - Maintain mapping of rows → ids and metadata (texts.jsonl, meta.jsonl, model.txt).

2) Retrieval
   - Vector search top‑K relevant chunks for the user query.
   - Graph facts inference via rule-based symptom matching (accent-insensitive) to produce concise facts and suspects.
   - Department routing with three layers:
     • Knowledge mapping from data/infor.md (curated diseases → department)
     • Heuristic keyword rules (e.g., đau quanh rốn → hố chậu phải → nghi viêm ruột thừa → Ngoại)
     • Embedding-based router tie-break (DepartmentRouter)

3) Conversation + Reasoning
   - Build a compact prompt with: system policy (Vietnamese only), Graph facts, retrieved context, recent hypothesis carry‑over, and patient profile summary.
   - Call LLM (Ollama preferred, HF fallback) and enforce Vietnamese output.

4) Booking Flow
   - Detect booking intent ("đặt lịch", "hẹn khám", ...).
   - Parse optional doctor name/weekday and select matching schedule; otherwise auto-pick department → doctors → earliest feasible slot.
   - Patient profile influences routing: e.g., ≤12 tuổi → Nhi; pregnant → Sản/Phụ sản.
   - Emit a clear Vietnamese confirmation (tạm giữ chỗ) with thời gian, phòng, bác sĩ, hotline; append profile summary.

Data & Knowledge Sources
- data/departments.csv — departments (id, name, aliases, floors, notes)
- data/doctors.csv — doctors (specialization, titles, phones, department)
- data/schedules.csv — day-of-week/time slots, room ids
- data/rooms.csv — room labels per department
- data/booking_channels.csv — hotline, web, counter, app
- data/medical_kg.csv — head/rel/tail triples used to enrich graph facts
- data/infor.md — curated disease → department lists (parsed to a mapping)

Generalization Strategy
- Accent-agnostic normalization for matching Vietnamese inputs reliably.
- Compose routing decisions from: (facts + suspects) ∪ (curated map) ∪ (router embeddings) to avoid single-point brittleness.
- Patient-profile extraction embedded into prompt and booking to reduce misrouting (e.g., pediatric vs adult appendicitis).

What You Can Expect
- Short, structured medical guidance (suspected diagnosis, safe self-care, red flags, booking suggestions) in Vietnamese.
- Evidence-citing prompt context (graph facts + retrieved snippets) to keep answers grounded.
- Deterministic, CSV-driven booking proposals that you can later wire to a real HIS system.

Demo Placeholders (add your images later)
![Demo Chat](docs/images/demo-chat-1.png)
![Booking Suggestion](docs/images/demo-booking-1.png)
![Graph Facts + Context](docs/images/demo-context-1.png)

Safety & Limitations
- Educational/triage assistant; not a substitute for a licensed clinician.
- Always advises escalation for red-flag symptoms (đau tăng, sốt cao, nôn liên tục, máu trong phân, mất nước nặng...).
- Coverage reflects the curated infor.md and available texts; expand these sources to broaden scope.

Roadmap
- Expand curated disease → department map, incl. pediatric vs adult nuances.
- Add lightweight forms to explicitly capture tuổi/giới tính/mang thai/thời điểm khởi phát.
- Optional appointment persistence to a DB or HIS API.
- Add evaluation harness of conversation transcripts for regression checks.

## Version 2: Callbot (...update soon...)

Contact (full code access)
- Le Doan Tho
- GitHub: https://github.com/tleeds1
- Email: thodoanle26@gmail.com
