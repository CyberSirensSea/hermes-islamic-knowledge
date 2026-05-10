# Hermes 伊斯兰知识技能

[Hermes Agent](https://github.com/NapthaAI/hermes-agent) 的综合伊斯兰知识参考技能——命令行 AI 助手的古兰经、圣训、教法知识库。

## 涵盖内容

- **古兰经** — 114 章完整索引（阿拉伯语/中文/英文名称、降示地点、节数、核心主题），集成 Quran.com API v4 实时经文查询与中文翻译
- **圣训** — 六大部圣训集（Kutub al-Sittah）及补充典籍；圣训等级体系（健全/良好/羸弱）；CDN 检索；搜索方法论
- **教法学（Fiqh）** — 四大逊尼学派（哈乃斐、马立克、沙斐仪、罕百里）及比较教法；法源学原理；教法准则（qawa'id fiqhiyyah）；断法五类
- **经注学（Tafsir）** — 主要经注著作、方法论分类、降示背景（asbab al-nuzul）
- **交叉验证框架** — 信息呈现前的多轴验证（古兰经↔古兰经、古兰经↔圣训、圣训↔圣训、学派↔学派）

## 安装

```bash
mkdir -p ~/.hermes/skills/knowledge/islamic-knowledge
cp SKILL.md ~/.hermes/skills/knowledge/islamic-knowledge/
```

Hermes 会自动发现该技能。当你询问任何伊斯兰相关话题时，技能会自动激活。

## 使用示例

安装后直接自然提问：

- "古兰经第2章255节是什么？" — 通过 Quran.com API 获取经文及中文翻译
- "关于求知，有什么圣训？" — 搜索圣训集并附等级评定
- "四大教法学派对利息的立场？" — 跨学派比较教法
- "雅辛章有什么尊贵？" — 章节信息及相关圣训
- "古兰经和圣训对饮酒的立场一致吗？" — 古兰经与圣训交叉验证

## 特性

- 默认输出语言：简体中文
- 实时 API 集成（Quran.com v4，无需 API Key）
- 圣训 CDN 英文文本 + 联网搜索中文译本
- 跨学派教法比较表
- 诚实边界：区分学者意见与天启律法，标注学派分歧
- 伊斯兰知识呈现前的验证清单

## 技能元数据

- **名称**：islamic-knowledge
- **分类**：knowledge
- **默认语言**：简体中文
- **使用 API**：Quran.com v4、MuslimSalat.com、Hadith CDN (jsDelivr)

## 许可证

MIT — 详见 [LICENSE](./LICENSE)
