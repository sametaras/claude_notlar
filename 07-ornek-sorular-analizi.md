# 12 Örnek Soru — Detaylı Analiz

Her soru için: **şıklar, doğru cevap, neden doğru, her distractor neden yanlış**, ve çıkardığımız **genelleştirilebilir ilke**.

---

## Senaryo: Customer Support Resolution Agent

### Soru 1 — Get_customer atlanması
> Agent %12 vakada `get_customer`'ı atlıyor, direkt `lookup_order`'a gidiyor. Misidentification + yanlış refund. En etkili düzeltme?

**A)** Programmatic prerequisite — `lookup_order` ve `process_refund`'ı blokla (get_customer verified ID dönene kadar).
**B)** System prompt'a "mandatory" yaz.
**C)** Few-shot: agent her zaman önce `get_customer` çağırsın.
**D)** Routing classifier, request tipine göre tool subset'ini enable et.

**✅ Doğru: A**

**Neden:**
- Kritik iş kuralı + finansal sonuç → **deterministik compliance** şart.
- Prompt (B) ve few-shot (C) probabilistic — sıfır-olmayan hata oranı.
- Routing (D) tool **sırası** değil, **availability**'sini ayarlar; gerçek problem sıra.

**Genel ilke:** Finansal / güvenlik / uyum gerektiren sıra bağımlılıklarında hook.

---

### Soru 2 — Benzer description, yanlış tool seçimi
> `get_customer` vs `lookup_order` minimal description, model yanlış tool seçiyor. İlk adım?

**A)** 5-8 few-shot örnek.
**B)** Her tool'un description'ını genişlet (input format, örnek, edge, boundary).
**C)** Routing layer keyword-based.
**D)** İki tool'u tek `lookup_entity`'ye birleştir.

**✅ Doğru: B**

**Neden:**
- Tool description = LLM'in seçim sinyali. Root cause: minimal description.
- Few-shot (A) token yakar, ilgili değil.
- Routing (C) LLM'in doğal dil understanding'ini bypass eder.
- Consolidation (D) mimari değişiklik, "first step" için fazla.

**Genel ilke:** Distractor'lar semptom tedavisi önerir; doğru cevap root cause'u çözer.

---

### Soru 3 — Escalation calibration
> %55 FCR, %80 hedef. Basit vakalar escalate, karmaşık vakaları autonomously çözmeye çalışıyor. Düzeltme?

**A)** Explicit escalation criteria + few-shot (escalate vs resolve).
**B)** Self-reported confidence score threshold.
**C)** Historical ticket üzerinde trained classifier.
**D)** Sentiment-based escalation.

**✅ Doğru: A**

**Neden:**
- Root cause: belirsiz decision boundary → explicit criteria.
- Self-confidence (B) poorly calibrated — agent zaten hard case'lerde yanlış confident.
- Classifier (C) over-engineered, labeled data gerekli.
- Sentiment (D) ≠ complexity.

**Genel ilke:** Prompt optimizasyonu denenmeden ML/classifier eklemek = over-engineering.

---

## Senaryo: Code Generation with Claude Code

### Soru 4 — Takım çapında /review
> Her developer'da mevcut olsun. Nereye koymalı?

**A)** `.claude/commands/` (repo içi).
**B)** `~/.claude/commands/` (home).
**C)** `CLAUDE.md` root.
**D)** `.claude/config.json` commands array.

**✅ Doğru: A**

**Neden:**
- Project scope = version control = takım otomatik alır.
- (B) kişisel, paylaşılmaz.
- (C) komut tanımı için değil, context için.
- (D) **mevcut değil** — böyle bir mekanizma yok.

**Genel ilke:** "Takım", "version control ile", "herkeste olsun" → project scope.

---

### Soru 5 — Monolith → microservices
> Dozens of files, servis sınırları + modül dependency kararları. Yaklaşım?

**A)** **Plan mode** — codebase'i keşfet, dependency'i anla, approach tasarla.
**B)** Direct execution, incremental — implementation boundary'leri göstersin.
**C)** Direct + upfront detailed spec.
**D)** Direct, sonra karmaşıklaşırsa plan mode.

**✅ Doğru: A**

**Neden:**
- Plan mode tam olarak bu senaryo için tasarlanmış: large-scale + architectural + multiple valid approach.
- (B) geç dependency keşfi = maliyetli rework.
- (C) zaten bilmiyorsun ki detaylı spec yazasın.
- (D) karmaşıklık **zaten belli** — sonradan keşfedilmeyecek.

**Genel ilke:** Karmaşıklık önceden biliniyorsa plan mode **baştan**.

---

### Soru 6 — Test file'lar her yerde
> `Button.test.tsx`, `Button.tsx` yanında. Convention'lar her dosya tipine aynı uygulansın. Maintainable çözüm?

**A)** `.claude/rules/` + YAML frontmatter glob pattern.
**B)** Root CLAUDE.md'de header'lar altında tüm convention'lar.
**C)** `.claude/skills/` her code type için.
**D)** Her subdirectory'e ayrı CLAUDE.md.

**✅ Doğru: A**

**Neden:**
- Glob (`**/*.test.tsx`) konuma bakmadan file-type'a uygulanır.
- (B) inference'a bağımlı, unreliable.
- (C) skill = on-demand; "automatic apply" talebiyle çelişir.
- (D) subdirectory CLAUDE.md directory-bound; scatter pattern için uygun değil.

**Genel ilke:** Convention file-type'a bağlı ve çok dizine yayılmışsa → path rule + glob.

---

## Senaryo: Multi-Agent Research System

### Soru 7 — Eksik coverage
> "Impact of AI on creative industries" → rapor sadece visual arts. Coordinator subtask'ları: "AI in digital art", "graphic design", "photography". Root cause?

**A)** Synthesis agent coverage gap tespiti yapmıyor.
**B)** Coordinator'un task decomposition'ı dar; subagent atamaları tüm alanları kapsamıyor.
**C)** Web search agent'ın sorguları yetersiz.
**D)** Document analysis non-visual kaynakları filtreliyor.

**✅ Doğru: B**

**Neden:**
- Log direkt gösteriyor: "creative industries" → yalnızca visual subtopics. Müzik, writing, film yok.
- Subagent'lar atandıkları scope'ta **doğru** çalışıyor.
- (A, C, D) çalışan downstream ajanları suçluyor.

**Genel ilke:** Coverage gap = coordinator decomposition hatası. Subagent'lar sadece atandıkları işte mükemmel olabilir.

---

### Soru 8 — Web search subagent timeout
> Error propagation yaklaşımı?

**A)** Structured error context (failure type, attempted query, partial, alternatives).
**B)** Subagent içinde retry with backoff, sonra generic "search unavailable".
**C)** Timeout'u yakala, empty result'ı success olarak dön.
**D)** Timeout'u top-level handler'a propagate, tüm workflow terminate.

**✅ Doğru: A**

**Neden:**
- Coordinator'un intelligent karar vermesi için context gerek.
- (B) generic status bilgi kaybı.
- (C) error suppression = sessiz bozukluk.
- (D) gereksiz terminate, recovery mümkündü.

**Genel ilke:** Multi-agent sistemlerde error = **veri**, suppressed olmamalı.

---

### Soru 9 — Synthesis → verification overhead
> Synthesis fact-check için coordinator'a dönüyor, 2-3 round trip, %40 latency. %85 basit, %15 karmaşık. Çözüm?

**A)** Synthesis'e scoped `verify_fact` tool ver; karmaşık vakalar coordinator'dan geçsin.
**B)** Synthesis batchleyip sona toplasın, coordinator bir kerede çözsün.
**C)** Synthesis'e tüm web search tool'larını ver.
**D)** Web search proactive cache (anticipate what synthesis needs).

**✅ Doğru: A**

**Neden:**
- Principle of least privilege + majority case optimization.
- (B) synthesis step'leri birbirine bağımlı; blocking batch yanlış.
- (C) over-provision, specialization bozulur.
- (D) speculative cache güvenilir değil.

**Genel ilke:** %85 vakayı ucuzlatan scoped tool + %15 için existing coordination.

---

## Senaryo: Claude Code for CI/CD

### Soru 10 — Job hang
> `claude "Analyze..."` interactive input bekliyor. Doğru?

**A)** `claude -p "..."`.
**B)** `CLAUDE_HEADLESS=true` env var.
**C)** `stdin < /dev/null` redirect.
**D)** `claude --batch "..."`.

**✅ Doğru: A**

**Neden:**
- `-p` / `--print` = dokümante edilmiş non-interactive mode.
- (B, D) non-existent feature.
- (C) Unix workaround; Claude Code'un command syntax'ına uygun değil.

**Genel ilke:** CI → `-p` refleksi.

---

### Soru 11 — Pre-merge + nightly her ikisi de Batch?
> Blocking pre-merge check + nightly technical debt. %50 cost savings için ikisi de batch'e geçsin mi?

**A)** Sadece nightly batch; pre-merge realtime.
**B)** Her ikisi batch + status polling.
**C)** Her ikisi realtime kalsın, batch ordering sorunu.
**D)** Her ikisi batch + realtime fallback (timeout'ta).

**✅ Doğru: A**

**Neden:**
- Batch 24h window, SLA yok → blocking pre-merge için **uygun değil**.
- Nightly latency-tolerant → batch ideal.
- (B) "genelde hızlıdır" güvenilmez.
- (C) yanılgı — `custom_id` ile ordering korunur.
- (D) gereksiz karmaşıklık.

**Genel ilke:** API'yi use-case'e göre seç; hepsini aynı kaba sokma.

---

### Soru 12 — 14 dosya PR review inconsistent
> Detailed feedback bazı dosyalarda, superficial diğerlerinde, contradictory findings. Restructure?

**A)** File-by-file local pass + cross-file integration pass.
**B)** Developer 3-4 dosya halinde submit etsin.
**C)** Daha büyük context window (üst tier model).
**D)** 3 bağımsız review pass, 2/3 consensus'ta flag.

**✅ Doğru: A**

**Neden:**
- Attention dilution = doğru problem. Split = doğru çözüm.
- (B) sorunu developer'a atar.
- (C) context window ≠ attention quality.
- (D) gerçek bug'ları bastırır (intermittent detection).

**Genel ilke:** "Inconsistent" + "contradictory" + "many files" → multi-pass split.

---

## 12 Sorudan Çıkan Meta-İlkeler

1. **Hook > prompt** (kritik compliance'ta).
2. **Root cause > semptom** (en düşük efor + en yüksek impact).
3. **Tool description** tool selection'ın birincil kaldıracı.
4. **Prompt optimization** önce denenmeli — ML/classifier sonra.
5. **Self-confidence & sentiment** güvenilmez escalation sinyalleri.
6. **Subagent isolated context** → prompt'ta açıkça ver.
7. **Coverage gap = coordinator sorunu**, subagent değil.
8. **Structured error** > generic status.
9. **Scoped tool** > over-provisioning veya speculative caching.
10. **Plan mode** complexity'si önceden belli büyük işler için.
11. **Path-based rules + glob** file-type convention scatter'ı için.
12. **Multi-pass split** attention dilution için; context window **değil**.
