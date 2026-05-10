---
name: islamic-knowledge
description: Query and reference the Quran (114 surahs), hadith collections, Islamic jurisprudence, and terminology. Use when the user asks about Quran verses/chapters, hadith sources, fiqh rulings, Islamic concepts, prayer times, or comparative religion. Default to Simplified Chinese output unless user specifies otherwise.
---

# Islamic Knowledge

## Overview

A comprehensive reference skill for Islamic knowledge retrieval. Covers the Quran (all 114 surahs with metadata), major hadith collections (Six Books + supporting), four Sunni schools of fiqh, core terminology, and API-based live verse retrieval via Quran.com v4.

This skill provides the *framework and index*. Actual verse text, hadith content, and real-time queries are fetched via web APIs or web search — not embedded in the skill file itself (which would be hundreds of MB).

## Activation Rules

- Load this skill when the user asks about any Islamic topic: Quran, hadith, fiqh, Islamic history, prayer, terminology
- Default language: Simplified Chinese
- For verse-by-verse queries, always fetch from Quran.com API rather than relying on memory
- For hadith queries with citation requirements, use web search to verify exact wording and chain
- For fiqh rulings, always cite the school (madhhab) and note that rulings may differ between schools
- Do not issue fatwas or definitive religious rulings — frame fiqh answers as "according to [school/ scholar]..."
- When quoting scripture, include chapter:verse reference in the answer

## Workflow

### Verse Lookup

1. Identify the surah number and ayah numbers from user query
2. Fetch via Quran.com API: `GET /api/v4/verses/by_key/{surah}:{ayah}?translations=56` (56 = Ma Jain Chinese)
3. Format output: Arabic text (if requested) + Chinese translation + verse reference + brief context

### Chapter Info

1. Look up in the 114 Surah Quick Index below
2. Return: surah name (Arabic/Chinese/English), revelation place, verse count, juz, key themes

### Hadith Search

1. **Identify collection**: Start with Bukhari and Muslim as gold standard
2. **Search CDN**: `curl -s 'https://cdn.jsdelivr.net/gh/fawazahmed0/hadith-api@1/editions/eng-bukhari.json' | grep -i '{keyword}'`
3. **Web search for Chinese**: Use DashScope with `圣训 [topic] [布哈里/穆斯林/etc]`
4. **Cite properly**: Collection name, Book number, Hadith number (e.g., Bukhari 1:1)
5. **Include grading**: State sahih/hasan/da'if when known; cite grading source
6. **Cross-reference**: Note when a hadith appears in multiple collections (muttafaqun 'alayh)
7. See full hadith methodology in ## Hadith Collections below

### Fiqh Query

1. **Identify the school(s)**: Hanafi / Maliki / Shafi'i / Hanbali (Sunni four); Ja'fari (Shi'a). See ## Fiqh > Four Sunni Schools for regional defaults
2. **Classify the ruling**: Which of the 5 ahkam categories does it fall under? (Fard/Mustahabb/Mubah/Makruh/Haram)
3. **Search Quran + Hadith**: DashScope `[topic] [school] fiqh ruling` + verify primary sources
4. **Check cross-school differences**: Use the Cross-School Comparative Table below; note khilaf (scholarly disagreement)
5. **Apply legal maxims**: Check if any of the 5 key maxims (qawa'id fiqhiyyah) apply (hardship → ease, certainty not overruled by doubt, etc.)
6. **Present with caveats**: "The majority view in the [X] school is..." + evidence + dissenting views if notable
7. See full fiqh methodology in ## Fiqh below

## API Reference

### Quran.com API v4 (Primary)

Base URL: `https://api.quran.com/api/v4`

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/chapters?language=en` | GET | All 114 chapters with metadata |
| `/chapters/{id}?language=en` | GET | Single chapter info |
| `/verses/by_key/{surah}:{ayah}?translations=56` | GET | Verse with Chinese translation (56=Ma Jain) |
| `/verses/by_key/{surah}:{ayah}?words=true` | GET | Verse with word-by-word breakdown |
| `/verses/by_key/{surah}:{ayah}?translations=56,109` | GET | Verse with multiple Chinese translations |
| `/search?q={query}&size={n}&language=en` | GET | Search in English/Arabic (NOT Chinese) |
| `/resources/translations` | GET | List all available translation resources |
| `/juz/{id}` | GET | Verses in a specific juz (1-30) |

**Chinese Translation IDs:**

| ID | Translator | Language |
|----|-----------|----------|
| 56 | Ma Jain (马坚) | chinese |
| 109 | Muhammad Makin | chinese |

**Limitations:**
- Search only works in English and Arabic — Chinese keyword search returns 0 results
- No API key required
- Cloudflare fronted, 8-day default cache
- Recommended: max 10 req/s

### MuslimSalat.com (Prayer Times)

Base URL: `https://muslimsalat.com/{city}.json`

Returns: fajr, shurooq, dhuhr, asr, maghrib, isha times + qibla direction

### Hadith CDN (Static, Free)

Base URL: `https://cdn.jsdelivr.net/gh/fawazahmed0/hadith-api@1/editions/`

Available collections (English): eng-bukhari, eng-muslim, eng-abudawud, eng-tirmidhi, eng-nasai, eng-ibnmajah, eng-malik, eng-ahmad

**Note:** No Chinese translations available in this CDN. For Chinese hadith, use web search.

### Web Search Fallback

When APIs are unavailable, use DashScope search or web_search with these patterns:
- Quran: `古兰经 第{X}章 第{Y}节 中文`
- Hadith: `圣训 [布哈里/穆斯林/etc] [主题]`
- Fiqh: `伊斯兰教法 [学派] [问题]`

## Hadith Collections

### Overview

Hadith (圣训, plural: ahadith) are the recorded sayings, actions, and tacit approvals of Prophet Muhammad ﷺ. They form the second primary source of Islamic law after the Quran. The science of hadith criticism (ulum al-hadith / 圣训学) is one of Islam's most rigorous scholarly disciplines, involving chain authentication (isnad), narrator biography (ilm al-rijal), and textual analysis (matn).

### Hadith Structure

| Component | Arabic | Description |
|-----------|--------|-------------|
| **Isnad** (传述链) | الإسناد | Chain of narrators linking the hadith back to the Prophet |
| **Matn** (正文) | المتن | The actual text/content of the report |
| **Taraf** (首句) | الطرف | Opening phrase used to identify the hadith |
| **Mukharrij** (辑录者) | المخرج | The compiler who recorded the hadith |

### Hadith Grading System (圣训等级)

| Grade | Arabic | Meaning | Reliability |
|-------|--------|---------|-------------|
| **Sahih** (健全) | صحيح | Sound/authentic — unbroken chain of trustworthy narrators with no hidden defects | Accepted for law and belief |
| **Hasan** (良好) | حسن | Good — slightly lesser narrator accuracy than sahih, but still reliable | Accepted for law |
| **Da'if** (羸弱) | ضعيف | Weak — chain break, narrator weakness, or contradiction | Not for law; may be cited for virtues (fada'il) with caution |
| **Mawdu'** (伪造) | موضوع | Fabricated — proven forgery | Rejected entirely |
| **Munkar** (隐昧) | منكر | Rejected — contradicts stronger sources | Rejected |
| **Mursal** (连环中断) | مرسل | Loose — Successor quotes Prophet directly, missing Companion link | Disputed; accepted by Hanafi/Maliki, rejected by Shafi'i unless supported |

**Key terminology in grading:**

| Term | Meaning |
|------|---------|
| Muttafaqun 'alayh (一致公认) | Agreed upon — recorded by both Bukhari and Muslim |
| Mutawatir (连续传述) | Mass-transmitted — so many chains it's impossible to be fabricated |
| Ahad (单传) | Single-chain — not mutawatir, but can be sahih |
| Gharib (孤僻) | Strange — only one narrator at some level of the chain |
| Maqbul (被接受) | Accepted — general term for sahih and hasan |

### Six Major Collections (Kutub al-Sittah / 六大圣训集)

| # | Collection | Arabic | Compiler | Death (AH) | Hadiths | Books | Distinctive Feature |
|---|-----------|--------|----------|------------|---------|-------|---------------------|
| 1 | **Sahih al-Bukhari** | صحيح البخاري | Muhammad al-Bukhari | 256 | 7,589 | 98 | Most authentic book after Quran; strictest criteria; ~2,600 unique without repetition |
| 2 | **Sahih Muslim** | صحيح المسلم | Muslim ibn al-Hajjaj | 261 | 7,563 | 57 | Second most authentic; superior organization; includes narrator-comparison methodology |
| 3 | **Sunan Abu Dawud** | سنن أبي داود | Abu Dawud al-Sijistani | 275 | 5,274 | 44 | Focus on legal (fiqh) hadiths; author's grading notes are preserved |
| 4 | **Jami' al-Tirmidhi** | جامع الترمذي | Al-Tirmidhi | 279 | 3,998 | 50 | Includes author's own grading + notes on how schools of law use each hadith |
| 5 | **Sunan al-Nasa'i** | سنن النسائي | Al-Nasa'i | 303 | 5,765 | 52 | Known as "al-Mujtaba" (the Selected); fewer weak hadiths than Abu Dawud or Tirmidhi |
| 6 | **Sunan Ibn Majah** | سنن ابن ماجه | Ibn Majah | 273 | 4,343 | 38 | Added later to the Six; contains ~3,000 unique hadiths not in other 5 |

**Why these six?** The consensus stabilized around the 11th century. Some early scholars included Muwatta Malik instead of Ibn Majah. The "Five" (without Ibn Majah) is also a recognized canonical set.

### Additional Major Collections

| Collection | Arabic | Compiler | Death | Hadiths | Note |
|-----------|--------|----------|-------|---------|------|
| **Muwatta Malik** | موطأ مالك | Malik ibn Anas | 179 | 1,858 | Earliest surviving hadith collection; organized by fiqh topics; founding text of Maliki school |
| **Musnad Ahmad** | مسند أحمد | Ahmad ibn Hanbal | 241 | ~27,000 | Organized by Companion narrator (musnad format), not by topic; foundational to Hanbali school |
| **Sunan al-Darimi** | سنن الدارمي | Al-Darimi | 255 | ~3,500 | Sometimes considered the "seventh" book |
| **Sahih Ibn Khuzaymah** | صحيح ابن خزيمة | Ibn Khuzaymah | 311 | ~3,000 | Rigorous sahih-only compilation |
| **Sahih Ibn Hibban** | صحيح ابن حبان | Ibn Hibban | 354 | ~7,500 | Sahih-focused with unique organizational system |
| **Mustadrak al-Hakim** | مستدرك الحاكم | Al-Hakim al-Naysaburi | 405 | ~8,800 | Claims to include sahih hadiths Bukhari and Muslim "missed"; controversial grading |

### Famous Hadith Compilations (Popular Use)

| Collection | Arabic | Compiler | Death | Count | Use |
|-----------|--------|----------|-------|-------|-----|
| **Forty Hadith Nawawi** | الأربعون النووية | Imam al-Nawawi | 676 | 42 | Most famous short collection; core hadiths every Muslim should know |
| **Riyad al-Salihin** | رياض الصالحين | Imam al-Nawawi | 676 | ~1,900 | "Gardens of the Righteous" — most popular devotional hadith book worldwide |
| **Bulugh al-Maram** | بلوغ المرام | Ibn Hajar al-Asqalani | 852 | ~1,600 | Fiqh-focused; hadiths underlying Shafi'i school rulings |
| **Mishkat al-Masabih** | مشكاة المصابيح | Al-Tabrizi | ~741 | ~6,000 | Widely used in South Asian madrasas |
| **Forty Hadith Qudsi** | الحديث القدسي | Various | — | 40 | Sacred hadiths where the Prophet quotes Allah directly |

### Hadith Qudsi vs. Hadith Nabawi vs. Quran

| Type | Source | Wording | Status |
|------|--------|---------|--------|
| **Quran** (古兰经) | Allah — exact words revealed through Jibril | Divine wording and meaning | Recited in prayer, inimitable (i'jaz), transmitted by tawatur |
| **Hadith Qudsi** (神圣圣训) | Allah — meaning from Allah, wording from Prophet | Divine meaning, Prophetic wording | NOT recited in prayer, NOT part of Quran's inimitability |
| **Hadith Nabawi** (先知圣训) | Prophet Muhammad ﷺ | Prophetic wording expressing Prophetic insight | Second source of law, not recited in prayer |

### Hadith Retrieval Methods

#### Method 1: CDN Direct Access (Fast, No Auth)

```bash
# Fetch full collection JSON (English)
curl -s 'https://cdn.jsdelivr.net/gh/fawazahmed0/hadith-api@1/editions/eng-bukhari.json' | jq '.hadiths[] | select(.hadithnumber == 1)'

# Get metadata-only first
curl -s 'https://cdn.jsdelivr.net/gh/fawazahmed0/hadith-api@1/editions/eng-bukhari.json' | jq '.metadata.sections'
```

**Available editions:** eng-bukhari (7,589), eng-muslim (7,563), eng-abudawud (5,274), eng-tirmidhi (3,998), eng-nasai (5,765), eng-ibnmajah (4,343), eng-malik (1,858), eng-nawawi (42), eng-qudsi (40), eng-dehlawi

**Limitations:**
- English only — no Chinese translations
- Full files are ~2-7MB each (Bukhari JSON ~7MB)
- Static JSON — no search API; must download full file and grep
- No grading metadata in many entries

#### Method 2: Web Search (For Chinese + Verification)

When Chinese text is needed or CDN lacks context:

```
DashScope search patterns:
- 布哈里圣训 [编号] [主题]
- 圣训 "求知" 布哈里 穆斯林
- hadith [topic] sahih bukhari muslim site:sunnah.com
- [Arabic hadith text] تخريج
```

#### Method 3: sunnah.com (Web)

Structure: `https://sunnah.com/{collection}/{book}:{hadith}`

Examples:
- `https://sunnah.com/bukhari:1:1` — Bukhari, Book 1, Hadith 1 (Revelation)
- `https://sunnah.com/muslim:1:1` — Muslim, Book 1, Hadith 1

Provides English + Arabic with grading. No API — use browser tool or scrape.

### Key Narrator Companions (Most Prolific)

| Companion | Arabic | Hadiths Narrated |
|-----------|--------|------------------|
| Abu Hurayrah | أبو هريرة | ~5,374 |
| Abdullah ibn Umar | عبد الله بن عمر | ~2,630 |
| Anas ibn Malik | أنس بن مالك | ~2,286 |
| Aisha bint Abu Bakr | عائشة بنت أبي بكر | ~2,210 |
| Abdullah ibn Abbas | عبد الله بن عباس | ~1,660 |
| Jabir ibn Abdullah | جابر بن عبد الله | ~1,540 |

### Hadith Search Workflow

1. **Identify collection**: Bukhari and Muslim are the gold standard. Start there.
2. **Search CDN**: If topic is known, grep the relevant JSON.
3. **Web search**: For Chinese text, context, grading, and commentary.
4. **Cite properly**: Collection name, Book number, Hadith number.
5. **Include grading**: State sahih/hasan/da'if when known.
6. **Cross-reference**: When a hadith appears in multiple collections (muttafaqun 'alayh), note it.

### Warning on Hadith Fabrication

- Imam Bukhari selected only ~7,500 from over 600,000 narrations he examined (1.2% acceptance rate)
- Fabrication of hadiths began during the Companions' era (political and sectarian motives)
- Key forgery detection methods: chain analysis, narrator biography, textual contradiction with established hadith
- Modern online sources vary in quality — prefer established scholars' verified editions
- Always verify: "Whoever narrates a hadith knowing it to be false is one of the liars" (Muslim)

## Fiqh (Islamic Jurisprudence / 伊斯兰教法学)

### Overview

Fiqh (فقه, literally "deep understanding") is the human endeavor to derive practical legal rulings from the revealed sources. It is distinct from Shari'ah (شريعة), which is the divine law itself. Fiqh is the fallible human interpretation; Shari'ah is the infallible divine ideal.

The scholar who practices fiqh is a **faqih** (فقيه, plural: fuqaha). A formal legal opinion is a **fatwa** (فتوى) issued by a **mufti** (مفتي). A judge who rules in court is a **qadi** (قاضي).

### Four Sunni Schools (Madhahib / 四大教法学派)

A **madhhab** (مذهب, plural: madhahib) is a school of legal methodology — not a different religion, but a different systematic approach to interpreting the same sources. All four Sunni schools recognize each other as equally orthodox.

| # | School | Arabic | Founder | Birth-Death (AH/CE) | Core Regions | Methodology Emphasis |
|---|--------|--------|---------|---------------------|--------------|---------------------|
| 1 | **Hanafi** (哈乃斐) | الحنفي | Abu Hanifa al-Nu'man | 80-150 / 699-767 | Turkey, Central Asia, South Asia, Balkans, China | Qiyas (analogy) + Istihsan (juristic preference); most flexible on custom (urf) |
| 2 | **Maliki** (马立克) | المالكي | Malik ibn Anas | 93-179 / 711-795 | North Africa, West Africa, Sudan, Gulf states | Amal ahl al-Madinah (practice of Medina) as primary source; Maslahah (public interest) |
| 3 | **Shafi'i** (沙斐仪) | الشافعي | Muhammad al-Shafi'i | 150-204 / 767-820 | Southeast Asia, East Africa, Yemen, Egypt | Synthesizer — first to systematize usul al-fiqh; strict on hadith authentication |
| 4 | **Hanbali** (罕百里) | الحنبلي | Ahmad ibn Hanbal | 164-241 / 780-855 | Saudi Arabia, Qatar, UAE, pockets elsewhere | Most literalist; minimizes qiyas; prefers weak hadith over analogy; influences Salafi thought |

**Historical note on school formation:**
- The "founders" did not set out to create "schools" — they were teaching circles that students systematized
- Schools crystallized over ~200 years after the founders' deaths
- Taqlid (تقليد — following a school) became the norm by the 4th Islamic century
- A lay Muslim follows one school for consistency; scholars may engage in ijtihad (独立判断) across schools

### Other Schools (Beyond the Four)

| School | Arabic | Founder | Status |
|--------|--------|---------|--------|
| **Ja'fari** (贾法里) | الجعفري | Ja'far al-Sadiq | Primary Shi'a school; accepts imams as infallible sources; differs on temporary marriage (mut'ah) and inheritance |
| **Zaydi** (宰德) | الزيدي | Zayd ibn Ali | Shi'a sub-branch closest to Sunni fiqh; found in Yemen |
| **Ibadi** (伊巴德) | الإباضي | Jabir ibn Zayd | Kharijite-descended; found in Oman and parts of North Africa |
| **Zahiri** (扎希里) | الظاهري | Dawud al-Zahiri | Literalist school rejecting qiyas; largely extinct but influential through Ibn Hazm |

### Usul al-Fiqh (法源学 / Principles of Islamic Jurisprudence)

The methodology by which rulings are derived from sources.

#### Agreed Primary Sources (All Four Schools)

| Source | Arabic | Definition |
|--------|--------|------------|
| **Quran** | القرآن | The revealed book — definitive in authenticity (qat'i al-thubut) |
| **Sunnah** | السنة | Prophetic precedent — words, actions, tacit approvals. Authenticated through hadith |
| **Ijma'** | الإجماع | Scholarly consensus — the agreement of qualified jurists of a generation on a legal ruling |
| **Qiyas** | القياس | Analogical reasoning — extending a ruling from an original case to a new case sharing the same effective cause (illah) |

#### Disputed / Supplementary Sources

| Source | Arabic | Accepted By | Definition |
|--------|--------|-------------|------------|
| **Istihsan** | الاستحسان | Hanafi, Maliki | Juristic preference — departing from strict qiyas for a stronger (but less obvious) reason |
| **Maslahah Mursalah** | المصلحة المرسلة | Maliki, Hanbali | Unrestricted public interest — no specific text but aligns with Shari'ah objectives |
| **Urf / Adah** | العرف / العادة | Hanafi, Maliki | Custom and usage — local norms recognized as legally relevant |
| **Amal Ahl al-Madinah** | عمل أهل المدينة | Maliki (unique) | Practice of the people of Medina — treated as a living transmission of Prophetic practice |
| **Istishab** | الاستصحاب | All (varying weight) | Presumption of continuity — a state remains until evidence proves change (innocent until proven guilty) |
| **Sadd al-Dhara'i** | سد الذرائع | Maliki, Hanbali | Blocking the means — prohibiting permissible acts that lead to prohibited results |
| **Qawl al-Sahabi** | قول الصحابي | Varies | Opinion of a Companion — Hanafi and Maliki give it weight; Shafi'i generally doesn't |
| **Shar' man Qablana** | شرع من قبلنا | Varies | Laws of previous prophets — if Quran/Sunnah confirm, they apply unless abrogated |

### Maqasid al-Shari'ah (教法宗旨 / Objectives of Islamic Law)

The higher objectives that all rulings ultimately serve. Formalized by al-Shatibi (d. 790 AH).

| Level | Arabic | Category | Examples |
|-------|--------|----------|----------|
| **Essential** (必需) | Daruriyyat | 5 universals protected by all rulings | Religion, Life, Intellect, Lineage/Progeny, Property |
| **Needed** (需要) | Hajiyyat | Eases hardship | Lease contracts, hunting permission, shortening prayers while traveling |
| **Embellishment** (改善) | Tahsiniyyat | Refinements for dignity | Etiquettes of eating, dress, greeting |

### Five Categories of Rulings (Ahkam / 断法五类)

Every human act falls into one of five categories.

| Ruling | Arabic | Meaning | Consequence | Example |
|--------|--------|---------|-------------|---------|
| **Fard / Wajib** (主命) | فرض / واجب | Obligatory | Reward for doing; sin for omitting | Five daily prayers, Ramadan fasting, Zakat |
| **Mustahabb / Mandub / Sunnah** (可嘉) | مستحب / مندوب | Recommended | Reward for doing; no sin for omitting | Extra prayers, charity beyond zakat, siwak (tooth-stick) |
| **Mubah / Halal** (允许) | مباح / حلال | Permissible / Indifferent | No reward, no sin | Eating, sleeping, commerce (generally) |
| **Makruh** (可憎) | مكروه | Disliked / Discouraged | No sin, but reward for avoiding | Divorce (most disliked permissible act), overeating, wasting water in ablution |
| **Haram** (禁止) | حرام | Prohibited / Forbidden | Sin for doing; reward for avoiding | Alcohol, gambling, pork, usury/riba, killing unjustly |

**Hanafi sub-distinction:** Hanafis split Fard (قطعي — definitive proof) from Wajib (ظني — probable proof). Others treat them as synonyms.

### Cross-School Comparative Table (Key Issues)

| Issue | Hanafi | Maliki | Shafi'i | Hanbali |
|-------|--------|--------|---------|---------|
| **Basmalah in Fatihah** | Recite quietly | Don't recite in loud prayers | Recite aloud in loud prayers | Recite quietly |
| **Hands in prayer** | Below navel | At sides (sadl) | On chest above navel | On chest |
| **Wudu breaks with touch** | Opposite-gender touch doesn't break | Touch with pleasure breaks | Any skin contact with opposite gender breaks | Skin contact with desire breaks |
| **Wudu after eating camel** | Does not break | Breaks wudu | Does not break | Breaks wudu |
| **Bleeding breaks wudu** | Breaks wudu | Does not break | Does not break | Does not break |
| **Reciting Fatihah behind imam** | Makruh to recite | Listen — don't recite in loud prayers | Must recite in all prayers | Must recite in silent prayers; listen in loud |
| **Amin after Fatihah** | Say quietly | Say aloud in loud prayers | Say aloud in loud prayers | Say aloud in loud prayers |
| **Minimum dowry (mahr)** | 10 dirhams | No minimum (whatever is considered valuable) | No minimum (whatever has value) | No minimum (whatever has value) |
| **Seafood** | Only fish (others makruh/haram) | All sea creatures halal | All sea creatures halal | All sea creatures halal (except frog, crocodile) |
| **Dog as pet** | Permitted for guarding/herding (saliva impure) | Permitted for need; saliva pure | Haram except for benefit; must wash 7x (1x with earth) if touched | Permitted for need; must wash 7x if touched |
| **Music** | Disliked (makruh); instruments prohibited except duff | Instruments prohibited; some permit singing | Instruments prohibited; some permit singing for occasions | Instruments prohibited; severe position |
| **Drawing living beings** | 2D drawings of living beings with full features prohibited | Same as Hanafi | Same; more strict on photography of living beings | Strictest — all 2D images of animate beings prohibited |
| **Contractual interest (riba)** | Prohibited. Murabaha, ijara, musharaka permitted | Same, with stricter adherence to Madinan commercial practice | Same, with detailed formal constraints | Same, with fewest permitted workarounds |
| **Fasting: deliberate eating** | Qada (make up) + Kaffarah (expiation) | Qada + Kaffarah | Qada + Kaffarah | Qada + Kaffarah |
| **Fasting: cupping/hijama** | Does not break fast | Breaks fast | Does not break fast (preferred view) | Breaks fast |

### Key Fiqh Terminology

| Term | Arabic | Definition |
|------|--------|------------|
| Ijtihad | اجتهاد | Independent legal reasoning by a qualified scholar |
| Taqlid | تقليد | Following a qualified school/scholar without re-examining primary sources |
| Fatwa | فتوى | Non-binding legal opinion issued by a mufti |
| Qada | قضاء | Binding judgment by a qadi (judge) in a court |
| Rukhsah | رخصة | Legal concession — ease in hardship (e.g., breaking fast while traveling) |
| Azimah | عزيمة | The original strict ruling without considering hardship |
| Ihtiyat | احتياط | Precaution — taking the safer opinion to avoid risk of sin |
| Khilaf | خلاف | Scholarly disagreement — recognized and tolerated within Sunni orthodoxy |
| Raf' al-Haraj | رفع الحرج | Removal of hardship — a core legal maxim |
| al-Yaqin la yazulu bi al-shakk | اليقين لا يزول بالشك | "Certainty is not overruled by doubt" — most famous legal maxim |

### Key Legal Maxims (al-Qawa'id al-Fiqhiyyah / 教法准则)

| # | Maxim (Arabic) | Translation | Application |
|---|---------------|-------------|-------------|
| 1 | الأمور بمقاصدها | Matters are judged by their intentions | Contracts, worship validity |
| 2 | اليقين لا يزول بالشك | Certainty is not removed by doubt | Presumption of innocence, purity |
| 3 | المشقة تجلب التيسير | Hardship brings ease | Travel concessions, illness exemptions |
| 4 | الضرر يزال | Harm must be removed | Liability, consumer protection |
| 5 | العادة محكمة | Custom is authoritative | Trade norms, local marriage customs |

### Fiqh Retrieval Methods

#### Method 1: Web Search (Primary — for current rulings)

```
DashScope / web search patterns:
- [ruling topic] [school] fiqh ruling
- 伊斯兰教法 [问题] [学派]
- fatwa [topic] [school] islam
- ruling on [topic] hanafi maliki shafii hanbali
```

#### Method 2: Structured Quran + Hadith Grounding

1. Identify the topic in the Quran (use search API for keywords)
2. Find relevant hadiths (CDN + web search)
3. Note: raw scripture → ruling requires usul al-fiqh methodology
4. When uncertain, present the evidence without claiming a definitive ruling

#### Method 3: Fatwa Sites (Reference Only)

Note: Online fatwa sites vary widely in quality. Prefer:
- Institutional fatwas (Dar al-Ifta, AMJA, European Council for Fatwa)
- Over individual scholar websites
- Match the madhhab to the questioner's context
- Be transparent about source quality

### Warning on Fiqh Answers

- **Never issue definitive fatwas.** Present rulings as "the majority view in the [X] school is..."
- **Always cite evidence:** Quran verse, hadith reference, scholarly source
- **Acknowledge disagreement:** "Hanafis say... while Shafi'is hold..." when differences are established
- **Distinguish binding from advisory:** Qadi judgments bind; mufti opinions (fatwas) are non-binding
- **The 5 categories are a spectrum, not binary:** Most things are mubah unless proven otherwise
- **Local custom matters:** Hanafi and Maliki schools especially incorporate urf
- **Context affects ruling:** Time, place, necessity (darurah), and custom all modify outcomes
- **When in doubt, refer to scholars:** This skill is a reference tool, not a scholar

## 114 Surah Quick Index

| # | Arabic | Chinese | English | Place | Verses | Juz | Key Themes |
|---|--------|---------|---------|-------|--------|-----|------------|
| 1 | الفاتحة | 开端章 | Al-Fatihah | Meccan | 7 | 1 | Opening prayer, praise, guidance |
| 2 | البقرة | 黄牛章 | Al-Baqarah | Medinan | 286 | 1-3 | Law, faith, history of Israel, Ayat al-Kursi (2:255) |
| 3 | آل عمران | 仪姆兰的家属章 | Ali 'Imran | Medinan | 200 | 3-4 | Family of Imran, Jesus, Battle of Uhud |
| 4 | النساء | 妇女章 | An-Nisa | Medinan | 176 | 4-6 | Women, inheritance, marriage, orphans |
| 5 | المائدة | 筵席章 | Al-Ma'idah | Medinan | 120 | 6-7 | Table spread, dietary laws, Jesus |
| 6 | الأنعام | 牲畜章 | Al-An'am | Meccan | 165 | 7-8 | Monotheism, Abraham, dietary prohibitions |
| 7 | الأعراف | 高处章 | Al-A'raf | Meccan | 206 | 8-9 | Heights, prophets, Adam & Iblis, covenant |
| 8 | الأنفال | 战利品章 | Al-Anfal | Medinan | 75 | 9-10 | Spoils of war, Battle of Badr |
| 9 | التوبة | 忏悔章 | At-Tawbah | Medinan | 129 | 10-11 | Repentance, jihad, hypocrites (no basmalah) |
| 10 | يونس | 优努斯章 | Yunus | Meccan | 109 | 11 | Jonah, divine signs, punishment of nations |
| 11 | هود | 呼德章 | Hud | Meccan | 123 | 11-12 | Prophet Hud, Noah, judgment day |
| 12 | يوسف | 优素福章 | Yusuf | Meccan | 111 | 12-13 | Story of Joseph — "best of stories" |
| 13 | الرعد | 雷霆章 | Ar-Ra'd | Medinan | 43 | 13 | Thunder, nature as signs, prayer |
| 14 | إبراهيم | 易卜拉欣章 | Ibrahim | Meccan | 52 | 13-14 | Abraham's prayer, gratitude vs. ingratitude |
| 15 | الحجر | 石谷章 | Al-Hijr | Meccan | 99 | 14 | Rocky tract, Lot, creation of Adam |
| 16 | النحل | 蜜蜂章 | An-Nahl | Meccan | 128 | 14 | Bee, creation signs, prohibitions |
| 17 | الإسراء | 夜行章 | Al-Isra | Meccan | 111 | 15 | Night Journey, Isra & Mi'raj, Ten Commandments |
| 18 | الكهف | 山洞章 | Al-Kahf | Meccan | 110 | 15-16 | Cave sleepers, Moses & Khidr, Dhul-Qarnayn |
| 19 | مريم | 麦尔彦章 | Maryam | Meccan | 98 | 16 | Mary, birth of Jesus, Abraham |
| 20 | طه | 塔哈章 | Ta-Ha | Meccan | 135 | 16 | Moses, golden calf, Adam's fall |
| 21 | الأنبياء | 众先知章 | Al-Anbiya | Meccan | 112 | 17 | All prophets, Abraham vs idols, judgment |
| 22 | الحج | 朝觐章 | Al-Hajj | Medinan | 78 | 17 | Pilgrimage, sacrifice, permission to fight |
| 23 | المؤمنون | 信士章 | Al-Mu'minun | Meccan | 118 | 18 | Qualities of believers, creation stages |
| 24 | النور | 光明章 | An-Nur | Medinan | 64 | 18 | Light verse, slander of Aisha, hijab |
| 25 | الفرقان | 准则章 | Al-Furqan | Meccan | 77 | 18-19 | Criterion, qualities of true servants |
| 26 | الشعراء | 众诗人章 | Ash-Shu'ara | Meccan | 227 | 19 | Poets, Moses vs Pharaoh, destroyed nations |
| 27 | النمل | 蚂蚁章 | An-Naml | Meccan | 93 | 19-20 | Ant, Solomon, Queen of Sheba |
| 28 | القصص | 故事章 | Al-Qasas | Meccan | 88 | 20 | Moses's full story, Qarun (Korah) |
| 29 | العنكبوت | 蜘蛛章 | Al-Ankabut | Meccan | 69 | 20-21 | Spider (weakest of houses), trials of faith |
| 30 | الروم | 罗马人章 | Ar-Rum | Meccan | 60 | 21 | Romans (Byzantines), signs in creation |
| 31 | لقمان | 鲁格曼章 | Luqman | Meccan | 34 | 21 | Luqman's wisdom to his son |
| 32 | السجدة | 叩头章 | As-Sajdah | Meccan | 30 | 21 | Prostration, creation, judgment |
| 33 | الأحزاب | 同盟军章 | Al-Ahzab | Medinan | 73 | 21-22 | Confederates, Battle of Trench, wives of Prophet |
| 34 | سبأ | 赛伯邑章 | Saba | Meccan | 54 | 22 | Sheba, Solomon, gratitude |
| 35 | فاطر | 创造者章 | Fatir | Meccan | 45 | 22 | Originator (of heavens & earth), angels |
| 36 | يس | 雅辛章 | Ya-Sin | Meccan | 83 | 22-23 | "Heart of the Quran", resurrection proofs |
| 37 | الصافات | 列班者章 | As-Saffat | Meccan | 182 | 23 | Ranged in rows, Abraham's sacrifice |
| 38 | ص | 萨德章 | Sad | Meccan | 88 | 23 | David, Solomon, Job, creation of Adam |
| 39 | الزمر | 队伍章 | Az-Zumar | Meccan | 75 | 23-24 | Troops, sincerity in worship, forgiveness |
| 40 | غافر | 赦宥者章 | Ghafir | Meccan | 85 | 24 | Forgiver, Pharaoh, believer from Pharaoh's family |
| 41 | فصلت | 奉绥来特章 | Fussilat | Meccan | 54 | 24-25 | Explained in detail, creation of heavens & earth |
| 42 | الشورى | 协商章 | Ash-Shura | Meccan | 53 | 25 | Consultation, unity of revelation |
| 43 | الزخرف | 金饰章 | Az-Zukhruf | Meccan | 89 | 25 | Gold ornaments, Abraham's legacy |
| 44 | الدخان | 烟雾章 | Ad-Dukhan | Meccan | 59 | 25 | Smoke, warning, Pharaoh's drowning |
| 45 | الجاثية | 屈膝章 | Al-Jathiyah | Meccan | 37 | 25 | Kneeling, signs for believers |
| 46 | الأحقاف | 沙丘章 | Al-Ahqaf | Meccan | 35 | 26 | Sand dunes, jinn listening to Quran |
| 47 | محمد | 穆罕默德章 | Muhammad | Medinan | 38 | 26 | Prophet Muhammad, hypocrites in battle |
| 48 | الفتح | 胜利章 | Al-Fath | Medinan | 29 | 26 | Victory (Treaty of Hudaybiyyah), pledge |
| 49 | الحجرات | 寝室章 | Al-Hujurat | Medinan | 18 | 26 | Private chambers, etiquettes, brotherhood |
| 50 | ق | 戛弗章 | Qaf | Meccan | 45 | 26 | Qaf, resurrection, recording angels |
| 51 | الذاريات | 播种者章 | Adh-Dhariyat | Meccan | 60 | 26-27 | Scatterers, Abraham's guests, creation purpose |
| 52 | الطور | 山岳章 | At-Tur | Meccan | 49 | 27 | Mount Sinai, paradise descriptions |
| 53 | النجم | 星宿章 | An-Najm | Meccan | 62 | 27 | Star, first revelation to Muhammad, Mi'raj |
| 54 | القمر | 月亮章 | Al-Qamar | Meccan | 55 | 27 | Moon (splitting), destroyed nations |
| 55 | الرحمن | 至仁主章 | Ar-Rahman | Medinan | 78 | 27 | The Most Merciful — "Which of your Lord's favors will you deny?" repeated |
| 56 | الواقعة | 大事章 | Al-Waqi'ah | Meccan | 96 | 27 | Inevitable event, three classes of people |
| 57 | الحديد | 铁章 | Al-Hadid | Medinan | 29 | 27 | Iron, charity, monasticism |
| 58 | المجادلة | 辩诉者章 | Al-Mujadilah | Medinan | 22 | 28 | Pleading woman, zihar, secret counsels |
| 59 | الحشر | 放逐章 | Al-Hashr | Medinan | 24 | 28 | Exile (Banu Nadir), 99 Names of Allah |
| 60 | الممتحنة | 受考验的妇人章 | Al-Mumtahanah | Medinan | 13 | 28 | Tested woman, treaty with disbelievers |
| 61 | الصف | 列阵章 | As-Saff | Medinan | 14 | 28 | Battle ranks, Jesus foretelling Ahmad |
| 62 | الجمعة | 聚礼章 | Al-Jumu'ah | Medinan | 11 | 28 | Friday prayer, knowledge obligation |
| 63 | المنافقون | 伪信者章 | Al-Munafiqun | Medinan | 11 | 28 | Hypocrites, their characteristics |
| 64 | التغابن | 相欺章 | At-Taghabun | Medinan | 18 | 28 | Mutual disillusion, Day of Gathering |
| 65 | الطلاق | 离婚章 | At-Talaq | Medinan | 12 | 28 | Divorce, waiting period, provision |
| 66 | التحريم | 禁戒章 | At-Tahrim | Medinan | 12 | 28 | Prohibition, wives of Prophet, examples |
| 67 | الملك | 国权章 | Al-Mulk | Meccan | 30 | 29 | Dominion, creation perfection, protector from grave punishment |
| 68 | القلم | 笔章 | Al-Qalam | Meccan | 52 | 29 | Pen, Prophet's character, story of garden owners |
| 69 | الحاقة | 真灾章 | Al-Haqqah | Meccan | 52 | 29 | Inevitable reality, past nations destroyed |
| 70 | المعارج | 天梯章 | Al-Ma'arij | Meccan | 44 | 29 | Ascending stairways, human impatience |
| 71 | نوح | 努哈章 | Nuh | Meccan | 28 | 29 | Noah's mission and flood |
| 72 | الجن | 精灵章 | Al-Jinn | Meccan | 28 | 29 | Jinn listening to Quran, their belief |
| 73 | المزمل | 披衣的人章 | Al-Muzzammil | Meccan | 20 | 29 | Enwrapped one, night prayer |
| 74 | المدثر | 盖被的人章 | Al-Muddaththir | Meccan | 56 | 29 | Cloaked one, first public call, Hell guardians |
| 75 | القيامة | 复活章 | Al-Qiyamah | Meccan | 40 | 29 | Resurrection, oath by the self-reproaching soul |
| 76 | الإنسان | 人章 | Al-Insan | Medinan | 31 | 29 | Human, creation from sperm, paradise reward |
| 77 | المرسلات | 天使章 | Al-Mursalat | Meccan | 50 | 29 | Sent forth (winds), judgment day oaths |
| 78 | النبأ | 消息章 | An-Naba | Meccan | 40 | 30 | Great news, resurrection proofs, hell & paradise |
| 79 | النازعات | 急掣的章 | An-Nazi'at | Meccan | 46 | 30 | Those who drag forth, Moses vs Pharaoh |
| 80 | عبس | 皱眉章 | 'Abasa | Meccan | 42 | 30 | He frowned (at the blind man), creation proofs |
| 81 | التكوير | 黯黮章 | At-Takwir | Meccan | 29 | 30 | Overthrowing (sun folded up), cosmic signs |
| 82 | الإنفطار | 破裂章 | Al-Infitar | Meccan | 19 | 30 | Cleaving (sky split), recording angels |
| 83 | المطففين | 称量不公章 | Al-Mutaffifin | Meccan | 36 | 30 | Defrauders, two books of deeds |
| 84 | الإنشقاق | 绽裂章 | Al-Inshiqaq | Meccan | 25 | 30 | Splitting open, sky & earth on judgment day |
| 85 | البروج | 十二宫章 | Al-Buruj | Meccan | 22 | 30 | Constellations, people of the ditch |
| 86 | الطارق | 启明星章 | At-Tariq | Meccan | 17 | 30 | Night-comer (piercing star), creation from fluid |
| 87 | الأعلى | 至尊章 | Al-A'la | Meccan | 19 | 30 | The Most High, glorification, success |
| 88 | الغاشية | 大灾章 | Al-Ghashiyah | Meccan | 26 | 30 | Overwhelming event, faces on judgment day |
| 89 | الفجر | 黎明章 | Al-Fajr | Meccan | 30 | 30 | Dawn, destroyed nations, soul at peace |
| 90 | البلد | 地方章 | Al-Balad | Meccan | 20 | 30 | The city (Mecca), two paths |
| 91 | الشمس | 太阳章 | Ash-Shams | Meccan | 15 | 30 | Sun, purified soul, Thamud's destruction |
| 92 | الليل | 黑夜章 | Al-Layl | Meccan | 21 | 30 | Night, two paths: ease vs. hardship |
| 93 | الضحى | 上午章 | Ad-Duha | Meccan | 11 | 30 | Morning brightness, comfort to Prophet |
| 94 | الشرح | 开拓章 | Ash-Sharh | Meccan | 8 | 30 | Expansion (of chest), with hardship comes ease |
| 95 | التين | 无花果章 | At-Tin | Meccan | 8 | 30 | Fig, olive, human created in best form |
| 96 | العلق | 血块章 | Al-'Alaq | Meccan | 19 | 30 | Clot — FIRST revelation (verses 1-5) |
| 97 | القدر | 高贵章 | Al-Qadr | Meccan | 5 | 30 | Power/Decree — Laylat al-Qadr (Night of Power) |
| 98 | البينة | 明证章 | Al-Bayyinah | Medinan | 8 | 30 | Clear proof, People of Book split |
| 99 | الزلزلة | 地震章 | Az-Zalzalah | Medinan | 8 | 30 | Earthquake, earth reports its news |
| 100 | العاديات | 奔驰的马章 | Al-'Adiyat | Meccan | 11 | 30 | Chargers (war horses), human ingratitude |
| 101 | القارعة | 大难章 | Al-Qari'ah | Meccan | 11 | 30 | Calamity, scales of deeds |
| 102 | التكاثر | 竞赛富庶章 | At-Takathur | Meccan | 8 | 30 | Rivalry in wealth, grave as certainty |
| 103 | العصر | 时光章 | Al-'Asr | Meccan | 3 | 30 | Time — "Man is in loss except..." |
| 104 | الهمزة | 诽谤者章 | Al-Humazah | Meccan | 9 | 30 | Slanderer, crushing fire (Hutamah) |
| 105 | الفيل | 象章 | Al-Fil | Meccan | 5 | 30 | Elephant, Abraha's army destroyed by birds |
| 106 | قريش | 古莱氏章 | Quraysh | Meccan | 4 | 30 | Quraysh tribe, winter & summer journeys |
| 107 | الماعون | 什物章 | Al-Ma'un | Meccan | 7 | 30 | Small kindnesses, hypocrisy in prayer |
| 108 | الكوثر | 多福章 | Al-Kawthar | Meccan | 3 | 30 | Abundance (river in paradise), sacrifice |
| 109 | الكافرون | 不信道的人们章 | Al-Kafirun | Meccan | 6 | 30 | Disbelievers — "To you your religion, to me mine" |
| 110 | النصر | 援助章 | An-Nasr | Medinan | 3 | 30 | Help/victory, people entering Islam in crowds |
| 111 | المسد | 火焰章 | Al-Masad | Meccan | 5 | 30 | Palm fiber, curse on Abu Lahab |
| 112 | الإخلاص | 忠诚章 | Al-Ikhlas | Meccan | 4 | 30 | Sincerity — "He is Allah, the One" — equals 1/3 of Quran |
| 113 | الفلق | 曙光章 | Al-Falaq | Meccan | 5 | 30 | Daybreak, seeking refuge from evil |
| 114 | الناس | 世人章 | An-Nas | Meccan | 6 | 30 | Mankind, seeking refuge from whisperer |

### Notable Verses Quick Reference

| Verse | Name | Significance |
|-------|------|-------------|
| 2:255 | Ayat al-Kursi (宝座经文) | Greatest verse — Allah's sovereignty |
| 2:285-286 | Last two verses of Al-Baqarah | Sufficient protection for the night |
| 3:103 | Hold fast to Allah's rope | Unity of the ummah |
| 9:128-129 | Last two of At-Tawbah | Prophet's compassion |
| 18:1-10 | Opening of Al-Kahf | Protection from Dajjal |
| 24:35 | Ayat an-Nur (光明经文) | Light verse — parable of divine light |
| 36:1-12 | Opening of Ya-Sin | "Heart of the Quran" |
| 55:1-78 | Ar-Rahman | Repeated refrain: "Which of your Lord's favors will you deny?" |
| 56:1-96 | Al-Waqi'ah | Three classes on judgment day |
| 59:22-24 | Last verses of Al-Hashr | Divine names/attributes |
| 67:1-30 | Al-Mulk | Protection from grave punishment |
| 112:1-4 | Al-Ikhlas | Pure monotheism, equals 1/3 of Quran |

### Juz Division (30 Parts)

| Juz | Surahs Covered | Starting Verse |
|-----|---------------|----------------|
| 1 | Al-Fatihah 1 – Al-Baqarah 141 | 1:1 |
| 2 | Al-Baqarah 142 – Al-Baqarah 252 | 2:142 |
| 3 | Al-Baqarah 253 – Ali 'Imran 92 | 2:253 |
| 4 | Ali 'Imran 93 – An-Nisa 23 | 3:93 |
| 5 | An-Nisa 24 – An-Nisa 147 | 4:24 |
| 6 | An-Nisa 148 – Al-Ma'idah 81 | 4:148 |
| 7 | Al-Ma'idah 82 – Al-An'am 110 | 5:82 |
| 8 | Al-An'am 111 – Al-A'raf 87 | 6:111 |
| 9 | Al-A'raf 88 – Al-Anfal 40 | 7:88 |
| 10 | Al-Anfal 41 – At-Tawbah 92 | 8:41 |
| 11 | At-Tawbah 93 – Hud 5 | 9:93 |
| 12 | Hud 6 – Yusuf 52 | 11:6 |
| 13 | Yusuf 53 – Ibrahim 52 | 12:53 |
| 14 | Al-Hijr 1 – An-Nahl 128 | 15:1 |
| 15 | Al-Isra 1 – Al-Kahf 74 | 17:1 |
| 16 | Al-Kahf 75 – Ta-Ha 135 | 18:75 |
| 17 | Al-Anbiya 1 – Al-Hajj 78 | 21:1 |
| 18 | Al-Mu'minun 1 – Al-Furqan 20 | 23:1 |
| 19 | Al-Furqan 21 – An-Naml 55 | 25:21 |
| 20 | An-Naml 56 – Al-Ankabut 45 | 27:56 |
| 21 | Al-Ankabut 46 – Al-Ahzab 30 | 29:46 |
| 22 | Al-Ahzab 31 – Ya-Sin 27 | 33:31 |
| 23 | Ya-Sin 28 – Az-Zumar 31 | 36:28 |
| 24 | Az-Zumar 32 – Fussilat 46 | 39:32 |
| 25 | Fussilat 47 – Al-Jathiyah 37 | 41:47 |
| 26 | Al-Ahqaf 1 – Adh-Dhariyat 30 | 46:1 |
| 27 | Adh-Dhariyat 31 – Al-Hadid 29 | 51:31 |
| 28 | Al-Mujadilah 1 – At-Tahrim 12 | 58:1 |
| 29 | Al-Mulk 1 – Al-Mursalat 50 | 67:1 |
| 30 | An-Naba 1 – An-Nas 6 | 78:1 |

### Makki vs. Madani Classification

- **Makki (Meccan)**: 86 surahs — revealed before Hijrah. Focus: monotheism, afterlife, stories of prophets, moral foundations
- **Madani (Medinan)**: 28 surahs — revealed after Hijrah. Focus: law, society, jihad, interactions with People of Book, hypocrites

Madani surahs: 2, 3, 4, 5, 8, 9, 13, 22, 24, 33, 47, 48, 49, 55, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 76, 98, 99, 110

## Query Patterns

### Pattern 1: Verse Lookup (经文直接查询)
```
User: "古兰经第2章255节是什么？"
Action: curl 'https://api.quran.com/api/v4/verses/by_key/2:255?translations=56'
Response: 中文翻译 + 经文背景 + 相关圣训
```

### Pattern 2: Topic Search (主题检索) — Enhanced

The Quran.com search API only indexes English and Arabic, not Chinese. Therefore, topic-based search requires a two-step method:

**Step A — English keyword bridge:**
```
User query (Chinese) → translate key terms to English → search API → get verse keys → fetch Chinese translations
```

**Step B — Web search for Chinese topics:**
```
DashScope: "古兰经 关于 [主题] 的经文"
```

**Common topic → English keyword mapping:**

| Chinese Topic | English Search Terms | Key Verses |
|--------------|---------------------|------------|
| 斋戒/封斋 | fasting, sawm, ramadan | 2:183-187 |
| 天课/施舍 | zakat, charity, sadaqah | 2:267-273, 9:60 |
| 朝觐 | hajj, pilgrimage | 2:196-203, 3:97, 22:26-37 |
| 婚姻 | marriage, nikah, wives | 4:3-4, 24:32, 30:21 |
| 离婚 | divorce, talaq | 2:226-237, 65:1-7 |
| 遗产/继承 | inheritance, mirath | 4:11-12, 4:176 |
| 利息/高利贷 | riba, usury, interest | 2:275-280, 3:130, 4:161 |
| 饮酒/酒 | wine, intoxicants, khamr | 2:219, 5:90-91 |
| 猪肉 | pork, swine | 2:173, 5:3, 6:145, 16:115 |
| 礼拜/祈祷 | prayer, salah, salat | 2:238-239, 4:101-103, 17:78-79 |
| 忍耐 | patience, sabr | 2:153-157, 3:200, 103:3 |
| 感恩/感谢 | gratitude, shukr | 14:7, 31:12, 39:7 |
| 饶恕/忏悔 | forgiveness, repentance, tawbah | 39:53-54, 66:8, 4:110 |
| 正义/公道 | justice, adl, qist | 4:58, 4:135, 5:8, 16:90 |
| 仁慈/怜悯 | mercy, rahmah | 7:156, 21:107, 39:53 |
| 知识/学问 | knowledge, ilm | 20:114, 39:9, 58:11, 96:1-5 |
| 父母/孝道 | parents, birr | 17:23-24, 31:14-15, 46:15 |
| 孤儿 | orphans, yatim | 4:2-10, 93:9, 107:2 |
| 邻居 | neighbor, jar | 4:36 |
| 禁止杀戮 | killing, murder, qatl | 5:32, 6:151, 17:33 |
| 作证/见证 | testimony, witness, shahadah | 2:282, 5:106-108, 24:4 |
| 妇女/女性 | women, nisa | 4:1-34, 24:31, 33:35 |
| 吉哈德/奋斗 | jihad, striving | 2:190-193, 9:5-6, 22:78, 25:52 |
| 末日/复活 | resurrection, judgment day, qiyamah | 75:1-15, 81:1-14, 82:1-19, 99:1-8 |
| 天堂/乐园 | paradise, jannah, firdaus | 3:133-136, 47:15, 55:46-76, 56:10-40 |
| 火狱 | hell, jahannam, nar | 4:56, 37:62-68, 56:41-56, 74:26-30 |
| 造物主/迹象 | creation, signs, ayat | 2:164, 3:190-191, 10:5-6, 45:3-5 |
| 独一/认主 | oneness, tawhid, ikhlas | 112:1-4, 2:163, 59:22-24 |
| 前定 | decree, qadr, destiny | 57:22-23, 64:11 |
| 信士/穆民 | believers, qualities | 8:2-4, 23:1-11, 49:10-13 |
| 伪信士 | hypocrites, munafiqun | 63:1-8, 2:8-16, 9:64-68 |
| 尔萨/耶稣 | Jesus, Isa | 3:45-59, 5:110-118, 19:16-36 |
| 穆萨/摩西 | Moses, Musa | 20:9-98, 28:3-42, 7:103-162 |

### Pattern 3: Hadith Lookup
```
User: "关于求知，有什么圣训？"
Action: cd CDN grep → web search "seeking knowledge hadith sahih bukhari muslim"
Response: 引用圣训全文 + 出处 (Collection Book:Hadith) + 等级 (sahih/hasan/da'if)
```

### Pattern 4: Comparative Fiqh
```
User: "四大教法学派对利息的立场？"
Action: web search "riba hanafi maliki shafii hanbali ruling"
Response: 分学派列出立场 + 经训依据 (Quran verse + hadith) + 差异要点
```

### Pattern 5: Surah Info
```
User: "雅辛章有什么尊贵？"
Action: Look up in 114 Surah Index (surah 36, Meccan, 83 verses)
Response: 基本信息 + 古兰经之"心" + 相关圣训 (如诵读 Ya-Sin 的尊贵)
```

### Pattern 6: Cross-Reference (经训交叉验证)
```
User: "古兰经和圣训对饮酒的立场一致吗？"
Action:
  1. Topic search "wine" → find Quran verses (2:219 → 4:43 → 5:90-91)
  2. CDN grep "wine/khamr" in Bukhari → find hadiths
  3. Present: gradual prohibition timeline (Quran) + enforcement hadiths
Response: 经文时间线 + 相关圣训 + 一致/差异分析
```

### Pattern 7: Seerah/History Query
```
User: "白德尔之战的古兰经依据？"
Action: web search "battle of badr quran verses surah al-anfal"
Response: 经文引用 (如 3:123, 8:5-19) + 简史 + 启示背景 (asbab al-nuzul)
```

## Tafsir (Quranic Exegesis / 古兰经注)

Tafsir is the scholarly discipline of explaining the Quran — its meanings, contexts, rulings, and linguistic nuances. Having a tafsir framework is essential for deepening any verse lookup beyond surface translation.

### Major Tafsir Works

| # | Tafsir | Arabic | Author | Death (AH) | Language | Type | Distinctive |
|---|--------|--------|--------|------------|----------|------|-------------|
| 1 | **Tafsir al-Tabari** | تفسير الطبري | Ibn Jarir al-Tabari | 310 | Arabic | Traditional (ma'thur) | Oldest surviving comprehensive tafsir; preserves earliest exegetical reports |
| 2 | **Tafsir Ibn Kathir** | تفسير ابن كثير | Ibn Kathir | 774 | Arabic | Hadith-based | Most famous and widely used; explains Quran by Quran, then by Sunnah; available in English |
| 3 | **Tafsir al-Qurtubi** | تفسير القرطبي | Al-Qurtubi | 671 | Arabic | Legal (ahkam) | Focus on legal rulings derived from verses; extensive Maliki fiqh |
| 4 | **Tafsir al-Jalalayn** | تفسير الجلالين | Al-Mahalli & Al-Suyuti | 864/911 | Arabic | Concise | "Tafsir of the Two Jalals" — most concise classical tafsir; first studied in madrasas |
| 5 | **Mafatih al-Ghayb** | مفاتيح الغيب | Fakhr al-Din al-Razi | 606 | Arabic | Rational/theological | "Keys to the Unseen" — philosophical; Ash'ari theology; very extensive (~32 volumes) |
| 6 | **Tafsir al-Baydawi** | تفسير البيضاوي | Al-Baydawi | 685 | Arabic | Concise theological | Condensed from Zamakhshari + Razi; standard in Ottoman madrasas |
| 7 | **Fi Zilal al-Quran** | في ظلال القرآن | Sayyid Qutb | 1386 (1966 CE) | Arabic | Thematic-modern | "In the Shade of the Quran" — influential modern tafsir; activist tone; English available |
| 8 | **Tafsir al-Manar** | تفسير المنار | Muhammad Abduh / Rashid Rida | 1323/1354 | Arabic | Modernist-reformist | Rational; engages science and modernity; incomplete (stopped at 12:107) |
| 9 | **Ma'ariful Quran** | معارف القرآن | Mufti Muhammad Shafi | 1396 (1976 CE) | Urdu (→ English) | Traditional Deobandi | Most comprehensive in English from South Asian tradition; 8 volumes |

### Tafsir by Methodology

| Method | Arabic | Description | Representative Works |
|--------|--------|-------------|---------------------|
| **Tafsir bi al-Ma'thur** | تفسير بالمأثور | Tradition-based — explains Quran with Quran, hadith, and Companion reports | Tabari, Ibn Kathir |
| **Tafsir bi al-Ra'y** | تفسير بالرأي | Reason-based — uses linguistic analysis, theology, and rational argument | Al-Razi, Al-Baydawi |
| **Tafsir Ahkam** | تفسير أحكام | Legal — focuses on verses with juristic rulings | Al-Qurtubi, Al-Jassas (Hanafi) |
| **Tafsir Ishari** | تفسير إشاري | Allegorical/mystical — esoteric meanings; Sufi approach | Tafsir al-Tustari, Ruh al-Ma'ani |
| **Tafsir 'Ilmi** | تفسير علمي | Scientific — seeks concordance between Quran and modern science | Modern: Tantawi Jawhari, Zaghlul al-Najjar |

### Asbab al-Nuzul (启示背景 / Occasions of Revelation)

Understanding *why* a verse was revealed is critical to correct interpretation.

| Key Work | Author | Content |
|----------|--------|---------|
| Asbab al-Nuzul | Al-Wahidi (d. 468 AH) | Most famous classical collection of revelation occasions |
| Lubab al-Nuqul | Al-Suyuti (d. 911 AH) | Standard reference used alongside tafsir |

**Principle:** "The general ruling is based on the generality of the wording, not the specificity of the occasion" (العبرة بعموم اللفظ لا بخصوص السبب). The occasion informs but does not restrict application.

### Tafsir Retrieval Strategy

1. **For general understanding**: Ibn Kathir (English available at quran.com or tafsir.com)
2. **For legal rulings**: Al-Qurtubi + cross-reference with relevant madhhab
3. **For linguistic depth**: Al-Tabari (earliest reports) + Al-Razi (rational analysis)
4. **For modern relevance**: Fi Zilal al-Quran or Tafsir al-Manar
5. **Web search**: `tafsir [surah:ayah] [ibn kathir/qurtubi]`

## Abrogation (Naskh / 废止学)

Naskh refers to the abrogation of an earlier ruling by a later revelation. Understanding naskh prevents apparent contradictions and chronological errors in applying rulings.

### Types of Abrogation

| Type | Description | Example |
|------|-------------|---------|
| **Naskh al-Hukm** | Abrogation of ruling only — verse remains recited | Gradual prohibition of alcohol: 2:219 → 4:43 → 5:90-91 |
| **Naskh al-Tilawah** | Abrogation of recitation only — ruling remains | (Rare; mostly theoretical) |
| **Naskh al-Hukm wa al-Tilawah** | Both ruling and recitation abrogated | (Few agreed examples) |

### Key Established Abrogations (Majority View)

| Earlier Ruling | Later Ruling | Subject |
|---------------|-------------|---------|
| 2:180 (bequest to parents mandatory) | 4:11-12 (fixed inheritance shares) | Inheritance |
| 2:240 (wives' waiting period: 1 year) | 2:234 (waiting period: 4 months 10 days) | Widows' iddah |
| 8:65 (1 Muslim vs 10 enemies) | 8:66 (1 Muslim vs 2 enemies) | Battle ratio |
| 33:50, 58:12 (private consultation with Prophet) | Abrogated by consensus | Private counsel |
| Qibla: Jerusalem | 2:144 (change to Mecca) | Prayer direction |

### Controversial / Disputed Abrogations

Scholars disagree on whether some verses abrogate others. Ibn al-Jawzi claimed ~20 abrogations; al-Suyuti reduced to ~5; modern scholars like al-Khuli and al-Ghazali questioned many. 

**Key principle:** Abrogation (naskh) should not be claimed without strong evidence. Apparent contradictions should first be reconciled (jam') before resorting to naskh. Hanafi scholars tend to identify more cases of naskh; Shafi'i and Maliki scholars fewer.

### Naskh Retrieval

```
Web search: "naskh al-quran [surah:ayah]" or "abrogated verses quran"
Key reference: Al-Nasikh wa al-Mansukh by Ibn al-Jawzi / Al-Suyuti
```

## Cross-Validation Rules

When presenting Islamic knowledge, verify across these axes before responding:

### Axis 1: Quran ↔ Quran
- Does the verse align with other verses on the same topic?
- Is there a chronological relationship (Meccan vs Medinan, abrogation)?
- Search: look up the topic in the topic-keyword mapping above; fetch multiple verses

### Axis 2: Quran ↔ Hadith
- Do authentic hadiths support, clarify, or restrict the verse?
- If a verse seems absolute (mutlaq), do hadiths qualify it (muqayyad)?
- Check: CDN grep the topic in Bukhari/Muslim → compare with Quran text

### Axis 3: Hadith ↔ Hadith
- Does this hadith contradict a stronger one? (shaadh vs mahfuz)
- Is the hadith in multiple collections? (muttafaqun 'alayh preferred)
- Check: search same topic in multiple collections; compare grading

### Axis 4: School ↔ School
- Do the four schools agree? (ijma') or differ? (khilaf)
- If they differ, what hadith/verse does each rely on?
- Check: cross-school comparative table above; web search `[topic] hanafi maliki shafii hanbali`

### Axis 5: Source Quality
- Quran.com API: authoritative for verse text; translations may vary in nuance
- Hadith CDN: authentic source texts; no grading metadata — verify grading separately
- Web search: quality varies; prefer sunnah.com, islamqa.info (supervised by scholars), dar-alifta.org
- Fatwa sites: match the madhhab to the user's context; prefer institutional over individual

### Verification Checklist

Before presenting any Islamic knowledge:

```
[ ] Verse reference: Surah:Ayah confirmed via API or index
[ ] Translation source: Ma Jain (56) or other — stated
[ ] Hadith grading: sahih/hasan/da'if — stated with source
[ ] School attribution: which madhhab's view — stated
[ ] Dissenting views: when significant khilaf exists — noted
[ ] Abrogation: if verse appears abrogated — checked and noted
[ ] Context: asbab al-nuzul relevant? — noted if yes
[ ] Boundaries: not presenting as fatwa unless explicitly from qualified source
```

### When to Refuse / Defer

| Scenario | Action |
|----------|--------|
| User asks for a definitive fatwa | Defer: "Consult a qualified scholar" |
| Complex new issue (crypto, AI ethics, etc.) | Present relevant principles + defer to institutional fatwa |
| Contradiction you can't resolve | Present both views with evidence, do not force a resolution |
| Medical/psychological question | "Islamic perspective" with disclaimer: consult both doctor and scholar |
| Political/sectarian dispute | Present factual positions without endorsing one side |
| User demands you pick a madhhab for them | Explain regional associations; let them choose |

## Five Pillars Quick Reference

| Pillar | Arabic | Description | Quranic Basis |
|--------|--------|-------------|---------------|
| Shahadah | الشهادة | Declaration of faith | 3:18, 47:19 |
| Salah | الصلاة | Five daily prayers | 2:238, 4:103, 17:78 |
| Zakat | الزكاة | Alms-giving (2.5%) | 2:277, 9:60 |
| Sawm | الصوم | Fasting in Ramadan | 2:183-185 |
| Hajj | الحج | Pilgrimage to Mecca | 3:97, 22:27-29 |

## Six Articles of Faith

| Article | Arabic | Brief |
|---------|--------|-------|
| Belief in Allah | الإيمان بالله | One God, 99 names |
| Belief in Angels | الإيمان بالملائكة | Jibril, Mikail, Israfil, etc. |
| Belief in Books | الإيمان بالكتب | Quran, Torah, Gospel, Psalms, Scrolls |
| Belief in Messengers | الإيمان بالرسل | 25 named in Quran, Muhammad final |
| Belief in Last Day | الإيمان باليوم الآخر | Resurrection, judgment, paradise, hell |
| Belief in Qadr | الإيمان بالقدر | Divine decree, good & bad |

## Honest Boundaries

- Fiqh rulings are presented as scholarly opinions, not divine edicts
- Differences between madhhabs are noted, not suppressed
- Chinese translations are human translations — nuances may differ from Arabic original
- Hadith grading follows majority scholarly consensus where available
- This skill is a reference tool, not a substitute for scholarly consultation
- Web search results vary by availability; always cite sources
- Quran.com API may have geo-restrictions; fall back to web search if needed
