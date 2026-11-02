# Optional Skills Collection / 可选技能合集

This folder collects optional, shareable skills for MuseDrive users.
They are **not** required for the core MuseDrive method in `SKILL.md`
at the repository root. You can use MuseDrive for read-only queries
and signed commands without any skill in this folder.

这个文件夹收集的是 MuseDrive 用户可以选用的、可分享的 skill。
它们**不是**核心 MuseDrive 方法的必需部分。不用这里的任何 skill，
照样可以用根目录的 `SKILL.md` 查车、控车。

## Skills inventory / 技能清单

As of 2026-11-03, this library contains one skill only:

截至 2026-11-03，本技能库仅收录 1 个技能：

- **2026-11-02 — Daily Morning Briefing / 每日早报** — `skills/daily-morning-briefing/SKILL.md` (English) and `skills/daily-morning-briefing/SKILL.zh-CN.md` (Chinese). One short, warm, first-person morning message with today's weather, read-only vehicle status, last night's sleep and yesterday's activity if synced, and personalized reminders, plus at most one urgent news item. The vehicle part requires your own MuseDrive setup; without a car, the same skill can run weather plus health only.
- **2026-11-02 — 每日早报 / Daily Morning Briefing** — `skills/daily-morning-briefing/SKILL.md`（英文版）、`skills/daily-morning-briefing/SKILL.zh-CN.md`（中文版）。一份短的、温暖的、第一人称的早报，包含当天天气、只读的车辆状态、昨晚睡眠和昨天活动（如果已同步）、个性化提醒，紧急新闻最多一条。车辆部分需要你自己的 MuseDrive 搭建；没有车的话，同一个技能也可以只跑天气加健康。

## How to use / 怎么用

1. Pick a skill folder below, read its `SKILL.md` (and
   `SKILL.zh-CN.md` if you prefer Chinese).
2. Fill in that skill's Customization Worksheet / 定制表 with your
   own time, places, vehicle, health source, and thresholds.
3. Give the filled-in skill to your AI assistant and follow its
   instructions. Do not paste private keys, tokens, secrets, or
   vehicle identifiers into a skill file or a prompt.

1. 从下面选一个 skill 文件夹，读它的 `SKILL.md`（想看中文就读
   `SKILL.zh-CN.md`）。
2. 按那个 skill 文末的定制表，填上自己的时间、地点、车辆、
   健康来源和提醒阈值。
3. 把填好的 skill 交给你的 AI 助手，按里面的说明执行。不要把
   私钥、token、密钥、车辆标识放进 skill 文件或提示词里。

## Skills included so far / 目前收录的技能

### 1. Daily Morning Briefing / 每日早报

- Folder: `skills/daily-morning-briefing/`
- English: `skills/daily-morning-briefing/SKILL.md`
- Chinese: `skills/daily-morning-briefing/SKILL.zh-CN.md`
- What it does: one short morning message, in the first person like
  a good friend talking, covering weather, vehicle status, health,
  and personalized reminders — plus one urgent news item only when
  one truly matters. Vehicle status is read-only, maintenance is
  mentioned only when its trigger is met, the briefing gives
  conclusions only (no internal thinking process), and wording-only
  changes edit the previous draft instead of re-running all data
  queries.
- 作用：每天早上用第一人称、好朋友的语气发一条短短的早报，
  包含天气、车辆状态、健康和顺口提醒，特别重要紧急的新闻才
  附一条。车辆只读不控，保养只在触发时说一句，早报只给结论、
  不写思考过程，改措辞就在原稿上改、不重新跑数据。

## Contributing a new skill / 怎么收录新的技能

Later, other people may have better ideas. A new skill can be
collected here if it meets all of these rules:

以后如果有人有更好的点子，也可以收录进来，但必须同时满足
下面这些规则：

1. **Generic, not personal.** No user names, cities tied to one
   person, vehicle identifiers, account details, tokens, keys,
   secrets, or private data. Use placeholders and a Customization
   Worksheet instead.
   **通用，不带个人信息。**不能有某个人的名字、城市、车辆标识、
   账号、token、密钥、隐私数据，要用占位符加定制表。
2. **Optional.** It must work as an add-on. It must not change the
   core MuseDrive method, its disclaimer, or its safety rules.
   **可选。**只能是附加玩法，不能改核心方法、免责条款和安全规则。
3. **Safe by default.** No vehicle commands, no high-frequency
   polling, no continuous tracking, no medical diagnosis, and no
   collecting other people's data, unless the skill itself says
   those need separate explicit confirmation.
   **默认安全。**不发车辆命令、不高频轮询、不持续追踪、不做医疗
   诊断、不收集别人的数据，除非 skill 里自己写明这些需要另外
   单独确认。
4. **Bilingual if possible.** Provide both an English `SKILL.md`
   and a Chinese `SKILL.zh-CN.md` when you can, so more people can
   use it.
   **尽量双语。**能做到的话，同时提供英文 `SKILL.md` 和中文
   `SKILL.zh-CN.md`，方便更多人用。
5. **One folder per skill.** Put each skill in its own folder
   under `skills/`, for example `skills/your-skill-name/`.
   **一个技能一个文件夹。**每个技能单独放在 `skills/` 下的一个
   文件夹里，例如 `skills/your-skill-name/`。

To propose a skill, open an issue or a pull request with the skill
folder, and say what it does, what it needs from the user, and what
it deliberately does not do.

想提议收录，就开一个 issue 或 pull request，附上 skill 文件夹，
并说清它能做什么、需要用户提供什么、以及它故意不做什么。
