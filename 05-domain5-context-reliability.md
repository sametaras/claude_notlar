# Domain 5 — Context Management & Reliability (%15)

En hafif ağırlıklı domain ama **soruların geneline** yedirilir. 6 task statement.

---

## 5.1 Uzun Etkileşimde Context Yönetimi

### Progressive summarization tehlikesi
Uzun konuşma özetlendikçe şu kaybolur:
- Sayısal değerler (amount, %)
- Tarihler
- Order ID, customer ID gibi transactional fact'lar
- Müşterinin **kendi sözleriyle** ifade ettiği beklenti

Özet "müşteri iade istiyor, bir sorun var" der geçer → geri dönüşte milyon dolarlık detay eksik.

### Çözüm: "Case facts" block
Transactional fact'ları ayrı bir **persistent block** halinde tut, her prompt'a gömülür:
```
<case_facts>
  customer_id: CUST-8421
  order_id: ORD-9932
  refund_amount: $142.50
  customer_request: "full refund + free return shipping due to 3rd damaged delivery"
  policy_citation: "Section 4.2 multiple-damage exemption"
</case_facts>
```

Summarize edilmiş geçmiş bunun **dışında** yaşar.

### Lost in the middle
Modeller **başlangıç ve bitiş**'teki bilgiyi güvenilir işler, **orta** kısımda bulgular kaybolabilir.

**Önlem:**
- Key findings özetini girdinin **başına** koy.
- Detaylı sonuçları **explicit section header**'larla organize et.
- Kritik info'yu asla sadece ortada bırakma.

### Tool output trimming
40+ field dönen order lookup'tan yalnızca **iadeye ilgili 5 field'ı** tut. Modele ulaşmadan önce tool result'ı kırp (PostToolUse hook'un başka bir kullanım vakası).

### Multi-issue session
Birden çok konu (order, billing, subscription) paralel işleniyorsa → her konu için **ayrı structured issue block**'u. Karışmasın.

### API request'lerde tam history
Sonraki API çağrılarında **tüm konuşma geçmişini** pass et — yoksa konuşma tutarlılığı bozulur. (Claude stateless.)

---

## 5.2 Escalation ve Ambiguity Resolution

### Geçerli escalation trigger'ları
1. **Müşteri açıkça insan istiyorsa** → hemen escalate, önce investigate etme.
2. **Policy exception / gap** → kural belirsiz ya da sessiz olduğu vakalar.
3. **Meaningful progress yapamamak** → araçlar yetmedi.

### Geçersiz escalation trigger'ları (sınavda distractor)
- ❌ **Sentiment analysis** ile frustre müşteriyi escalate etmek. Sentiment ≠ complexity.
- ❌ **Self-reported confidence score** — LLM kendi confidence'ını kötü kalibre eder.
- ❌ "Complex case" → explicit kriterler olmadan subjektif.

### Yumuşak vs sert kural
| Durum | Aksiyon |
|---|---|
| "Speak to a human!" | Anında escalate |
| Frustrasyon + basit çözülebilir konu | **Frustrasyonu kabul et + çözüm öner**. Müşteri hâlâ human isterse escalate. |

### Policy ambiguity
Kural "kendi sitemizde fiyat farkı" diyor, müşteri **competitor price match** istiyor → policy silent. Autonomous karar verme, **escalate**.

### Multiple customer matches
`lookup_customer("John Smith")` 3 sonuç döndürdü → heuristic ile seçme (en son kayıt, en aktif vs). **Extra identifier iste** (ZIP, son 4 hane, doğum tarihi).

---

## 5.3 Multi-Agent Error Propagation

### Structured error context
Subagent fail ettiğinde coordinator'a şu bilgileri ver:
- **failure_type** (timeout, rate limit, not found, invalid query)
- **attempted_query** (ne denendi)
- **partial_results** (elde edilebilen kısmı)
- **alternatives** (önerilen farklı yaklaşımlar)

Bu sayede coordinator intelligent recovery yapar.

### Anti-patterns
- ❌ Generic "search unavailable" → coordinator context'siz kalır.
- ❌ Empty result'ı success gibi döndürmek (error suppression) → sessiz bozukluk.
- ❌ Tek failure'da tüm workflow'u terminate etmek → recovery şansı yok.

### Access failure vs empty valid result (yine!)
| Durum | Nasıl raporla |
|---|---|
| Timeout | error + retry önerisi |
| Invalid input | error + correction mümkün mü |
| Geçerli sorgu, 0 sonuç | **success** + empty |

### Local recovery — global propagation
Subagent kendi seviyesinde transient failure'ı retry etsin. Yalnızca çözemediklerini coordinator'a yukarı ver — **attempted + partial** ile birlikte.

### Coverage annotation
Synthesis çıktısında: "Finding X well-supported, Topic Y **gap due to unavailable source**" şeklinde coverage işareti. Sessiz atlamak yerine açıkça belirt.

---

## 5.4 Büyük Codebase Exploration'da Context

### Context degradation sinyalleri
Uzun session'da:
- Model "typical patterns" demeye başlar (specific class ismini unutmuş)
- Tutarsız cevaplar (aynı soruya önce X, sonra Y)
- Daha önce bulunmuş fact'ları reinvent etmeye çalışır

### Scratchpad files
Agent'a **dosyaya yazdır**:
- `.claude/scratch/findings.md` → key findings
- Sonraki sorgularda scratchpad'i referans et
Context shrinks etse de bilgi kalıcıdır.

### Subagent delegation (Explore pattern)
Main agent coordinator kalsın. **Verbose exploration**'ı subagent'a yaptır, sadece özet dönsün. Main conversation context'ini koru.

### Crash recovery
Her agent state'ini bilinen bir lokasyona **export etsin** (manifest). Coordinator resume'da manifest'i yükleyip agent prompt'larına inject eder. Full restart olmadan kalın yerden devam.

### /compact
Uzun session'da context dolunca kullan. Verbose discovery output'u özete çevirir, kritik fact'lar korunur.

---

## 5.5 Human Review ve Confidence Kalibrasyonu

### Aggregate metric yanılgısı
"%97 overall accuracy" → harika görünür. Ama breakdown'da:
- Document type A: %99
- Document type B: %88
- Field X: %70

Stratify etmeden automate etmek belirli segmentte felaket üretir.

### Stratified random sampling
High-confidence extraction'lardan **her doküman tipi / field için** stratified örnek çek. Error rate'i ölç. Novel pattern yakalamanın da tek yolu.

### Field-level confidence
Model her field için ayrı confidence döndürsün (fine-grained). **Labeled validation set** ile threshold'ları kalibre et.

### Routing stratejisi
```
if model_confidence < 0.7 OR source_is_ambiguous OR conflict_detected:
    route_to_human()
else:
    auto_process()
```

Limited reviewer kapasitesini **prioritize** et.

---

## 5.6 Information Provenance (Multi-Source Synthesis)

### Attribution kaybı
Summarize ederken claim-source mapping'i **preserve etmelisin**. Yoksa "bu rakam nereden geldi?" sorusu cevapsız kalır.

### Structured claim-source mapping
Her subagent finding'i:
```json
{
  "claim": "US electric vehicle sales grew 47% YoY in 2023",
  "excerpt": "EVs accounted for 8.1% of new car sales...",
  "source_url": "https://...",
  "document_name": "IEA Global EV Outlook 2024",
  "publication_date": "2024-04-23"
}
```

Synthesis agent bu mapping'i **birleştirir ama korur**.

### Conflicting values
İki credible source farklı rakam veriyorsa **birini seçme**. Her ikisini attribute ederek raporla:
> "Market size estimated at $42B (Gartner, 2023) or $38B (IDC, 2023)."

### Temporal data
Publication date / collection date'i zorunlu field yap. "2019 veri vs 2024 veri" çelişki değil, **temporal fark**. Zaman bilgisi olmadan yanlış interpretasyon çıkar.

### Report strüktürü
- **Well-established findings** bölümü (consensus)
- **Contested findings** bölümü (source çelişkisi, attribute edilmiş)
- **Methodological context** (veri nasıl toplanmış)

### Content-type aware rendering
Finansal data → tablo. News → prose. Technical → structured list. Her şeyi uniform format'a zorlamak bilgiyi bozar.

---

## Domain 5 — Özet Çalışma Listesi
- [ ] Case facts block → summarize'dan ayrı.
- [ ] Lost in the middle → kritik bilgi başta/sonda.
- [ ] Tool output trimming → relevant field only.
- [ ] Escalation: human request / policy gap / no progress.
- [ ] Sentiment ve self-confidence YANLIŞ trigger.
- [ ] Structured error context = 4 field (type, attempted, partial, alternatives).
- [ ] Access failure ≠ empty valid result (3. tekrar).
- [ ] Scratchpad file = context degradation silahı.
- [ ] Stratified sampling → aggregate yanılgısını kırar.
- [ ] Claim-source mapping = attribution preservation.
- [ ] Temporal data = conflict değil, fark.
