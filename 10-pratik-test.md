# Pratik Test — 25 Soru

Kendi oluşturduğumuz ek pratik sınav. Sınav tarzında çoktan seçmeli, 1 doğru + 3 distractor. Önce çöz, sonra altta cevap anahtarı ve açıklamalar var.

**Hedef süre:** 60 dakika. Not: Gerçek sınavda 6 senaryodan 4'ü gelir; burada tüm senaryolara yayılmış sorular.

---

## Sorular

### S1 — (Domain 1)
Coordinator, synthesis subagent'ı çağırıyor ama synthesis "no findings provided" diye cevaplıyor. Coordinator'ın prompt'unda `Summarize the findings.` yazıyor ve web_search + doc_analysis subagent'ları zaten başarıyla çalışmış. Sebep?

A) Synthesis subagent'ının `allowedTools` listesi eksik.  
B) Coordinator, subagent findings'lerini synthesis'in prompt'una **açıkça koymamış**; subagent context isolated.  
C) Synthesis model versiyonu eski.  
D) Coordinator'ın system prompt'unda `"synthesize"` keyword'ü geçmiyor.

---

### S2 — (Domain 1)
Production'da refund flow bazen customer doğrulanmadan işleniyor. En güvenilir fix?

A) System prompt'a büyük harflerle "VERIFY CUSTOMER FIRST" yaz.  
B) Few-shot olarak 10 örnek ekle.  
C) PreToolUse hook ile `process_refund`'ı blokla — kayıtlı `verified_customer_id` yoksa.  
D) Confidence threshold altı kararları human'a route et.

---

### S3 — (Domain 1)
Agentic loop'u nasıl sonlandırmalı?

A) Assistant response'unda "done" kelimesi geçerse.  
B) `stop_reason == "end_turn"` olduğunda.  
C) 5 iteration sonra kesin.  
D) Tool result boş geldiğinde.

---

### S4 — (Domain 2)
İki MCP tool — `analyze_content` ve `analyze_document` — benzer şekilde tanımlı ve model karıştırıyor. İlk adım?

A) Tool'ları `analyze`'e birleştir.  
B) Description'ları genişlet + gerekirse rename (`analyze_web_result` vs `analyze_document`).  
C) Agent'a temperature=0 ver.  
D) Tool'un önüne routing classifier koy.

---

### S5 — (Domain 2)
Bir MCP tool transient error fırlattı. Agent ne bilgisi ile retry kararı verir?

A) HTTP status code only.  
B) Generic "operation failed" string.  
C) `isError: true`, `errorCategory: "transient"`, `isRetryable: true`, human-readable message.  
D) Stack trace.

---

### S6 — (Domain 2)
Takım paylaşımlı MCP server'ı nereye koyarsın?

A) `~/.claude.json`  
B) `.mcp.json` (repo içi)  
C) `.claude/commands/`  
D) `CLAUDE.md` @import

---

### S7 — (Domain 2)
Orada olmayan bir order arayan `lookup_order("ORD-9999")` 0 sonuç dönüyor. Nasıl raporlanmalı?

A) `isError: true, errorCategory: "validation"`.  
B) Success, empty array; error değil — valid empty result.  
C) Timeout gibi raporla, retry öner.  
D) Workflow'u terminate et.

---

### S8 — (Domain 3)
Takımın kullandığı `/review` slash command'ı nereye koymalı?

A) `.claude/commands/review.md`  
B) `~/.claude/commands/review.md`  
C) `CLAUDE.md` içinde inline tanım  
D) `.claude/config.json` commands array

---

### S9 — (Domain 3)
Test dosyaları (`*.test.tsx`) her dizine yayılmış. Hepsine aynı convention'ı nasıl garanti edersin?

A) Her dizine CLAUDE.md koy.  
B) Root CLAUDE.md'de "test dosyalarında..." bölümü yaz.  
C) `.claude/rules/test.md` + frontmatter `paths: ["**/*.test.tsx"]`.  
D) Skill olarak yaz, her testte manuel invoke et.

---

### S10 — (Domain 3)
50+ dosyayı etkileyecek bir library migration. Yaklaşım?

A) Direct execution, iterativ düzelt.  
B) Plan mode — keşfet, design, onayla, sonra uygula.  
C) Plan mode gereksiz; Claude zaten planlar.  
D) Her dosya için ayrı Claude Code session aç.

---

### S11 — (Domain 3)
CI pipeline'da `claude "review this PR"` hang oldu. Nedeni ve fix?

A) Model too slow; upgrade.  
B) Interactive input bekliyor; `-p` flag kullan.  
C) Output buffered; `--stream` flag kullan.  
D) `CLAUDE_HEADLESS=1` env var set et.

---

### S12 — (Domain 3)
Claude Code skill'in verbose exploration output üretiyor ve main session context'i şişiyor. Fix?

A) Skill'i çağırdıktan sonra `/compact`.  
B) SKILL.md frontmatter'ına `context: fork`.  
C) `allowed-tools: []` ver.  
D) Skill içinde `--output-format json`.

---

### S13 — (Domain 4)
JSON syntax error'ları elimine etmenin en güvenilir yolu?

A) Regex post-processing.  
B) Prompt'a "valid JSON üret" yaz.  
C) `tool_use` + JSON schema.  
D) Modelin `response_format: "json"` parametresi.

---

### S14 — (Domain 4)
Extraction'da dokümanda olmayan field var. Model hallucinate ediyor. Fix?

A) Prompt'a "asla uydurma" yaz.  
B) Field'ı schema'da `nullable` yap.  
C) Temperature düşür.  
D) Multiple extractions çalıştır, majority vote.

---

### S15 — (Domain 4)
Batch API ne zaman uygun?

A) Pre-merge blocking check.  
B) Live chat agent.  
C) Nightly technical debt report.  
D) Real-time fraud detection.

---

### S16 — (Domain 4)
Validation fail → retry ne zaman anlamlı?

A) Information dokümanda **yok**.  
B) Date formatı "Jan 5, 2024" geldi, ISO istedin.  
C) Hiçbir zaman — retry hep fabrikasyon üretir.  
D) Her fail durumunda.

---

### S17 — (Domain 4)
False positive yüksek PR review. İlk adım?

A) Explicit review criteria (hangi issue flag, hangi skip).  
B) Modeli bigger model'e upgrade et.  
C) "Be conservative" prompt'a ekle.  
D) 3 run'dan 2'sinde flag'lerse kabul et.

---

### S18 — (Domain 4)
Generated kodun review'ini kim yapmalı?

A) Aynı session'daki Claude.  
B) Geliştiriciler elle; Claude review'i zaten güvenilmez.  
C) Bağımsız ikinci Claude instance (generator reasoning yok).  
D) Extended thinking mode ile aynı model.

---

### S19 — (Domain 5)
Uzun konversasyonda kritik sayılar kayboluyor. En iyi pattern?

A) `/compact` sık kullan.  
B) Case facts block persist et, summarize dışında tutulur.  
C) Konuşmayı 10 turn'de kes, yeni session aç.  
D) Context window'u büyüt.

---

### S20 — (Domain 5)
Müşteri "insanla konuşmak istiyorum" dedi, konu basit. Ne yaparsın?

A) Önce yine de autonomous çöz, sonra belki escalate.  
B) Anında escalate.  
C) "Bu basit, ben hallederim" de.  
D) Confidence yüksekse direkt çöz.

---

### S21 — (Domain 5)
Iki subagent farklı istatistik döndürdü (aynı konuda). Synthesis ne yapar?

A) Daha yeni tarihli olanı seç.  
B) Ortalama hesapla.  
C) Her ikisini de kaynak attribution ile raporla, "contested findings" başlığı altında.  
D) Coordinator'a sor, o seçsin.

---

### S22 — (Domain 5)
%97 overall accuracy'li extraction. Otomatize edelim mi?

A) Evet, yüksek yeterli.  
B) Stratified sampling yap: doc-type x field breakdown; segmentlerde düşük performans varsa human keep.  
C) %99'a çıkar, öyle karar ver.  
D) Extended thinking ile %99.5'e çıkar.

---

### S23 — (Domain 5)
Web search subagent timeout oldu. Coordinator ne alır?

A) Generic "search unavailable".  
B) Empty result, success flag.  
C) Structured error: failure_type, attempted_query, partial_results, alternatives.  
D) Exception propagate edilir, workflow terminate.

---

### S24 — (Domain 1)
Coordinator paralel 3 subagent çalıştırmak istiyor ama sıralı çalışıyor. Sebep?

A) Subagent'ları farklı coordinator turn'lerinde çağırmış; aynı turn'de birden çok Task call olmalı.  
B) `allowedTools` eksik.  
C) Paralel çalışmıyor, mimari böyle.  
D) Token bütçesi yetmiyor.

---

### S25 — (Domain 3)
Convention'lar karmaşık: React hook'lar, API handler async/await, DB model repository pattern. Bakım kolay olsun diye?

A) Tek devasa CLAUDE.md içinde başlıklar.  
B) `.claude/rules/` altında topic-specific dosyalar + uygun path glob'lar.  
C) Her developer'ın `~/.claude/CLAUDE.md`'sine ekle.  
D) README.md'de yaz, Claude README okur.

---

## Cevap Anahtarı

| # | Cevap | Kısa açıklama |
|---|---|---|
| 1 | **B** | Subagent isolated context — findings'i prompt'a açıkça koy. |
| 2 | **C** | Kritik compliance → PreToolUse hook. |
| 3 | **B** | Normal sonlandırma `end_turn`. Doğal dil / cap primary olmaz. |
| 4 | **B** | Tool description = selection sinyali. Root cause fix. |
| 5 | **C** | Structured error metadata = intelligent recovery. |
| 6 | **B** | Paylaşılan → project-scope `.mcp.json`. |
| 7 | **B** | Access failure ≠ empty valid result. 0 sonuç success. |
| 8 | **A** | Takım paylaşımı → `.claude/commands/` repo. |
| 9 | **C** | File-type + scatter → rule + glob. |
| 10 | **B** | Large-scale, architectural → plan mode. |
| 11 | **B** | `-p` non-interactive mode (dokümante). `CLAUDE_HEADLESS` yok. |
| 12 | **B** | `context: fork` verbose skill'i izole eder. |
| 13 | **C** | tool_use + schema = syntax error eliminasyonu. |
| 14 | **B** | Nullable field → fabrikasyonu keser. |
| 15 | **C** | Overnight, latency-tolerant → Batch. |
| 16 | **B** | Format/structural fail → retry; info absent → nullable. |
| 17 | **A** | Explicit criteria (kategorik). "Be conservative" işe yaramaz. |
| 18 | **C** | Independent instance, generator'ın reasoning context'i yok. |
| 19 | **B** | Transactional fact'lar için persistent case facts block. |
| 20 | **B** | Explicit human request → anında escalate. |
| 21 | **C** | Attribute ederek her ikisini — "contested findings". |
| 22 | **B** | Aggregate yanılgısı — stratified sampling. |
| 23 | **C** | Structured error context. |
| 24 | **A** | Paralel = tek turn'de birden çok Task call. |
| 25 | **B** | Topic-specific + path glob = modüler, bakılabilir. |

---

## Skorlama
- **22-25**: Hazırsın. Tuzaklara son kez göz at, sınava gir.
- **18-21**: İyi. Yanlışları kategorize et (domain + tuzak tipi), ilgili dosyaları tekrar gözden geçir.
- **14-17**: Temel iyi ama zayıf domain var. O domain'e 2-3 gün ek ayır.
- **<14**: Hazırlık planını en baştan takip et. Hands-on eksik olabilir.

Yanlışlarını şöyle analiz et:
- Hangi domain'de kaybettin?
- Hangi tuzak ailesine düştün? (semptom-tedavisi, prompt-güven, sentiment-escalation, vs.)
- Gerçekten bilmediğin mi yoksa distraktör'e mi aldandığın?

İyi çalışmalar. 🎯
