# Domain 1 — Agentic Architecture & Orchestration (%27)

Sınavın en ağır domaini. 7 görev ifadesi (task statement) var. Her birini teker teker inceliyoruz.

---

## 1.1 Ajanik Döngü (Agentic Loop) Tasarımı

### Bilinmesi gerekenler
Bir **ajanik döngü** şu lifecycle'ı takip eder:
1. Modele request gönder.
2. **`stop_reason`**'ı incele.
3. `"tool_use"` ise: istenen tool'u çalıştır, sonucu conversation history'ye ekle, yeniden modele gönder.
4. `"end_turn"` ise: döngüyü sonlandır, yanıtı kullanıcıya ver.

Tool sonuçları conversation history'ye eklenir; model sonraki adımı buna dayanarak akıl yürütür.

**Model-driven vs pre-configured**: Hangi tool'un çağrılacağına Claude karar verir (context'e bakarak). Kod içinde if/else karar ağacı yazıp sıralı tool çağırma **agentic loop değildir** — pipeline'dır.

### Anti-patterns (sınavda distractor olarak çıkar)
- ❌ Doğal dil sinyalleriyle loop sonlandırma ("eğer modelin cevabında 'tamam' geçiyorsa dur").
- ❌ Arbitrary iteration cap'i **birincil** durdurma mekanizması yapma. (Güvenlik için üst limit koyabilirsin ama normal sonlandırma `end_turn` ile olmalı.)
- ❌ Assistant text content varsa bitti saymak — aynı turda hem text hem tool_use olabilir.

### Pseudo-code
```python
messages = [initial_user_message]
while True:
    response = claude.messages.create(model=..., messages=messages, tools=tools)
    messages.append({"role": "assistant", "content": response.content})
    if response.stop_reason == "end_turn":
        break
    if response.stop_reason == "tool_use":
        tool_results = [execute(tu) for tu in response.tool_uses]
        messages.append({"role": "user", "content": tool_results})
```

---

## 1.2 Multi-Agent: Coordinator-Subagent Pattern

### Hub-and-spoke mimari
Tek bir **coordinator** (hub) vardır. Tüm subagent iletişimi, hata yönetimi ve bilgi routing'i coordinator'dan geçer. Subagent'lar birbirleriyle doğrudan konuşmaz.

### Kritik gerçek
Subagent'lar **isolated context** ile çalışır. Coordinator'ın history'sini **otomatik almazlar**. Ne vereceksen prompt'ta açıkça vermelisin.

### Coordinator'ın rolleri
- Task decomposition
- Dynamic delegation (her sorgu için subagent seçimi — her seferinde tüm pipeline'ı koşturma!)
- Result aggregation
- İteratif refinement (synthesis'te boşluk varsa targeted query ile yeniden delegate)

### Yaygın hata
**Dar decomposition**: "creative industries" topic'i yalnızca "digital art, graphic design, photography" alt-görevlerine bölünürse müzik/edebiyat/film tamamen dışarıda kalır. Örnek soru 7'nin cevabı bu.

### Skills
- Coordinator'ı **quality criteria** ile yönlendir, adım-adım prosedür dikte etme → subagent adaptability artar.
- Research scope'u **partition** et, duplication minimum olsun.
- Iterative refinement loop: synthesis → gap detect → re-delegate → synthesize again.

---

## 1.3 Subagent Spawning ve Context Passing

### Task tool
Subagent spawn etmenin mekanizması **Task tool**'dur. Coordinator'ın `allowedTools` listesinde **`"Task"` mutlaka olmalı**, yoksa subagent çağıramaz.

### AgentDefinition
Her subagent tipinin:
- `description`
- `systemPrompt`
- `allowedTools` (kısıtlamalı)
tanımı olur.

### Paralel subagent
**Tek coordinator turn'ünde birden çok Task tool call** emit edilirse paralel spawn olur. Farklı turn'lere yayarsan sıralı olur, gecikme katlanır. (Örnek soru 9'la ilgili değil ama Exercise 4'te ölçülüyor.)

### Context passing best-practice
- Prior agent bulgularını **tamamen** subagent prompt'una koy (web search sonuçları, doc analysis outputları).
- **Structured data format** kullan: content + metadata ayrımı (source URL, doc adı, sayfa no) — attribution korunur.
- `fork_session` ile aynı baseline analizden divergent branch'ler aç.

---

## 1.4 Multi-step Workflows: Enforcement vs Guidance

### Programmatic enforcement vs prompt-based guidance
Kritik karar noktası:

| Gereksinim | Doğru araç |
|---|---|
| **Deterministik zorunluluk** (finansal, yasal, güvenlik) | Programmatic hook / prerequisite gate |
| Yumuşak yönlendirme | Prompt instruction |

Prompt-tabanlı talimatların **sıfır olmayan hata oranı** vardır. Para transferi/iade/kimlik doğrulama gibi kritik akışlarda prompt'a güvenmek yetersizdir.

### Canonical örnek (Sample Q1)
`get_customer` doğrulanmadan `process_refund` çağrılmasın isteniyorsa → **programmatic prerequisite** (Task 1.4 + 1.5). System prompt'a "zorunludur" yazmak %12 hata bıraktı.

### Structured handoff (escalation)
İnsan ajana devrederken conversation transcript yoksa, yapılandırılmış handoff ver:
- customer ID
- root cause
- attempted actions
- refund amount
- recommended action

---

## 1.5 Hooks: Tool Call Interception & Normalization

### İki ana hook kalıbı
| Hook | Amaç |
|---|---|
| **PostToolUse** | Tool result'ı **modele ulaşmadan önce** dönüştür/normalize et |
| **Tool call interception** (PreToolUse) | Outgoing tool call'u blokla, policy enforce et |

### Örnek kullanımlar
- **PostToolUse normalization**: Bir MCP Unix timestamp dönüyor, diğeri ISO 8601, üçüncüsü numeric status code. Hook tek formata çevirsin — agent downstream inconsistency yaşamasın.
- **PreToolUse policy**: `process_refund(amount=$750)` çağrısı $500 threshold'unu aşıyor → hook blokluyor, human escalation'a yönlendiriyor.

### Hook vs prompt kararı (tekrar)
Deterministik compliance şartsa → **hook**. Probabilistic yönlendirme yeterliyse → prompt.

---

## 1.6 Task Decomposition Stratejisi

### İki ana desen
| Desen | Ne zaman |
|---|---|
| **Fixed sequential pipeline (prompt chaining)** | Tahmin edilebilir, çok-yönlü review (ör. her dosya için analiz → cross-file pass) |
| **Dynamic / adaptive decomposition** | Açık-uçlu araştırma (ör. "eski codebase'e comprehensive test ekle") |

### Büyük code review (Sample Q12)
14 dosyalık PR tek pass'te review edilirse:
- Bazı dosyalara derin feedback, bazılarına yüzeysel (attention dilution)
- Aynı pattern bir yerde flag, başka yerde onay (contradictory findings)
**Çözüm:** per-file local analysis + ayrı cross-file integration pass. (Daha büyük context window fix **değil** — attention quality sorunu.)

### Adaptive decomposition
Legacy codebase'e test ekleme gibi işlerde:
1. Yapıyı haritala
2. Yüksek impact alanları belirle
3. Dependency keşfedildikçe **adapt eden** prioritized plan üret

---

## 1.7 Session State: Resumption ve Forking

### Araçlar
| Komut | Amaç |
|---|---|
| `--resume <session-name>` | Belirli bir named session'a devam et |
| `fork_session` | Ortak analiz baseline'ından bağımsız branch aç |

### Kullanım kararı
- **Resume**: Prior context hâlâ büyük ölçüde geçerliyse.
- **Start fresh + structured summary**: Tool result'lar bayatladıysa — stale context ile devam etmekten güvenilirdir.
- Resume ediyorsan: agent'a **o sırada değişen dosyaları söyle**, targeted re-analysis yapsın.

### Fork kullanım vakası
Aynı codebase analizinden iki farklı testing stratejisini / refactoring yaklaşımını paralel denemek.

---

## Domain 1 — Özet Çalışma Listesi
- [ ] `stop_reason` lifecycle'ını ezbere bil.
- [ ] Coordinator'ın `allowedTools`'unda **Task** olmak zorunda.
- [ ] Subagent isolated context: prompt'a açıkça ver.
- [ ] Paralel subagent = tek turn'de birden çok Task call.
- [ ] Deterministik = hook; yumuşak = prompt.
- [ ] Dar decomposition → topic coverage gap.
- [ ] Büyük review → file-by-file + cross-file integration pass.
- [ ] Fork_session ile divergent exploration.
