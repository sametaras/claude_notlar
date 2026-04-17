# Domain 4 — Prompt Engineering & Structured Output (%20)

6 task statement. Bu domain, "extraction + reliable output" mühendisliğinin özü.

---

## 4.1 Explicit Criteria ile Precision Yükseltmek

### Vague vs explicit
**Vague (yanlış):** "Be conservative", "only report high-confidence issues", "check that comments are accurate".

**Explicit (doğru):** "Flag comments only when the claimed behavior **contradicts** the actual code behavior (e.g., comment says 'returns null on error' but function throws)."

### Neden fark eder?
Vague talimatlar false positive oranını düşürmez. Yalnızca **kategorik** kriterler (hangi tip issue raporlanacak / atılacak) false positive azaltır.

### Yüksek false-positive kategorilerde strateji
1. Kategoriyi **geçici devre dışı bırak** (developer trust restore).
2. Prompt'u explicit criteria ile yeniden yaz.
3. Sample üzerinde test et → geri aç.

### Severity tanımı
"High / Medium / Low" demek yetmez. Her seviye için **concrete code example** ver:
- **High:** SQL injection, unvalidated user input in shell exec
- **Medium:** N+1 query, leaking connection
- **Low:** unused import, style nit

---

## 4.2 Few-Shot Prompting

### Ne zaman few-shot?
Detaylı prose talimat tutarsız sonuç üretiyorsa, few-shot **en etkili tekniktir**.

### Kullanım vakaları
- **Ambiguous tool selection**: 2-4 örnekle ajanın ikilem vakalarını nasıl çözeceğini göster.
- **Output format tutarlılığı**: `(location, issue, severity, suggested_fix)` şeklinde hep aynı şekilde.
- **False positive ayırımı**: acceptable pattern vs genuine issue karşılaştırması.
- **Varied document format**: inline citation vs bibliography, narrative vs table.

### Örnek sayısı
**2-4 örnek** sweet spot. Dokümantasyon bu rakamı özellikle vurguluyor. Az → generalization zayıf; çok → token yanar, over-fit edebilir.

### Kritik: Reasoning'i göster
Sadece input → output vermek değil. "Bu durumda X'i seçtik çünkü..." şeklinde **plausible alternatif** üzerinden mantık yürütmeyi göster. Model generalization'ı buradan öğrenir.

### Few-shot'un hallucination etkisi
Extraction task'larında (informal ölçüler, çeşitli doküman yapıları) few-shot **hallucination'ı azaltır**. Model "böyle bir vaka daha önce gördüm, bu şekilde handle ediliyor" der.

---

## 4.3 Structured Output: tool_use + JSON Schema

### Neden tool_use?
Schema-compliant output için **en güvenilir** yol. JSON syntax error'larını **eliminate** eder.

```json
{
  "name": "extract_invoice",
  "description": "Extract fields from invoice document",
  "input_schema": {
    "type": "object",
    "properties": {
      "invoice_number": {"type": "string"},
      "total_amount": {"type": "number"},
      "line_items": {"type": "array", "items": {...}},
      "currency": {"type": "string", "enum": ["USD","EUR","TRY","other"]},
      "currency_detail": {"type": ["string","null"]}
    },
    "required": ["invoice_number","total_amount"]
  }
}
```

### tool_choice matrisi
| Ayar | Davranış | Use-case |
|---|---|---|
| `"auto"` | Model text de dönebilir | Konversasyon |
| `"any"` | Bir tool **çağrılmak zorunda** | Çok schema var, doküman tipi bilinmiyor — mutlaka birine mapple |
| `{"type":"tool","name":"extract_metadata"}` | O tool **zorunlu** | Metadata extraction sonra enrichment çalışmalı |

### Ne eliminate eder, ne etmez
**Eder:** JSON syntax error, eksik bracket, yanlış tip.
**Etmez:** Semantic error — line_items toplamı total'a eşit değil, değer yanlış field'da, uydurulmuş değer.

### Hallucination önleyici schema design
- Opsiyonel field'ları **nullable** yap (`"type": ["string","null"]`). Model "var olmayan" bir veriyi uydurmak zorunda kalmasın.
- Extensible enum: `enum: ["invoice","receipt","quote","other"]` + ayrıca `type_detail` nullable string. "Other" dediğinde detayı açıklar.
- `"unclear"` enum value ekle → ambiguous kalan vakayı işaretlesin.

### Prompt + schema tandem
Schema strict olsa da **format normalization kuralları** prompt'ta kalmalı: "Tarihleri ISO 8601'e çevir", "para birimi sembolünü atla, sadece kod kullan".

---

## 4.4 Validation, Retry ve Feedback Loop

### Retry-with-error-feedback
Schema validation fail ettiğinde **yeni bir model çağrısı** yap; prompt'a şunları ekle:
1. Orijinal döküman
2. Başarısız extraction (JSON)
3. **Spesifik** validation error mesajı

Model self-correct eder.

### Retry ne zaman işe yarar, ne zaman yaramaz?
| Hata tipi | Retry sonucu |
|---|---|
| Format mismatch (date string yerine ISO istendi) | İşe yarar |
| Structural error (required field boş geldi ama dokümanda var) | İşe yarar |
| Information simply absent (field dokümanda hiç yok) | **Yaramaz** — fabrikasyon riski, nullable yap |

### Feedback loop (systemic analysis)
Her finding'e `detected_pattern` field'ı ekle. Developer dismiss ettiklerinde **hangi kod pattern'larının** yanlış flag aldığı ortaya çıkar → prompt'u o pattern'a karşı iyileştir.

### Self-validation hileleri
- `calculated_total` + `stated_total` ikisini birden çıkart, eşleşmiyorsa flag.
- `conflict_detected: bool` — source içi çelişki varsa model işaretler.

---

## 4.5 Batch Processing Stratejisi

### Message Batches API — olgular
| Özellik | Değer |
|---|---|
| Maliyet | **%50** daha ucuz |
| İşleme penceresi | **24 saate kadar** |
| Latency SLA | **Yok** |
| Multi-turn tool call | **Desteklemez** (mid-request tool execute yok) |
| Request/response correlation | `custom_id` field |

### Uygun / uygun değil
| Workflow | Batch OK? |
|---|---|
| Overnight technical debt raporu | ✅ |
| Weekly audit | ✅ |
| Nightly test generation | ✅ |
| **Pre-merge blocking check** | ❌ (developer bekliyor) |
| Live chat agent | ❌ |

### Sample Q11 refleksi
"Her iki workflow'u batch'e geçirelim %50 tasarruf için" → **Yanlış.** Blocking'i synchronous bırak, non-blocking'i batch'e çek.

### SLA hesabı
Örneğin 30-saatlik SLA'nız var, batch 24 saate kadar çalışabiliyor → **4 saatlik pencerelerde** submit et → worst case 4 + 24 = 28 saat.

### Batch failure handling
`custom_id` ile failed document'ları tespit et, yalnızca onları resubmit et. Eğer oversized ise chunk'la.

### Pre-flight: prompt refinement
Büyük batch'ten önce **küçük sample**'da prompt'u optimize et. First-pass success rate düşükse iteratif resubmit maliyetli.

---

## 4.6 Multi-Instance ve Multi-Pass Review

### Self-review limit
Model, kod üretirken kullandığı reasoning context'i taşır → kendi kararlarını sorgulamak konusunda zayıf. **Extended thinking** de bunu çözmez.

### Independent review instance
İkinci bir Claude instance, generator'ın reasoning'ini görmeden review yapar → subtle bug'lar için belirgin biçimde daha etkili.

### Multi-pass review (hatırlatma)
Büyük multi-file PR → per-file local pass + cross-file integration pass. Sample Q12 bunun üzerine.

### Confidence reporting
Her finding'in yanında self-reported confidence → kalibre edilmiş review routing. Düşük confidence → human. Yüksek + critical → otomatik.

---

## Domain 4 — Özet Çalışma Listesi
- [ ] Explicit criteria > "be conservative".
- [ ] Few-shot: 2-4 örnek, reasoning göster.
- [ ] tool_use + JSON schema = syntax error eliminasyonu.
- [ ] tool_choice: auto / any / forced.
- [ ] Nullable field = hallucination önleyici.
- [ ] Retry ancak format/structural hatalarda işe yarar.
- [ ] Batch API: %50 ucuz, 24h, pre-merge ❌, overnight ✅.
- [ ] custom_id ile failure tracking.
- [ ] Independent instance review > self-review.
