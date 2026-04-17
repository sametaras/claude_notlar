# 4 Haftalık Hazırlık Planı

Günde 45-60 dakika çalışmayla 4 haftada hazır olursun. Hands-on pratik + teorik tekrar dengeli.

---

## Hafta 1 — Temeller ve Domain 1 (Agentic)

### Gün 1-2: Sınav haritası
- `00-sinav-genel-bakis.md` — iki kez oku.
- 6 senaryoyu listele, hangisinde hangi domain ağırlıkta not al.
- Anthropic resmi docs'tan Claude Agent SDK sayfalarını tara (agentic loop, AgentDefinition, Task tool, hooks).

### Gün 3-4: Agentic loop
- Basit bir agent yaz:
  ```python
  while True:
      resp = claude.messages.create(...)
      if resp.stop_reason == "end_turn": break
      if resp.stop_reason == "tool_use": 
          # execute, append, continue
  ```
- 2 custom tool ekle, test et.

### Gün 5: Multi-agent
- Coordinator + 2 subagent kur (web_search + document_analyzer).
- Paralel spawn test et: tek turn'de 2 Task call emit.
- Subagent context isolation'ı gözlemle — prompt'ta findings geçmeden synthesis'e çağır, eksik çıkışa şahit ol.

### Gün 6: Hook pratiği
- PreToolUse hook yaz: $500 üstü refund'u blokla.
- PostToolUse hook yaz: iki farklı timestamp format'ı ISO 8601'e normalize et.

### Gün 7: Rest + tekrar
- `01-domain1-agentic.md` çalışma listesini kontrol et.
- Örnek Soru 1, 7, 8, 9'u tekrar çöz.

---

## Hafta 2 — Domain 2 + Domain 3

### Gün 8: Tool tasarımı
- 2 benzer tool yaz (`analyze_content` vs `analyze_document`).
- Minimal description ile test → yanlış seçim oranını ölç.
- Description'ı kapsamlı yaz → oranı kıyasla.

### Gün 9: MCP server
- Basit MCP server kur (community Jira MCP veya kendi yazdığın).
- `.mcp.json` + `${ENV_VAR}` ile secret management.
- `isError` + `errorCategory` + `isRetryable` structured error implement et.

### Gün 10: Built-in tools + tool distribution
- Grep / Glob / Read / Write / Edit senaryolarında hangi hangi durumda pratik yap.
- Subagent'a 4 tool ver, sonra 18 tool ver → selection reliability'yi kıyasla.

### Gün 11: CLAUDE.md hiyerarşisi
- Bir repo'da project CLAUDE.md + subdirectory CLAUDE.md + `@import` dene.
- `/memory` komutu ile hangi dosya yükleniyor gözlemle.

### Gün 12: Slash commands + Skills
- `.claude/commands/review.md` yaz.
- `.claude/skills/explore-legacy/SKILL.md` frontmatter ile (`context: fork`, `allowed-tools`) yaz.
- Verbose skill'in main context'i kirletip kirletmediğini test et (fork varken yokken).

### Gün 13: .claude/rules/ + glob
- `paths: ["**/*.test.tsx"]` frontmatter'lı rule yaz.
- Test dosyası düzenlerken rule'un yüklendiğini, normal dosyada yüklenmediğini doğrula.

### Gün 14: Plan mode + CI/CD
- Plan mode ile küçük bir refactor planla, sonra direct execute.
- CI'de `claude -p "..." --output-format json` ile basit bir PR review job'ı yaz.

---

## Hafta 3 — Domain 4 + Domain 5

### Gün 15: Structured output
- JSON schema tasarla: required, nullable, enum + "other" + detail.
- tool_use + `tool_choice: "any"` ile invoice extraction yap.

### Gün 16: Few-shot
- Ambiguous tool selection vakaları için 3 örnekli few-shot yaz.
- Varied document format (inline citation vs bibliography) için 2 örnek ekle.

### Gün 17: Validation-retry
- Pydantic model ile schema doğrula.
- Fail → retry with error feedback.
- "Information absent" vakasında retry'ın faydasız olduğunu dene → nullable'a geç.

### Gün 18: Batch API
- 20 doküman için batch submit et.
- `custom_id` ile success/fail ayır; oversized'ları chunk'layarak resubmit et.
- Cost kıyaslaması hesapla.

### Gün 19: Context management
- Uzun konuşmada "case facts" block implement et.
- Tool output trimmer yaz (40 field → 5 field).
- Scratchpad file yaz/oku pattern dene.

### Gün 20: Escalation
- Explicit escalation criteria + 4 few-shot örnek ekle.
- Policy gap, explicit human request, no-progress için test dialog'ları çalıştır.

### Gün 21: Multi-source synthesis
- 3 farklı "source"tan finding üret, claim-source mapping ile topla.
- Çelişkili iki source ver → synthesis'in attribution yaparak her ikisini raporladığını doğrula.

---

## Hafta 4 — Tekrar, zayıf noktalar, pratik sınav

### Gün 22-23: Tüm çalışma listelerini gözden geçir
- `01` → `05` her dosyanın sonundaki checklist'i tek tek cevapla.
- Zayıf olduğun 2-3 task statement'ı işaretle.

### Gün 24: 12 örnek soru
- `07-ornek-sorular-analizi.md` — sadece soru + şıklara bak, 45 dk içinde çöz.
- Sonra analizleri oku, her distractor'ın **neden** yanlış olduğunu tekrar et.

### Gün 25: Tuzaklar + anti-pattern
- `08-tuzaklar-ve-antipatterns.md` iki kez.
- Kendine **her tuzak için bir örnek** düşün.

### Gün 26: Pratik test (repo'daki `10-pratik-test.md`)
- Zaman tut: 25 soru, 60 dakika.
- Sonunda skor al, yanlışları **kategorize et** (hangi domain? hangi tuzak?).

### Gün 27: Yanlışlara odaklı tekrar
- Kategorize edilmiş yanlışlar için ilgili domain dosyasını tekrar oku.
- Ek olarak Anthropic resmi Practice Exam'i çöz (eğer link paylaşılmışsa).

### Gün 28: Light review + rest
- Son tuzak listesini oku.
- Erken uyu, sınava dinlenmiş gir.

---

## Hazırlık egzersizleri (resmi, sınav guide'ından)
PDF bunları vurguluyor:

1. **Multi-tool agent with escalation** — 3-4 MCP tool, agentic loop, structured error, PreToolUse hook, multi-concern decomposition.
2. **Claude Code team workflow** — project CLAUDE.md + rules + skill + MCP server + plan mode.
3. **Structured extraction pipeline** — JSON schema (nullable, enum+other), validation-retry, varied format few-shot, batch, human review routing.
4. **Multi-agent research** — coordinator `"Task"`, paralel spawn, claim-source mapping, structured error, conflict preservation.

Her egzersizi en az **bir kere** gerçekten yaz — kağıt üstünde bilmek yetmez.

---

## Son gün checklist
- [ ] 5 domain özet listesi okundu mu?
- [ ] 12 örnek soru + açıklamaları revize mi?
- [ ] 25 tuzağın hepsini tanıyor muyum?
- [ ] "Non-existent feature" distractor örnekleri aklımda mı?
- [ ] Pratik sınavda nereden yanıldığımı biliyor muyum?
- [ ] Sınava erken girme, zaman kalırsa işaretlediğim soruları gözden geçirme planı?

Başarılar. 720'yi kolay geçersin.
