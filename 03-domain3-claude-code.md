# Domain 3 — Claude Code Configuration & Workflows (%20)

6 task statement. Claude Code'un günlük geliştirici workflow'una gömülmesi.

---

## 3.1 CLAUDE.md Hiyerarşisi

### Üç seviye
| Seviye | Konum | Kapsam |
|---|---|---|
| **User** | `~/.claude/CLAUDE.md` | Sadece o kullanıcı — version control'de **yok** |
| **Project** | `.claude/CLAUDE.md` veya root `CLAUDE.md` | Tüm takım (repo'da) |
| **Directory** | Subdirectory içindeki `CLAUDE.md` | O dizine özgü konvansiyonlar |

### Klasik hata
Yeni takım üyesi belirli talimatları almıyor → bakılırsa konfigürasyon **user-level**'da. Proje için olanı **project-level**'a taşı.

### @import syntax
`CLAUDE.md` içinde `@import ./standards/api.md` gibi — monolitik CLAUDE.md yerine modüler.

### .claude/rules/ dizini
CLAUDE.md'yi patlatmayalım diye **topic-specific** rule dosyaları: `testing.md`, `api-conventions.md`, `deployment.md`.

### /memory komutu
Hangi memory dosyalarının yüklendiğini görmek ve tutarsız davranışı debug etmek için.

---

## 3.2 Custom Slash Commands & Skills

### Slash commands
| Scope | Konum |
|---|---|
| **Project** | `.claude/commands/` — repo'da, takım kullanır |
| **User** | `~/.claude/commands/` — kişisel |

**Sample Q4 canonical'i**: `/review` komutunu tüm takıma dağıtmak → `.claude/commands/` (A şıkkı).

### Skills (`.claude/skills/SKILL.md`)
Frontmatter opsiyonları:
- `context: fork` → skill **izole subagent context**'te koşar, main session'ı kirletmez (verbose output skill'leri için kritik)
- `allowed-tools` → skill sırasında hangi tool'lara izin var (ör. sadece file write)
- `argument-hint` → skill argümansız çağrılırsa developer'a hangi parametrelerin istendiğini gösterir

### Personal skill variant
`~/.claude/skills/` içinde **farklı bir isimle** kişisel variant yarat, takım skill'ini override etmeden kullan.

### Skill vs CLAUDE.md kararı
| Seçim | Ne zaman |
|---|---|
| **Skill** | On-demand, task-specific workflow |
| **CLAUDE.md** | Her zaman yüklenmesi gereken universal standart |

---

## 3.3 Path-Specific Rules (Koşullu Konvansiyon)

### .claude/rules/*.md + YAML frontmatter
```yaml
---
paths: ["**/*.test.tsx"]
---
Test dosyaları için konvansiyonlar...
```

### Ne zaman path rules, ne zaman subdirectory CLAUDE.md?
- Konvansiyonlar **bir dizine** lokalize → subdirectory CLAUDE.md
- Konvansiyonlar **bir dosya tipine** uygulanıyor ama dosyalar **çok dizine yayılmış** → `.claude/rules/` + glob pattern

**Sample Q6 canonical'i**: Test dosyaları `Button.test.tsx` kendi komponent'inin yanında, her yere serpilmiş → glob rule tek doğru (A). Subdirectory CLAUDE.md (D) yanlış çünkü dosyalar dizine bağlı değil.

### Avantajı
Rule **yalnızca** matching dosyalar düzenlenirken yüklenir → token tasarrufu + ilgisiz context azalır.

---

## 3.4 Plan Mode vs Direct Execution

### Plan mode ne zaman
- Large-scale değişiklik (onlarca dosya)
- Mimari kararlar
- Çok sayıda valid approach
- Infrastructure farkı olan entegrasyon seçenekleri

### Direct execution ne zaman
- Single-file bug fix (stack trace net)
- Tek fonksiyonda validation eklemek
- Well-scoped, clear değişim

### Combine pattern
Plan mode ile investigate et → planı onayla → direct execution ile uygula.

### Explore subagent
Verbose discovery output'u izole et, main conversation context'ini koru. Büyük keşif fazlarında kullan.

**Sample Q5**: Monolith → microservices restructure = mimari karar + onlarca dosya → **plan mode** (A). "Once başla, sonra görürsek plan'a geçeriz" (D) yanlış — karmaşıklık zaten baştan belli.

---

## 3.5 Iterative Refinement Teknikleri

### En etkili teknikler (sıralı tercih)
1. **Concrete input/output örnekleri** — prose bazen tutarsız yorumlanır, örnek kesin.
2. **Test-driven iteration** — test suite yaz, failure'ları paylaş.
3. **Interview pattern** — Claude'a sorular sor; üretmeden önce dev'in düşünmediği soruları yüzeye çıkarır.

### Tek mesaj vs sıralı düzeltme
- **Interacting problems** (birbirine bağlı, düzeltmeler çakışır) → tek detaylı mesaj
- **Independent problems** → sıralı, tek tek

### Örnek kullanımlar
- "Null handling bozuk" demek yerine 2-3 concrete örnek ver.
- Migration script için: test failures → model self-corrects.

---

## 3.6 CI/CD Entegrasyonu

### Headless flag
```bash
claude -p "Analyze this pull request for security issues"
```
`-p` (veya `--print`) = non-interactive mode. Stdout'a yanıt basıp exit eder. **Sample Q10 canonical'i** — `CLAUDE_HEADLESS`, `--batch` gibi flag'ler **mevcut değil**.

### Structured output in CI
```bash
claude -p "..." --output-format json --json-schema schema.json
```
Parse edilebilir output → inline PR yorumu olarak otomatik post.

### CI context isolation insight
Kod üreten session ile review yapan session **aynı olmamalı**. Generator, kendi reasoning context'ini taşır ve kendi kararlarını sorgulamaz. **Bağımsız ikinci instance** daha etkili review yapar.

### Duplicate comment önlemi
Re-review sonrası: prior review findings'i context'e dahil et → Claude "yalnızca **yeni veya hâlâ çözülmemiş** issue'ları raporla" talimatıyla duplicate yorumu engeller.

### Test generation kalitesi
Existing test dosyalarını context'e koy → duplicate test suggestion azalır.

### CLAUDE.md CI'de
Testing standards, fixture conventions, review criteria'yı **CLAUDE.md'de** tanımla → CI-invoked Claude Code onları alır.

---

## Domain 3 — Özet Çalışma Listesi
- [ ] CLAUDE.md hiyerarşi: user / project / directory.
- [ ] `.claude/commands/` (takım) vs `~/.claude/commands/` (kişisel).
- [ ] SKILL.md frontmatter: `context: fork`, `allowed-tools`, `argument-hint`.
- [ ] `context: fork` verbose output'ları izole eder.
- [ ] `.claude/rules/` + glob = path-specific, cross-directory konvansiyon.
- [ ] Plan mode: mimari + large-scale; direct: well-scoped.
- [ ] `-p` flag = headless; `--output-format json` + `--json-schema`.
- [ ] Independent Claude instance > self-review.
- [ ] Prior findings context'te = duplicate comment yok.
