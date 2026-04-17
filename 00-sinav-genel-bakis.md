# Claude Certified Architect – Foundations: Sınav Genel Bakış

## Sertifikanın Amacı
Bu sertifika, **Claude ile gerçek dünya çözümleri** üretirken doğru **trade-off** kararları verebilen pratisyenleri ölçer. Kavramsal bilgi yetmez; üretimde mimari, konfigürasyon ve güvenilirlik kararları verebilmelisin.

Kapsam dört teknoloji etrafında:
1. **Claude Agent SDK** — ajanik döngü, subagent, hook
2. **Claude Code** — CLAUDE.md, skills, slash commands, plan mode
3. **Claude API** — tool_use, tool_choice, stop_reason
4. **Model Context Protocol (MCP)** — tool/resource tasarımı, server scoping

## Hedef Aday Profili
- **6+ ay pratik** Claude deneyimi.
- Agentic uygulama inşa etmiş (multi-agent orchestration, subagent delegation, lifecycle hooks).
- Claude Code'u takım workflow'una yerleştirmiş (CLAUDE.md, Skills, MCP).
- Structured output ve prompt engineering tecrübesi var.
- Context window yönetimi ve CI/CD entegrasyonu yapabilir.

## Format ve Puanlama
| Özellik | Değer |
|---|---|
| Soru tipi | Çoktan seçmeli, 1 doğru + 3 distractor |
| Puan | 100–1000 arası scaled score |
| Geçme notu | **720** |
| Boş bırakma cezası | Yok — tahmin et |
| Sonuç | Pass/Fail |

## Alan Ağırlıkları
| # | Domain | Ağırlık |
|---|---|---|
| 1 | Agentic Architecture & Orchestration | **%27** |
| 2 | Tool Design & MCP Integration | **%18** |
| 3 | Claude Code Configuration & Workflows | **%20** |
| 4 | Prompt Engineering & Structured Output | **%20** |
| 5 | Context Management & Reliability | **%15** |

> **Strateji:** Domain 1 en ağır — oraya en fazla zamanı ayır. Domain 5 en hafif ama Context Management kavramları 1-4'ün tüm sorularında "gizlice" sorulur.

## Senaryo Yapısı
Sınavda **6 senaryodan 4'ü rastgele seçilir**. Her senaryo, birkaç soruyu kapsayan gerçekçi bir prodüksiyon bağlamı verir.

| # | Senaryo | Ana Domainler |
|---|---|---|
| 1 | Customer Support Resolution Agent | 1, 2, 5 |
| 2 | Code Generation with Claude Code | 3, 5 |
| 3 | Multi-Agent Research System | 1, 2, 5 |
| 4 | Developer Productivity with Claude | 2, 3, 1 |
| 5 | Claude Code for CI/CD | 3, 4 |
| 6 | Structured Data Extraction | 4, 5 |

## Kapsam Dışı (NOT on exam)
- Fine-tuning / model eğitimi
- API auth/billing/rate limiting
- MCP server deployment/infra
- Constitutional AI, RLHF detayları
- Embedding / vector DB implementasyonu
- Computer use, vision, streaming API
- Prompt caching implementasyon detayları

## Çalışma Sırası (Önerilen)
1. `01-domain1-agentic.md` — ajanik mimari (en ağır)
2. `02-domain2-tool-mcp.md` — tool tasarımı
3. `03-domain3-claude-code.md` — Claude Code workflows
4. `04-domain4-prompt-structured.md` — prompt & schema
5. `05-domain5-context-reliability.md` — context & escalation
6. `06-senaryolar.md` — 6 senaryo detayı
7. `07-ornek-sorular-analizi.md` — 12 örnek sorunun çözüm mantığı
8. `08-tuzaklar-ve-antipatterns.md` — sınavda sıkça tekrar eden yanılsamalar
9. `09-hazirlik-plani.md` — 4 hafta çalışma planı
