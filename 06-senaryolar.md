# Senaryolar — Detaylı İnceleme

Sınavda 6 senaryodan **4'ü rastgele** çıkar. Her senaryonun konseptsel haritasını çıkarıyoruz: hangi tuzaklara dikkat, hangi kalıplara güven.

---

## Senaryo 1: Customer Support Resolution Agent

**Bağlam:** Claude Agent SDK ile kurulu customer support. Belirsiz (high-ambiguity) istekler: iade, billing dispute, account issue. Custom MCP tool'lar: `get_customer`, `lookup_order`, `process_refund`, `escalate_to_human`. Hedef: **%80+ first-contact resolution** + doğru escalation.

**Ana domainler:** 1 (Agentic), 2 (Tool/MCP), 5 (Context/Reliability).

### Sınavda karşılaşacağın kalıplar
| Problem | Doğru pattern |
|---|---|
| `get_customer` atlanıyor, yanlış müşteriye refund | **Programmatic prerequisite** (hook) |
| Benzer tool description'lar → yanlış seçim | Description'ları **genişlet** (input formatları, boundary) |
| Ajan basit vakaları escalate, karmaşık vakaları üstleniyor | **Explicit escalation criteria** + few-shot |
| $500 üstü refund'u engelle | **PreToolUse hook** |
| Transient timeout vs permanent error ayırımı yok | `errorCategory` + `isRetryable` |
| Multiple customer match | Extra identifier iste — heuristic ile seçme |
| Human'a devrederken transcript yok | Structured handoff (customer_id, root_cause, attempted, recommendation) |

### Trigger kelimeler → doğru refleks
- "guaranteed", "policy", "financial" → hook
- "frustrated customer", "sentiment" → **dikkat**, sentiment escalation YANLIŞ
- "first-contact resolution below target" → escalation criteria problemi
- "misidentification" → prerequisite problemi

---

## Senaryo 2: Code Generation with Claude Code

**Bağlam:** Claude Code ile kod generation, refactoring, debug, docs. Custom slash command'lar, CLAUDE.md config, plan mode vs direct.

**Ana domainler:** 3 (Claude Code), 5 (Context).

### Sınavda karşılaşacağın kalıplar
| Problem | Doğru pattern |
|---|---|
| Takım için `/review` command lazım | `.claude/commands/` (project scope) |
| Monolith → microservice restructure | **Plan mode** |
| Single-file bug fix | **Direct execution** |
| Convention dosyalar her yere yayılmış (test file'lar) | `.claude/rules/` + glob pattern |
| Yeni üye talimat almıyor | Config **user-level**'da; project-level'a taşı |
| Uzun exploration → context degrade | Scratchpad file + Explore subagent |
| Monolitik CLAUDE.md bakımı zor | `@import` ile modüler / `.claude/rules/` topic-specific |

### Trigger kelimeler
- "version control'le paylaşılsın" → project scope
- "mimari karar", "birçok dosya" → plan mode
- "convention'lar çok dizine yayılmış" → path-based rule (glob)
- "skill verbose output üretiyor" → `context: fork`

---

## Senaryo 3: Multi-Agent Research System

**Bağlam:** Coordinator + subagent'lar: web search, document analysis, synthesis, report generator. Comprehensive cited report üretir.

**Ana domainler:** 1, 2, 5.

### Sınavda karşılaşacağın kalıplar
| Problem | Doğru pattern |
|---|---|
| Final rapor tüm topic'i kapsamıyor | Coordinator'un decomposition'ı dar → **iterative refinement** + genişlet |
| Web search timeout | **Structured error** context (type, attempted, partial, alternatives) |
| Synthesis sık fact-check istiyor, round trip'ler çok | Scoped `verify_fact` tool synthesis'e ver |
| Paralel subagent istemine rağmen sıralı | Tek turn'de **birden çok Task call** emit |
| Subagent context inherit etmiyor (doğru!) | Findings'i prompt'a **açıkça** koy |
| Citation kayboluyor | Structured **claim-source mapping**, synthesis preserve |
| Çelişkili kaynaklar | Attribute ederek **her ikisini** raporla |
| Farklı publication date → conflict zannedildi | Temporal metadata zorunlu |

### Trigger kelimeler
- "missing coverage" → decomposition dar
- "timeout / failure" + "how to propagate" → structured error
- "latency %40" + "coordinator trip" → scoped tool
- "source attribution" → claim-source mapping

---

## Senaryo 4: Developer Productivity with Claude

**Bağlam:** Legacy codebase exploration, boilerplate, otomasyon. Built-in tool'lar + MCP.

**Ana domainler:** 2, 3, 1.

### Kalıplar
| Problem | Doğru pattern |
|---|---|
| Built-in tool seçimi (Grep vs Glob) | İçerik araması = **Grep**; yol pattern = **Glob** |
| Edit fail ediyor (non-unique anchor) | **Read + Write** fallback |
| MCP tool'u Grep yüzünden tercih edilmiyor | MCP description'ı **kapsamlı** yaz |
| Kimlik için credential commit edilmesin | `.mcp.json` + `${ENV_VAR}` expansion |
| Takım + kişisel MCP server birlikte | Project `.mcp.json` + user `~/.claude.json` |
| Exhaustive upfront reading | Incremental Grep → Read → follow imports |
| Jira entegrasyonu | **Community MCP server** (custom yazma) |

### Trigger kelimeler
- "credential", "secret" → env var expansion
- "Edit failed, text not unique" → Read + Write
- "Agent Grep'i tercih ediyor" → MCP description zayıf

---

## Senaryo 5: Claude Code for CI/CD

**Bağlam:** Otomatik code review, test generation, PR feedback. False positive az, actionable feedback.

**Ana domainler:** 3, 4.

### Kalıplar
| Problem | Doğru pattern |
|---|---|
| CI job hang, Claude input bekliyor | `-p` / `--print` flag |
| Machine-parseable output | `--output-format json` + `--json-schema` |
| Duplicate PR yorumları (re-review) | Prior findings context, "yalnızca yeni/unresolved" |
| Duplicate test suggestion | Existing test file'ları context'e koy |
| Blocking pre-merge + nightly report | Pre-merge synchronous, nightly **Batch API** |
| False positive yüksek | **Explicit criteria** + few-shot (confidence-based filter değil) |
| 14 dosya PR review inconsistent | **Per-file pass + cross-file integration pass** |
| Generated code'u self-review | **Independent second instance** |
| CLAUDE.md CI'da faydalı mı? | Evet — testing standards, fixtures, criteria |

### Trigger kelimeler
- "hang", "interactive input" → `-p`
- "JSON output", "schema" → `--output-format json`
- "attention dilution", "contradictory findings" → multi-pass
- "batch cost savings for pre-merge" → **ikisi de batch YANLIŞ**

---

## Senaryo 6: Structured Data Extraction

**Bağlam:** Unstructured doc → structured JSON. Schema validation, edge case, downstream sistem.

**Ana domainler:** 4, 5.

### Kalıplar
| Problem | Doğru pattern |
|---|---|
| JSON syntax error'lar | `tool_use` + JSON schema |
| Fabrikasyon (dokümanda yok ama field dolu) | **Nullable** field |
| "other" kategorisi | Enum + detail string pattern |
| Validation fail | Retry with error feedback (format/structural hatalarda) |
| Info dokümanda hiç yok | Retry işe yaramaz → nullable + flag |
| 100 doküman overnight | **Batch API** (50% ucuz) |
| Blocking realtime | **Sync API** |
| Batch içinden failure | `custom_id` ile retarget |
| line_items toplam = total kontrolü | `calculated_total` + `stated_total`, fark → `conflict_detected` |
| %97 accuracy iddiası | **Stratified sample**, doc-type x field breakdown |
| Düşük confidence | Human review routing |
| Multi-source conflict | Attribute et, **seçme** |

### Trigger kelimeler
- "guaranteed schema" → tool_use
- "fabricating values" → nullable
- "overnight / weekly" → Batch API
- "%97 overall" → stratify trap
- "conflicting statistics" → attribute, don't pick

---

## Cross-scenario strateji
Bir scenario'ya "ait" bir kalıp başka scenario'da **analojiyle** çıkabilir. Örneğin:
- Escalation kriterleri (Senaryo 1) → Senaryo 6'da "human review routing"
- Structured error (Senaryo 3) → Senaryo 1'de MCP tool error
- Independent review (Senaryo 5) → Senaryo 6'da extraction validation

Soruyu çözerken senaryoyu değil, **sorulan ilkeyi** hedefle.
