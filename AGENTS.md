# AI Agent Guidelines

## Rules for AI Working on This Project
1. **Never make unsupported success claims.**
2. **Reproduce problems before fixing them.**
3. **Use one hypothesis at a time.**
4. **Verify every fix.**
5. **Never repeat a failed approach unless new evidence justifies it.**
6. **After every failed debugging attempt, immediately record it in DEBUG_LOG.md.**
7. **After a successful fix, record the root cause, final fix, and verification in DEBUG_LOG.md.**
8. **If debugging reaches 3 consecutive failures, stop making random changes and enter investigation mode.**
9. **If codebase-memory-mcp tool is available, use `search_graph`/`trace_call_path` to explore code structure before grepping manually or reading many files.**

---

## Naming Conventions & Architecture (Absolute Rules)

### BlockNote System
- **"BlockNote"** adalah nama STANDAR untuk sistem catatan yang dipakai seluruh format di project ini (Article, Timeline Note, Canvas Note, dll).
- **"BlockNote Template"** adalah referensi ke template/standar BlockNote, termasuk cara konversi ke preview mode.
- Setiap kali menyebut "BlockNote", ini MERUJUK ke standar sistem catatan ini — BUKAN nama library (`@blocknote/react`).

### Page & Article Page
- **"Page"** adalah container untuk catatan (analog dengan Page di Notion). Isi dari Page adalah **BlockNote** — mengikuti kapabilitas dan fitur editor BlockNote.
- **"Article Page"** adalah salah satu format/tipe dari Page.
- Konsep: Page = wrapper, BlockNote = content/editor engine di dalamnya.

### Implikasi Arsitektur
- **SATU** konfigurasi BlockNote (schema, uploadFile, extensions, paste handler) yang dishare ke SEMUA format/page.
- Semua editor surface (Article Page, Timeline Note, Canvas Note) memanggil shared BlockNote factory/hook yang sama.
- Tidak boleh ada BlockNote instance baru yang dikonfigurasi independently tanpa melewati shared factory.
- `uploadFile`, math, mention, paste handler — SEMUA konsisten di setiap surface.

### Contoh Referensi
- "BlockNote Template" konfigurasi ada di: `src/components/Editor.tsx` (canonical instance)
- BlockNote di Timeline Note: `src/timeline-engine/components/BlockNoteWrapper.jsx`
- Konversi ke preview: gunakan pattern yang sudah ada di BlockNote Template (read-only BlockNoteView)
