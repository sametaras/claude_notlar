# Claude Certified Architect – Foundations | Türkçe Çalışma Notları

Anthropic'in **Claude Certified Architect – Foundations** sertifika sınavı için hazırlanmış kapsamlı Türkçe çalışma materyali.

> **Samet Aras** tarafından, [bazenda.com](https://bazenda.com)'u Claude Code ile public'e açan projeden sonra, sertifika hazırlığı sürecinde tutulan ders notları. Herkesin faydalanabilmesi için açık.

---

## İçerik

| Dosya | Konu |
|---|---|
| [`egitim/00-sinav-genel-bakis.md`](00-sinav-genel-bakis.md) | Sınav formatı, ağırlıklar, senaryo yapısı, kapsam dışı konular |
| [`egitim/01-domain1-agentic.md`](01-domain1-agentic.md) | **Domain 1** — Agentic Architecture & Orchestration (%27) |
| [`egitim/02-domain2-tool-mcp.md`](02-domain2-tool-mcp.md) | **Domain 2** — Tool Design & MCP Integration (%18) |
| [`egitim/03-domain3-claude-code.md`](03-domain3-claude-code.md) | **Domain 3** — Claude Code Configuration & Workflows (%20) |
| [`egitim/04-domain4-prompt-structured.md`](04-domain4-prompt-structured.md) | **Domain 4** — Prompt Engineering & Structured Output (%20) |
| [`egitim/05-domain5-context-reliability.md`](05-domain5-context-reliability.md) | **Domain 5** — Context Management & Reliability (%15) |
| [`egitim/06-senaryolar.md`](06-senaryolar.md) | 6 resmi senaryonun detaylı analizi |
| [`egitim/07-ornek-sorular-analizi.md`](07-ornek-sorular-analizi.md) | 12 resmi örnek sorunun çözüm mantığı |
| [`egitim/08-tuzaklar-ve-antipatterns.md`](08-tuzaklar-ve-antipatterns.md) | 25 klasik sınav tuzağı ve anti-pattern |
| [`egitim/09-hazirlik-plani.md`](09-hazirlik-plani.md) | 4 haftalık günlük hazırlık planı |
| [`egitim/10-pratik-test.md`](10-pratik-test.md) | 25 soruluk pratik sınav + cevap anahtarı |

---

## Kimin İçin?
- Claude Agent SDK, Claude Code, Claude API veya MCP ile **6+ aydır** prodüksiyon uygulaması geliştiriyorsan.
- **Claude Certified Architect – Foundations** sınavına hazırlanıyorsan.
- LLM-based ajanik mimariyi Türkçe kaynakla pekiştirmek istiyorsan.

---

## Nereden Başlamalı?

1. Önce [`00-sinav-genel-bakis.md`](00-sinav-genel-bakis.md) ile sınav haritasını çıkar.
2. Domain sırayla `01 → 05`.
3. [`06-senaryolar.md`](06-senaryolar.md) ile senaryoya özgü tuzakları öğren.
4. [`07-ornek-sorular-analizi.md`](07-ornek-sorular-analizi.md) ile her distractor'ın **neden** yanlış olduğunu öğren.
5. [`08-tuzaklar-ve-antipatterns.md`](08-tuzaklar-ve-antipatterns.md) — 25 klasik hatayı içselleştir.
6. Son hafta [`10-pratik-test.md`](10-pratik-test.md) ile zaman tutarak dene.
7. Zaman planı için [`09-hazirlik-plani.md`](09-hazirlik-plani.md).

---

## Sınav Özeti (Hızlı Referans)

**Format:** Çoktan seçmeli, 1 doğru + 3 distractor.
**Puan:** 100–1000 scaled score; geçme **720**.
**Senaryo:** 6 resmi senaryodan 4'ü rastgele.

**Domain ağırlıkları:**
- Domain 1 — Agentic Architecture & Orchestration: **%27**
- Domain 2 — Tool Design & MCP Integration: **%18**
- Domain 3 — Claude Code Configuration & Workflows: **%20**
- Domain 4 — Prompt Engineering & Structured Output: **%20**
- Domain 5 — Context Management & Reliability: **%15**

---

## Sınavın 3 Altın Kuralı

1. **Deterministik vs probabilistik:** Kritik compliance → hook/prerequisite/schema. Yumuşak yönlendirme → prompt.
2. **Root cause > semptom:** Distractor'ların çoğu semptom tedavisi önerir.
3. **Proportionate response:** Prompt iteration denenmeden ML/classifier/routing layer = over-engineering.

---

## Katkı ve Geri Bildirim

Bu notlar canlı bir doküman. Hatalı bir şey gördüysen veya eklemek istediğin bir şey varsa issue aç veya PR gönder.

Sınav değerlendirmesi Feb 10 2025 Version 0.1 exam guide baz alınarak hazırlanmıştır. Anthropic resmi dokümantasyonu güncellendikçe burada da güncelleme yapılır.

---

## Yasal

Bu repo **üçüncü taraf** bir çalışma materyalidir. Anthropic, PBC veya Claude ile resmi bir bağlantısı **yoktur**. Sınav ücreti, kayıt ve resmi dokümantasyon için Anthropic resmi kanallarını kullan.

Bu repo'daki özetler kamuya açık Anthropic exam guide dokümanı kapsamında yeniden ifade edilmiş bilgilerdir.

---

## Yazar

**Samet Aras**
- [bazenda.com](https://bazenda.com) — Claude Code ile geliştirilip public'e açılan proje
- LinkedIn üzerinden sertifikasyon sürecini paylaşıyor

Çalışma notları açık kaynak; faydalı bulursan **star** ver, paylaş. Başarılar.
