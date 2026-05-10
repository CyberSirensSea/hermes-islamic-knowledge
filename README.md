# Hermes Islamic Knowledge Skill

A comprehensive Islamic knowledge reference skill for [Hermes Agent](https://github.com/NapthaAI/hermes-agent) — the CLI AI agent.

## What This Skill Covers

- **Quran** — All 114 surahs with metadata (Arabic/Chinese/English names, revelation place, verse count, key themes), plus Quran.com API v4 integration for live verse retrieval with Chinese translations
- **Hadith** — Six major collections (Kutub al-Sittah) plus supporting collections; grading system (sahih/hasan/da'if); CDN-based retrieval; search methodology
- **Fiqh (Islamic Jurisprudence)** — Four Sunni schools (Hanafi, Maliki, Shafi'i, Hanbali) with comparative rulings; usul al-fiqh principles; legal maxims (qawa'id fiqhiyyah); five categories of rulings (ahkam)
- **Tafsir** — Major exegesis works, methodology classification, asbab al-nuzul (occasions of revelation)
- **Cross-validation framework** — Multi-axis verification (Quran↔Quran, Quran↔Hadith, Hadith↔Hadith, School↔School) before presenting any information

## Installation

```bash
# Copy the skill to your Hermes skills directory
mkdir -p ~/.hermes/skills/knowledge/islamic-knowledge
cp SKILL.md ~/.hermes/skills/knowledge/islamic-knowledge/
```

Then Hermes will auto-discover it. The skill activates automatically when you ask about any Islamic topic.

## Usage Examples

Once installed, just ask Hermes naturally:

- "古兰经第2章255节是什么？" — fetches verse with Chinese translation via Quran.com API
- "关于求知，有什么圣训？" — searches hadith collections with grading
- "四大教法学派对利息的立场？" — comparative fiqh across schools
- "雅辛章有什么尊贵？" — surah info with related hadith
- "古兰经和圣训对饮酒的立场一致吗？" — cross-reference Quran + Hadith

## Features

- Default output language: Simplified Chinese
- Live API integration with Quran.com v4 (no API key needed)
- Hadith CDN access for English text + web search for Chinese
- Cross-school fiqh comparison tables
- Honest boundaries: distinguishes scholarly opinions from divine edicts; notes disagreements between schools
- Verification checklist before presenting any Islamic knowledge

## Skill Metadata

- **Name**: islamic-knowledge
- **Category**: knowledge
- **Language**: Simplified Chinese (default)
- **APIs Used**: Quran.com v4, MuslimSalat.com, Hadith CDN (jsDelivr)

## License

MIT — see [LICENSE](./LICENSE)
