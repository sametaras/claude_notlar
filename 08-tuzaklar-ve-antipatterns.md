# Tuzaklar ve Anti-Patterns

Sınav distractor'larının %80'i şu **tipik yanlışlar** ailesinden üretilir. Refleks kazanmak için her birini oku ve soruda bunu görür görmez hemen **eleme** yap.

---

## Tuzak 1: Deterministik yerine probabilistik
Prompt, few-shot, "be conservative" gibi yönlendirmeler **asla** guaranteed compliance vermez.

**Red flag kelimeler:** "financial", "policy", "must", "verification before", "guaranteed", "compliance".

**Doğru refleks:** Hook (PreToolUse blok veya PostToolUse normalize), programmatic prerequisite, tool_use + strict schema.

---

## Tuzak 2: Semptom tedavisi
Distractor bir katmanın üstüne yeni katman eklemeyi önerir.

**Örnekler:**
- Tool description zayıf → "routing layer yaz" ❌
- Tool description zayıf → "classifier eğit" ❌
- False positive yüksek → "3 run'da 2'si flag'lerse kabul et" ❌
- Coverage gap → "synthesis agent'a gap detection ekle" ❌

**Doğru refleks:** Root cause neydi? Onu çöz.

---

## Tuzak 3: Over-engineering
Prompt iteration denenmeden ML, classifier, custom framework.

**Red flag kelimeler:** "train a classifier", "deploy separate model", "build a routing layer".

**Doğru refleks:** En düşük efor + en yüksek impact seçeneği.

---

## Tuzak 4: Self-referential güvensizlik
Model kendi confidence'ını veya kendi output'unu güvenilir değerlendiremez.

**Yanlış sinyaller:**
- Self-reported confidence score ile escalation
- Generator'ın kendi kodunu review etmesi
- Extended thinking ile self-correction umudu

**Doğru refleks:** Independent second instance, labeled validation set ile calibration, stratified sampling.

---

## Tuzak 5: Sentiment ≠ complexity
Frustre müşteri vs karmaşık vaka birbirinden bağımsız.

**Yanlış:** "Negative sentiment > threshold → escalate."
**Doğru:** Explicit criteria (policy gap, explicit human request, inability to progress).

---

## Tuzak 6: Error suppression
Failure'ı empty success olarak dönmek sistemi **sessizce** bozar.

**Yanlış:** `except TimeoutError: return []` success flag'iyle.
**Doğru:** Structured error (type, attempted, partial, alternatives).

**Alt-trap:** **Access failure ≠ empty valid result**. 0 match bulunan bir sorgu başarılı; timeout failure. Aynı response şekli kullanma.

---

## Tuzak 7: Tam terminate
Tek bir subagent failure'ında workflow'un tamamını sonlandırmak.

**Doğru:** Coordinator partial result + coverage annotation ile devam.

---

## Tuzak 8: Context window fetişizmi
Büyük context window = attention quality zannı.

**Yanlış:** "Daha büyük modele geç, hepsi tek pass'te review olsun."
**Doğru:** Multi-pass split; context değil, **attention quality** sorunu.

---

## Tuzak 9: Stale resume
Dosyalar değiştiyse eski session'ı resume etmek.

**Doğru:** Fresh session + structured summary inject; veya resume + "şu dosyalar değişti" açıkça söyle.

---

## Tuzak 10: Tool inflation
Agent'a "her ihtimale karşı" 18 tool vermek.

**Sonuç:** Decision complexity artar, seçim güvenilirliği düşer, cross-specialization misuse.
**Doğru:** Role-scoped tool set; high-frequency cross-role için constrained tool.

---

## Tuzak 11: Automatic context inheritance yanılgısı
"Subagent, coordinator'ın history'sini otomatik alır" sanmak.

**Gerçek:** Subagent **isolated context**. Ne vereceksen **prompt'ta açıkça** ver.

---

## Tuzak 12: Task tool'u unutma
Coordinator'ın `allowedTools`'unda `"Task"` yoksa subagent spawn yapamaz.

**Sınav tuzağı:** "Neden subagent'lar çağrılmıyor?" sorusunun cevaplarından biri bu.

---

## Tuzak 13: Paralel sandığın sıralı
Coordinator paralel subagent istiyorsa **tek turn'de birden çok Task call** emit etmeli. Farklı turn'lere yayarsan sıralı.

---

## Tuzak 14: Progressive summarization kaybı
Uzun konuşmada tarih, miktar, sayı, order ID özet dışına çıkarılmazsa **kaybolur**.

**Doğru:** "Case facts" block'u ayrı persist et.

---

## Tuzak 15: Aggregate metric yanılgısı
"%97 overall accuracy" güvenilir sanılır.

**Gerçek:** Doc-type x field breakdown'da %70 segment olabilir. **Stratified sampling** zorunlu.

---

## Tuzak 16: Scope'u karıştırma
`~/.claude/` = kişisel. `.claude/` (repo) = proje.

Yeni üye proje talimatlarını almıyorsa → config user-level'da; **taşı**.

---

## Tuzak 17: Path rule vs directory CLAUDE.md
- Dizine bağlı convention → subdirectory CLAUDE.md.
- File-type'a bağlı + çok dizine yayılmış (test file'lar, config file'lar) → `.claude/rules/` + glob.

---

## Tuzak 18: Batch API her şeye
50% cost savings cazip. Ama:
- **Blocking pre-merge** asla batch'e.
- **24h window + SLA yok**.
- Multi-turn tool call yok.

---

## Tuzak 19: tool_use eliminates everything
`tool_use + JSON schema` **syntax error'ı** yok eder. **Semantic error'ı değil**:
- Line items toplamı total'ı tutmayabilir.
- Değer yanlış field'da olabilir.
- Fabrikasyon yine mümkün (nullable yapmazsan).

---

## Tuzak 20: Retry her hatayı çözer
- Format mismatch → retry OK.
- Structural / schema → retry OK.
- **Information absent from source** → retry **fabrikasyona** yol açar. Nullable + flag kullan.

---

## Tuzak 21: Claim-source kaybı
Summarize ederken claim'i kaynağından koparmak → "bu rakam nereden?" cevapsız.

**Doğru:** Structured claim-source mapping, synthesis boyunca koru.

---

## Tuzak 22: Temporal ≠ conflict
2019 data vs 2024 data çelişki **değil**, temporal fark. Publication date metadata zorunlu.

---

## Tuzak 23: "Non-existent feature" distractor
Sınav zaman zaman **var olmayan** flag / config önerir:
- `CLAUDE_HEADLESS` env var ❌
- `--batch` flag ❌
- `.claude/config.json` commands array ❌

Tanımadığın bir şey görürsen **şüphelen**, dokümante edilmiş alternatif var mı bak.

---

## Tuzak 24: Iteration cap'i birincil durdurma
Normal sonlandırma `stop_reason == "end_turn"`. Cap yalnızca safety için.

---

## Tuzak 25: Doğal dil ile loop sonlandırma
"Cevapta 'done' geçiyorsa çık" → asla.

---

## Kontrol listesi (sınav öncesi son kez oku)
- Deterministik gerekiyor mu? → hook / prerequisite / schema.
- Root cause'u çözüyor mu? → yoksa distractor.
- En düşük efor + en yüksek leverage mi? → over-engineering yok.
- Sentiment / self-confidence → ASLA escalation trigger.
- Subagent context → explicit.
- Paralel → tek turn.
- Error → structured.
- Stale tool result → fresh + summary.
- Batch → overnight OK, blocking ❌.
- Nullable field → hallucination azaltır.
- Retry → sadece format/structural.
- Stratified sampling → aggregate trap.
- "Non-existent" flag → şüphelen.
