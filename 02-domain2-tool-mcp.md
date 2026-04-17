# Domain 2 — Tool Design & MCP Integration (%18)

5 task statement. Tool tasarımı kalitesi, ajan güvenilirliğinin **birincil kaldıracı**dır.

---

## 2.1 Effective Tool Interfaces

### Anahtar fikir
**Tool description'ları, LLM'in tool seçiminde kullandığı birincil mekanizmadır.** Minimal description → güvenilmez seçim.

### İyi bir description içerir
- Tool'un amacı (tek cümleyle net)
- Girdi formatı
- Örnek sorgular
- Edge case'ler
- Benzer alternatiflerden **farkı** (boundary explanation)

### Canonical hata (Sample Q2)
- `get_customer`: "Retrieves customer information"
- `lookup_order`: "Retrieves order details"
→ Model "check my order #12345" sorgusunda `get_customer` çağırıyor. İlk çözüm: **description'ları genişlet**. Few-shot eklemek daha az leveraged. Routing layer over-engineered.

### Refactor kalıpları
- `analyze_content` + `analyze_document` gibi yakın tool'ları **yeniden isimlendir**: `extract_web_results` vs `analyze_document`.
- Generic tool'u purpose-specific'e böl: `analyze_document` → `extract_data_points`, `summarize_content`, `verify_claim_against_source`.
- System prompt'ta yazan anahtar kelimeler de seçimi etkiler — oradaki keyword-sensitive talimatları gözden geçir.

---

## 2.2 Structured Error Responses (MCP)

### isError flag
MCP, tool failure'ı agent'a bildirmek için **`isError`** flag'ini kullanır.

### Hata kategorileri
| Kategori | Örnek | Retry? |
|---|---|---|
| **transient** | timeout, 503 | Evet |
| **validation** | invalid input format | Input düzeltilirse evet |
| **business** | policy violation, refund limit | **Hayır** |
| **permission** | RBAC denial | Hayır |

### Neden structured?
Uniform "Operation failed" → agent hangi kurtarma stratejisini uygulayacağını bilemez. **Wasted retry** olur.

### Response şeması
```json
{
  "isError": true,
  "errorCategory": "business",
  "isRetryable": false,
  "message": "Refunds above $500 require human approval.",
  "detail": "..."
}
```

### Access failure vs valid empty result ayrımı
Çok önemli: `lookup_order("ORD-999")` sonuç bulmadıysa bu **başarılı bir sorgu**, hata değil. Timeout ise failure. Aynı response yapısıyla ikisini birleştirmek agent'ı yanıltır.

### Subagent local recovery
Transient failure'ları subagent **kendi içinde** kurtarsın. Coordinator'a yalnızca çözemediklerini **partial results ve ne denendiği** ile birlikte propagate etsin.

---

## 2.3 Tool Distribution & tool_choice

### Principle of least privilege
Bir agent'a **18 tool vermek** ≠ 4-5 tool vermekten daha iyi. Aksine:
- Decision complexity artar
- Yanlış-specialization tool kullanımı yaygınlaşır
- Tool selection güvenilirliği düşer

### Scoped cross-role tools
Synthesis agent'a tüm web search tool'larını verme; ama %85 vakanın basit fact-check olduğu ölçüldüyse dar bir `verify_fact` tool'u ver. Karmaşık vakalar yine coordinator'dan geçsin. (Sample Q9)

### tool_choice konfigürasyonu
| Değer | Davranış | Ne zaman |
|---|---|---|
| `"auto"` | Model ister tool çağırır ister text döner | Default konuşma |
| `"any"` | Model **bir** tool **çağırmak zorunda**, hangisi serbest | Structured output garantisi isteniyorsa |
| `{"type": "tool", "name": "X"}` | Tam olarak X çağırılır | Zorunlu ilk adım (ör. `extract_metadata` önce, sonra enrichment) |

---

## 2.4 MCP Server Integration

### Scoping
| Scope | Dosya | Paylaşım |
|---|---|---|
| **Project-level** | `.mcp.json` | Version control ile takım paylaşır |
| **User-level** | `~/.claude.json` | Kişisel, paylaşılmaz |

### Credential management
`.mcp.json` içinde `${GITHUB_TOKEN}` gibi env var expansion — secret'ları repo'ya commit etmeden kullanabilirsin.

### Simultaneous access
Configure edilmiş tüm MCP server'ların tool'ları connection time'da keşfedilir ve agent'a **aynı anda** expose edilir.

### Description quality (önemli!)
Zayıf MCP tool description yazarsan, agent built-in Grep'i MCP'den daha capable tool yerine tercih edebilir. MCP tool description'larını **kapsamlı** yaz.

### MCP Resources
Tool'dan farklı olarak, **resource** bir content catalog'tur:
- Issue özet listeleri
- Documentation hiyerarşileri
- Database schema'ları

Resource expose etmek, agent'ın "discovery call"'larını azaltır — verinin var olduğunu görür, körlemesine aramaz.

### Build vs adopt
Standart entegrasyon (Jira, GitHub, Slack) için **community MCP server** kullan. Custom server'ı takıma özgü workflow'lar için sakla.

---

## 2.5 Built-in Tools: Read / Write / Edit / Bash / Grep / Glob

### Seçim matrisi
| Amaç | Tool |
|---|---|
| Dosya **içeriğinde** pattern ara (function name, error message, import) | **Grep** |
| Dosya **yolunda** pattern match (`**/*.test.tsx`) | **Glob** |
| Tüm dosyayı oku | **Read** |
| Dosyayı sıfırdan yaz/override | **Write** |
| Targeted modifikasyon (unique text match) | **Edit** |
| Shell komut | **Bash** |

### Edit → Write fallback
Edit'in bulduğu `old_string` dosyada **unique değilse** fail eder. O zaman Read + Write ile düzelt.

### İncremental exploration kalıbı
1. **Grep**: entry point bul.
2. **Read**: import'ları izle, flow trace et.
3. Her dosyayı baştan okumaya kalkma — attention bütçesi yanar.

### Wrapper trace pattern
Fonksiyon kullanımını wrapper modüller arasında takip ederken:
1. Önce tüm exported isimleri topla.
2. Sonra her ismi codebase boyunca ara.

---

## Domain 2 — Özet Çalışma Listesi
- [ ] Tool description = tool selection'ın birincil sinyali.
- [ ] `isError` + `errorCategory` + `isRetryable` kombinasyonu.
- [ ] 4 hata kategorisi: transient / validation / business / permission.
- [ ] Access failure ≠ valid empty result.
- [ ] Subagent local recovery, coordinator'a sadece çözülmeyenler.
- [ ] `tool_choice`: auto / any / forced.
- [ ] `.mcp.json` (proje) vs `~/.claude.json` (kişisel).
- [ ] Env var expansion = `${VAR_NAME}` credential için.
- [ ] MCP resource = content catalog; MCP tool = action.
- [ ] Grep (content) vs Glob (path) ayrımı.
