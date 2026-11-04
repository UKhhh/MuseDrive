# MuseDrive for Tesla

[English](./README.md) | [中文](./README.zh-CN.md)

> **Unofficial personal project. Not affiliated with, endorsed by, or supported by Tesla.**
> **非官方个人项目，与特斯拉无关，未获特斯拉认可或支持。**

A working method for connecting your own Tesla to an AI assistant through Tesla's official Fleet API (China region), for read-only queries and signed vehicle commands. See `SKILL.md` for the full method.

## Support scope — 支持范围

> **Source: Tesla's official Fleet API / Vehicle Command project documentation.**
> **来源备注：特斯拉官方 Fleet 项目说明。**

- **Supported vehicles: Tesla Model S / 3 / X / Y.** Only models sold in China are listed here; models not sold in China are not listed. Model S / X from mid-2018 onwards are supported; older S / X only with an MCU2 retrofit. Only a Model 3 has been tested end-to-end in this project — other models are officially supported but project-untested, so verify every step on your own car.
- **Firmware / account:** Vehicle Command Protocol is required (firmware 2023.38+ requires explicitly requesting `location_data` for location). Tesla owner account only; a share account with `hide_private` set to `true` receives no location.
- **Region — China by default, other regions read this:** this project is written for the China region. Outside China, replace the China hosts with the global hosts throughout the steps — developer portal `developer.tesla.com` (not `.cn`), authorization/token host `auth.tesla.com`, Fleet API host `fleet-api.prd.na.vn.cloud.tesla.com` (not `fleet-api.prd.cn.vn.cloud.tesla.cn`), and the same token audience as the Fleet API host. Do not mix China and global hosts in one setup, and re-verify partner registration, OAuth, and pairing after switching hosts.
- **Other AIs:** yes, in principle — the method is plain markdown plus curl, Go, `tesla-control`, and a local browser for OAuth, so any AI that can read the whole skill, run terminal commands, store secrets in 600-permission files, and verify each step can follow it, after re-running the Step -1 safety gate. Muse-specific scheduling, memory, and browser do not transfer automatically, and tokens/keys must not be copied across AIs in chat — re-authorize for the new AI by default.

## Disclaimer — please read before use / 免责声明（使用前必读）

**This project is provided "as is", without warranty of any kind, and the author is not liable for any claim, damage, or other liability arising from its use. By using it, you accept all risks below.**

1. **Unofficial, personal, non-commercial only.** This is for your own vehicle, with your own Tesla account, for personal use only. Do not use it on a vehicle you do not own or have not been explicitly authorized to control. Do not offer it as a service to third parties, do not collect other people's tokens, keys, or vehicle data, and do not use it for a commercial fleet.
2. **Tesla's terms override this project.** Tesla, its logos, and its API names belong to Tesla. Follow Tesla's current Developer Terms, Fleet API documentation, and acceptable-use rules. Tesla may change, deprecate, or withdraw endpoints, scopes, pricing, or any monthly credit at any time, and may restrict or revoke a developer app. No continued access, approval, or free allowance is guaranteed, especially for the China region — see Costs below for what the China subscription page currently lists and why it is not a guarantee.
3. **Commands are your responsibility.** Before any command that changes the car's state, verify the car is parked (shift P, speed 0), unoccupied in a way that could create a hazard, and the surroundings are safe. Never send lock, unlock, remote-start, climate, trunk/frunk, or charging commands while the vehicle is driving. Always verify the final state afterwards. The author is not responsible for vehicle damage, loss, fines, charges, or account restrictions resulting from misuse or misconfiguration.
4. **Known limits and failure modes.** This method may fail, and failure is expected in normal use: the vehicle may be asleep (vehicle_data returns 408 until woken), a token may have expired (access tokens last 8 hours, refresh tokens about 3 months and rotate), a scope change requires fresh re-authorization, the cloud browser may fail OAuth even when registration is correct (use a local browser), the REST command endpoint returns 403 and only signed commands work, and network errors can burn a single-use authorization code. Do not retry a failed command blindly without checking whether the first attempt already took effect. Location has its own trap, corrected 2026-11-03: for firmware 2023.38+, a plain vehicle_data call returns no coordinates — `location_data` must be requested explicitly, and `vehicle_data_cached` never contains location (see Step 6 and the Updates section above).
5. **Foreseeable misuse risks.** Do not use this method to access another person's vehicle, to bypass Tesla security, to build a competing product, to sell personal data, to poll at high frequency, to monitor continuously, or to run automated command loops. Excessive wakes, queries, or commands can incur charges, trigger rate limits, or get an app restricted or revoked.
6. **Credentials are yours to protect.** A leaked private key, client secret, or token can allow control of the vehicle within the approved scopes. Never publish a private key, client secret, access token, refresh token, authorization code, or VIN in a public repo, chat message, screenshot, or forwarded message. If you suspect a leak, stop vehicle commands first, revoke the Tesla authorization, delete the virtual key in the car, rotate or delete the developer credentials, and delete the local secret files immediately.
7. **No warranty, no liability.** To the fullest extent permitted by law, the software and documentation are provided "as is", without warranties of merchantability, fitness for a particular purpose, or non-infringement. In no event shall the author be liable for any claim, damages, or other liability, whether in contract, tort, or otherwise, arising from the use of this project. This disclaimer does not remove liability that cannot be limited by law, and it does not replace your own duty to check safety, legality, and Tesla's current terms before each use.
8. **Vehicle safety and in-car property theft risk — this risk is real, read it carefully.** Any method that can unlock a car remotely creates a theft risk if it is misused or leaked, and that risk is not theoretical:
   - If an attacker obtains both your access token and your private key, they could unlock the car, open the trunk or frunk, turn off sentry mode, and, with remote start, potentially drive it — within the scopes you approved. The public key alone cannot do this, but the token plus the private key together can, which is why both must never be published or shared.
   - If location data leaks, an attacker could learn where the car is parked, when it is unattended, and when you are away from it, and use that to target the car or property inside it.
   - If you unlock remotely and do not verify that the car is locked again, or you unlock it in a public place while you cannot see it, property inside the car is exposed even without any attacker or leak.
   - Mitigations you must follow: never leave valuables in the car; always verify the car is locked again after any unlock; keep sentry mode on when parked somewhere unfamiliar; do not post your license plate, VIN, home or work location, or live location in demo videos, screenshots, or repo issues; do not share your Muse, Tesla, or GitHub login, and do not let another person send commands from your account. If you notice an unfamiliar unlock, an unfamiliar key, or an unfamiliar third-party authorization, treat it as a possible theft attempt: check the car and its contents immediately, revoke the Tesla authorization, delete the virtual key, and change your passwords before using the method again.
   - The author is not liable for vehicle theft, break-in, or loss of property inside the car, whether caused by a leaked credential, a mis-sent command, a failure to re-lock, or a third party's misuse of this method.

## Security reminder — read before installing / 安全提醒（安装前必读）

**Full text: [`SECURITY.md`](./SECURITY.md). Read it before installing. Do not start installation until you have read it and explicitly confirmed it.**

In short, and this summary does not replace the full reminder:

- Only your own vehicle, with your own Tesla account, for personal, non-commercial use. Do not use it on another person's car, do not offer it as a service, and do not collect other people's tokens or keys.
- Enable two-factor authentication on your Tesla, GitHub, and assistant accounts, and regularly review third-party authorizations and the key list in the car. Remove anything unfamiliar.
- Never publish or share a private key, client secret, token, authorization code, VIN, plate, or live location. Each person must use their own developer app, domain, and key pair — whoever holds the private key and token can control the car within the approved scopes.
- Remote unlock carries a real theft risk: never leave valuables in the car, always verify re-lock after any unlock, keep sentry on somewhere unfamiliar, and never send a state-changing command while driving.
- Grant only the scopes you need, do not poll at high frequency or monitor continuously, and do not add a payment method without explicit approval — China billing is not fully verified as free.
- If you see an unfamiliar unlock, key, or authorization, treat it as a possible theft attempt: check the car, revoke authorization, delete the virtual key, and change passwords before using this method again.

**Installation gate: an assistant installing MuseDrive for a user must present `SECURITY.md` and the disclaimer in full, in the user's language, and must not start any installation step until the user has explicitly confirmed they have read it, understand it, and accept it. This gate is now Step -1 in `SKILL.md`.**

## Updates — 更新说明 / 这个项目还在维护

This project is actively maintained.

**2026-11-04 — Support scope simplified and moved to the front, foreign-user hosts added, skill rename dropped:** support scope is now abbreviated (Tesla Model S / 3 / X / Y only, no models not sold in China), placed at the very front of the README, with a source note to Tesla's official Fleet project documentation; foreign users get a China-to-global host mapping (`.cn` to `.com` / global Fleet API host). The earlier skill rename idea is dropped — naming stays as originally published, `SKILL.md` unchanged.
- 支持范围已简化并移到最前面，国外用户地址映射已加，skill 改名已取消，仍按原来来。

**2026-11-04 — Support scope added / 新增支持范围：** supported vehicles and other-AI installability, per Tesla's official Fleet API documentation, in the new Support scope section above and in `SKILL.md`. Only Model 3 is project-tested; other models are officially supported but project-untested.
- 支持范围已按特斯拉官方 Fleet API 文档新增，见上面的支持范围一节和 `SKILL.md`：车型、固件、账号、地区，以及能否装到别的 AI 上。实测过的只有 Model 3，其他车型是官方支持、项目未测。

**2026-11-04 — Security reminder and installation gate / 安全提醒和安装门：**
- Added [`SECURITY.md`](./SECURITY.md), a standalone bilingual security reminder, and linked it from both READMEs.
- Added Step -1 to `SKILL.md`: an assistant must present the security reminder and disclaimer in full, in the user's language, and obtain explicit confirmation before any installation step. A vague reply, a skipped step, or implied consent does not count.
- 新增独立的中英双语安全提醒 [`SECURITY.md`](./SECURITY.md)，并在中英文 README 里都做了链接。
- `SKILL.md` 里新增第 -1 步安装门：助手必须先用用户语言完整告知安全提醒和免责条款，拿到用户明确确认后才能开始任何安装步骤，含糊回复、跳过步骤、默认同意都不算。
 When a method in it turns out to be wrong or incomplete, it is corrected here and in `SKILL.md`, with the date, instead of quietly leaving the old text in place.

**2026-11-03 — Location query corrected / 位置查询已修正：**
- Before this fix, the skill's read-only query examples did not mention `location_data`, so a reader following them would get battery, range, and tire pressure but no GPS coordinates at all. That is now corrected in `SKILL.md` Step 6.
- 在这次修正之前，技能里的只读查询例子没有提到 `location_data`，按那个写法走只能拿到电量、续航、胎压，就是拿不到 GPS 坐标。现在已在 `SKILL.md` 第 6 步修正。
- Root cause / 原因：for vehicles running firmware 2023.38 or later, Tesla's documentation requires `location_data` to be requested explicitly to fetch vehicle location; a plain `vehicle_data` call returns no location even when the `vehicle_location` scope is granted. Requesting it shows a location-sharing icon on the vehicle UI. The coordinates then appear in `drive_state`, while the `location_data` block itself may stay null — that is not a failure. `vehicle_data_cached` never contains location data, so it cannot be used for this. If location is still missing, check `granular_access.hide_private` in the vehicle list: `true` means no location even with the scope granted.
- 原因：固件 2023.38 及以上版本的车，按特斯拉文档必须显式请求 `location_data` 才能拿到位置；不加这个参数，哪怕给了 `vehicle_location` 权限，普通 `vehicle_data` 调用也不会返回位置。请求后车机上会显示位置共享图标。坐标出现在 `drive_state` 里，`location_data` 那一块本身可能还是空的，这不算失败。`vehicle_data_cached` 永远不含位置数据，不能用来查位置。如果加了参数还是没有位置，先查车辆列表里的 `granular_access.hide_private`：为 `true` 时即使有权限也不会给位置。
- No personal data was added by this fix: no coordinates, VINs, tokens, keys, or account details appear in this repo.

## What you can do — functions and how Muse links in / 能实现什么功能，以及怎么和 Muse 联动

### 1. Read-only queries — 只读查询（已经验证能用）

You ask Muse in normal chat, for example "电量多少" or "胎压怎么样". Muse checks the token, wakes the car if it is asleep, queries the Fleet API, converts units, and answers in your language. No command is sent to the car in this mode.

- Battery and range: battery level, usable level, rated range in kilometres, charging state
- Charging: charge limit, charge current setting, scheduled charging time, charge port open/locked state, energy added last session
- Location and driving state: GPS location, heading, shift state, speed, power — for firmware 2023.38+, location requires explicitly requesting the `location_data` endpoint (see Updates above and `SKILL.md` Step 6); a plain `vehicle_data` call returns no coordinates
- Lock and body state: locked or not, each door, frunk, trunk, and window open or closed, sentry mode, valet mode, dashcam state
- Tire pressure: all four pressures in bar, recommended pressures, soft/hard warnings, last-seen time
- Climate: inside and outside temperature, set temperatures, climate on or off, seat and steering-wheel heat
- Vehicle info: total mileage (odometer), software version, model and trim, exterior colour, wheels, driver-assist hardware, charge-port type
- Configuration and display: 24-hour time, km/hr, Celsius, Bar, rated-range display

### 2. Signed commands — 签名命令

You ask Muse in normal chat, for example "闪一下灯" or "开锁然后再锁上". Muse first checks the car is parked (shift P, speed 0), asks you to confirm any sensitive command, sends the signed command with `tesla-control`, and then re-queries to verify the final state before telling you it is done.

- **Verified working in this project:** flash lights, unlock, lock (unlock-then-lock tested with final state verified as locked and all doors closed)
- **Available in `tesla-control` but not yet tested in this project:** honk, climate on/off and set temperature, frunk/trunk open, remote start, sentry mode, media control, charge-port control
- **Charging commands, available only after a charging-scope re-authorization:** charging set limit, set amps, start, stop, and schedule. The old token does not gain this scope automatically; a fresh OAuth authorization with `vehicle_charging_cmds` is required first, and each charging change still needs your explicit per-action confirmation.

### 3. How Muse links in — Muse 是怎么联动的

Muse is the chat front-end and the scheduler, not a Tesla App replacement:

- **Chat control:** You talk to Muse in the same chat you are using now, in natural language. Muse translates your words into the right Fleet API query or signed command, and reports back in Chinese, with miles converted to kilometres and times converted to Beijing time.
- **Safety gate:** Low-risk actions (flash lights) run on your request. Sensitive actions (unlock, remote start, trunk/frunk, any charging change) always need your per-command confirmation, and Muse checks parked/safe state first and verifies the final state afterwards.
- **Scheduled automation (possible, not set up by default):** Because Muse can run scheduled tasks, it can combine car data with other information you already use, for example a morning report (weather + battery + range + tire pressure), a low-battery or low-tire-pressure alert, a pre-trip range check before a weekend drive, a parking-lot find (flash + honk), a pre-cool / pre-heat before departure, or a charge-complete notification. Each automation needs your explicit approval, a schedule, and a token that is still valid; none of them poll the car every minute.
- **Personalized morning briefing (optional, the most useful automation in this project):** Once a morning, at a time you choose, Muse wakes the car once, pulls battery, range, charging, tire pressure, lock state, and location in that single wake, combines them with that day's weather at the car's location, and gives you one short briefing with personalized car-use advice — not just raw numbers. For example: whether the range is enough for today's planned driving, whether to charge tonight (and whether the 23:00 scheduled charge is already set), whether the tires need air before departure, and whether the car is locked and sentry is on. You choose what goes in it (for example workday commute vs. weekend drive, your usual departure time, your preferred charge limit), and you can turn any part off. It runs once a morning, not continuously, so it costs one wake plus a few data calls, not minute-level polling.
- **Weather-linked tire-pressure alert (optional, on top of the briefing):** Tire pressure falls when temperature falls — roughly 0.1 bar for about every 10°C drop, measured cold — so a sudden cold snap can push an already-low tire further below recommended pressure without any leak. When Muse sees a forecast sharp temperature drop (for example an overnight drop of 8–10°C or more, or a morning temperature much lower than yesterday's), it checks tire pressure once (one wake, one vehicle_state query), and only alerts you if a tire is below its recommended pressure or a TPMS soft/hard warning is present. The alert says which tire, its pressure vs. recommended (for example 2.75 bar vs. 2.9 bar), that cold pressure should be measured before driving, and that you should inflate promptly and avoid high-speed long-distance driving until you do. If pressures are normal, it stays silent — no alert for a normal cold morning. This alert also needs your explicit approval and a threshold you choose; it does not run every hour and does not track the car continuously.
- **What Muse does not do:** Muse does not replace the Tesla App, does not control the car while it is driving, does not track the car continuously in real time, and does not control another person's vehicle.

## Before you start — preparation checklist / 准备工作清单

Do not start Step 1 until all of these are ready. Missing one will block a later step.

1. **Tesla owner account — 特斯拉车主账号**
   - The account that already owns the vehicle in the Tesla App. A non-owner account cannot authorize or pair.
   - Two-factor authentication on, and an authenticator app available for developer-portal sign-in.
   - The Tesla App installed on your phone, for the later virtual-key pairing step.
2. **GitHub account — GitHub 账号**
   - For hosting the public key and the OAuth callback page with GitHub Pages.
   - Two-factor authentication on (authenticator app plus a passkey backup, recovery codes stored offline).
   - Ability to create public repos and enable Pages. A free account works; private repos do not publish Pages on a free account, so the public-key and callback repos must stay public.
3. **A public domain you control — 一个你能控制的公开域名**
   - Either a GitHub Pages user site domain, or your own domain with HTTPS.
   - Tesla must be able to fetch the public key from it without login, at `/.well-known/appspecific/com.tesla.3p.public-key.pem`, and the OAuth callback page must be reachable from a normal browser.
4. **A machine to run commands on — 一台能跑命令的机器**
   - Either an AI assistant host with a terminal, a browser, and secure local file storage (for example Muse's cloud computer), or your own Mac / Linux / Windows machine.
   - It needs `curl`, a Go toolchain for building `tesla-control`, and a place to store a private key, a client secret, and tokens in 600-permission files only. A phone alone cannot do this step.
5. **A local browser — 本地浏览器**
   - Safari or Chrome on your own phone or PC, for the OAuth authorization step. Do not use the Tesla App's built-in browser, and do not use an AI assistant's cloud browser for this step — the cloud browser has repeatedly failed it.
6. **No payment, no third-party subscription needed to start — 开始不需要付费**
   - No payment method is added in this method, and no third-party subscription is required. See Costs below: Tesla's China page currently lists a $0 Discovery tier with daily limits, but calls it temporary, so keep usage light and check Tesla's current terms before scaling up.
7. **Time and patience — 时间**
   - This is not a one-click setup. Expect developer-app approval, key hosting, partner registration, OAuth, and vehicle pairing in that order, with a possible retry on the OAuth code (codes are single-use and short-lived).

## Costs — what this project actually costs / 费用 — 这个项目到底花多少钱

The author charges nothing for this project: it is MIT-licensed documentation and a skill file. The costs below are three separate bills, none of them paid to the author. Prices and limits change, so check Tesla's current pages before you scale up from one car and one morning briefing.

### 1. Tesla Fleet API — 特斯拉 Fleet API

Source checked 2026-11-03: Tesla's China subscription-plans page, https://developer.tesla.cn/docs/fleet-api/getting-started/subscription-plans

- **Currently listed plan: Discovery, $0 / vehicle / month**, monthly, auto-renewing until cancelled.
- **Daily limits per vehicle on that page:** device data 200 requests, commands 50 requests, wakes 15 requests, charging commands 5 requests.
- **Why this is not a promise of free forever:** the same page says this plan is temporary and will be replaced with different options in 2024. That wording is already stale, which means the page has not been kept up to date — treat $0 as the current listing, not as a guaranteed future price.
- **A second, different Tesla picture exists:** Tesla's global Billing and Limits page describes a pay-per-use model, with wakes as the most expensive category, data next, and commands lower, plus a $10 / month discount for individual developers and small applications, and a billing limit that defaults to $0 until a payment method is added. Whether a China-region account gets the same rates and the same $10 discount is **not** shown on the China subscription page, so do not budget on the global rates for a China account.
- **What this project actually uses:** one car, one morning briefing. That is one wake plus a few data calls per day — far below the Discovery daily limits above. In this project no payment method has been added and no Tesla API charge has been paid. High-frequency polling, continuous monitoring, or many cars would change that picture fast, because wakes are the expensive part.

### 2. GitHub — GitHub

- **$0 for what this project needs.** A free GitHub account is enough: public repositories and GitHub Pages are free, and those are exactly what this project uses to host the public key and the OAuth callback page.
- **No paid GitHub plan is required.** Private repositories do not publish Pages on a free account, so the public-key and callback repositories must stay public — that is a hosting requirement, not a reason to pay.
- If you later want private hosting, a custom domain, or a paid static host, that is your own optional cost, not a MuseDrive cost.

### 3. Muse and the machine that runs it — Muse 和运行它的机器

- **MuseDrive itself costs $0.** It is not a hosted service and it does not bill you.
- **If you run it on Muse:** the author of this project used Muse's free tier, with no subscription purchased. Muse has a free weekly allowance; if you exceed it, paid tiers exist, with prices shown in the Muse app and on the Muse website and varying by region — check the current pricing page rather than relying on a price quoted in a README, because it changes.
- **If you run it on your own machine:** there is no Muse fee, but you pay your own hardware and electricity costs and you become the maintainer.
- **Third-party Tesla apps are not required.** Third-party services are separate subscriptions if you choose them; this project does not need them, so do not add that cost unless you deliberately want that service.

### How to stay cheap — 怎么保持低成本

- Keep it to one wake per briefing and batch the data calls into that wake. Do not poll every few minutes.
- Do not add a payment method, another car, or a new automation without first re-checking Tesla's subscription and billing pages.
- If Tesla ever asks for payment or disables access for exceeding a limit, that is Tesla's billing decision, not a MuseDrive charge — stop, check the portal's usage and billing limit, and scale back before paying anything.

## How to let your local AI learn this project and start executing / 怎么让本地的 AI 学会这个项目并开始执行

This is a reminder, not a one-click install. A new AI cannot control your car from this repo alone — it needs the skill, the accounts, and your authorizations, in this order:

1. **Give the AI the skill file.** Clone this repo (or download `SKILL.md`) and put `SKILL.md` where your AI loads skills from. For Muse, that is the custom-skills folder. For other AI tools, use that tool's skill / plugin / instruction folder, or simply upload `SKILL.md` and tell the AI to follow it as its operating manual. Do not paste your private key, secret, token, or VIN into the prompt — the skill tells the AI to read those from 600-permission local files only.
2. **Tell the AI to start at Step 0, not at Step 4.** Say, in your own words: "Read SKILL.md and start from Step 0, developer-account registration, and ask me before every step that needs my login or confirmation." A good AI will first check the preparation list (Tesla owner account, GitHub account, public domain, a machine with curl and Go, a local browser) and stop if anything is missing, instead of jumping straight to a command.
3. **You do the human-only parts.** The AI cannot do these for you: Tesla developer-portal sign-in and agreement, GitHub repo and Pages setup, OAuth login and scope confirmation in your local browser, and virtual-key pairing in the Tesla App. When the AI gives you an authorize URL, open it in your local browser, confirm the app name and scopes, and send back the callback URL promptly — codes are single-use and short-lived.
4. **Let the AI verify, step by step.** After each stage, the AI should verify before moving on: app shows valid, public-key URL returns 200 and matches, partner registration returns 200, token exchange succeeds, vehicle list returns your car, wake works, a read-only query works, pairing is confirmed, and a signed command (for example flash lights) exits 0 and is verified afterwards. If any stage fails, stop and fix that stage; do not skip to commands.
5. **Check the AI can actually execute, not just chat.** If your local AI has no terminal, no browser, no secure local file storage, and no scheduled tasks, it can only help you write code — it cannot run queries, sign commands, or send a morning briefing by itself. In that case you need a host that does have those (for example Muse's cloud computer, or your own Mac/Linux machine with a backend), and the AI is the front-end that talks to that host.
6. **Set the briefing only after the basics work.** Personalized morning briefings and weather-linked tire alerts are Step 8-style automations, not Step 0. Set them only after read-only queries and at least one signed command are verified, and only with your explicit schedule and thresholds.

## What SKILL.md does — what work the skill actually covers / SKILL.md 大致做了哪些工作

`SKILL.md` is the playbook an AI assistant follows when it helps you with this project. It does not contain your secrets, and it does not control your car by itself. In short, it does these jobs:

1. **Decides when to use it:** battery, range, charging, tire pressure, location, lock state, mileage, climate, and vehicle configuration queries; flash, honk, lock, unlock, climate, trunk/frunk, remote start, and charging commands; scope explanations and command-failure diagnosis. It refuses other people's vehicles, commercial fleets, and unapproved scopes.
2. **Sets the safety rules:** confirm sensitive commands, check parked/safe state before state-changing commands, verify the final state afterwards, keep scopes minimal, and never publish a private key, secret, token, code, or VIN.
3. **Fixes the China endpoints:** developer.tesla.cn, auth.tesla.cn, and the China Fleet API host and audience, and warns that the global hosts do not contain a China-registered app.
4. **Walks through the 8 setup steps in order (Step 0–7):** developer-account registration, developer app, key pair and public-key hosting (GitHub Pages is only an example host; any public HTTPS host works, and the `.nojekyll` requirement applies only to GitHub Pages), partner registration before OAuth, local-browser OAuth, token refresh and rotation, read-only queries (semicolon endpoints, wake on 408, tire-pressure fields, miles-to-km conversion), and signed commands with `tesla-control` (including the Go build location and the 403 REST endpoint trap).
5. **Handles tokens and scope changes:** access 8 hours, refresh about 3 months and rotating, and a fresh authorization after any scope change.
6. **Records the hard-won pitfalls:** cloud-browser OAuth failure, single-use codes burned by a lost network response, comma-separated endpoints returning empty sections, and urllib failures on this VM (prefer curl).
7. **Covers accounts and costs:** a three-account security checklist (Muse, Tesla, GitHub, plus Apple ID as separate), and costs as described in the README Costs section — no payment method without your approval.

## What this covers

- China developer-account registration and developer app registration (developer.tesla.cn), minimal scopes, and app approval
- EC key pair generation, public-key hosting, and partner registration before OAuth. Public-key hosting with GitHub Pages is only an example — any public HTTPS host works (Cloudflare Pages, Netlify, Vercel, a VPS or home server, or any other static host you control), as long as Tesla can fetch the key from `/.well-known/appspecific/com.tesla.3p.public-key.pem` without login. The GitHub Pages `.nojekyll` requirement applies only to GitHub Pages.
- OAuth in the user's local browser, token refresh and rotation, and scope-change re-authorization
- Read-only queries: battery, range, charging state, tire pressure, location, lock state, mileage, climate, and vehicle configuration
- Signed vehicle commands with Tesla's official `tesla-control` tool (flash lights, honk, lock, unlock, climate, trunk/frunk, charging commands where the charging scope is approved)

## What this does not do

- It does not provide a hosted service, a server, or an app that controls other people's cars.
- It does not contain, and must never contain, any private key, client secret, token, authorization code, VIN, or personal domain.
- It does not guarantee that Tesla will approve, keep, or continue to support any developer app.

## Skills library — what is in `skills/` / 技能库说明

The `skills/` folder is an optional collection of reusable skills. It is **not** required for the core MuseDrive method, and a skill in it is **not** auto-installed: copy the skill file to your own assistant and fill in its customization worksheet before first use.

**How the library is maintained:** the owner merges new skills; other people contribute by fork and pull request, or by opening an Issue for the owner to review. Every collected skill must be generic, contain no personal data, tokens, keys, VINs, or account details, and be optional and safe by default. New entries are added to the list below with their upload date, so the list itself is the changelog.

**Currently included (as of 2026-11-03, one skill):**

- **2026-11-02 — Daily Morning Briefing / 每日早报** — `skills/daily-morning-briefing/SKILL.md` (English), `skills/daily-morning-briefing/SKILL.zh-CN.md` (Chinese). One short, warm, first-person morning message that reads like a good friend, not a report: today's weather and clothing advice, read-only vehicle status (battery, range, charging, tire pressure vs. recommended, lock state, inside/outside temperature), last night's sleep and yesterday's activity if synced, and personalized reminders woven into the chat — plus at most one urgent news item, only when one truly matters. No bold section headings, no counted suggestion lists, conclusions only. The vehicle part requires your own MuseDrive setup; without a car, the same skill can run weather plus health only.

**Not counted as an optional skill:** the core MuseDrive playbook, root `SKILL.md`, published 2026-11-02. That is the method itself, not part of the optional collection.

**Format for future entries:** `YYYY-MM-DD — Skill name — one-line description — file path`. When a new skill is uploaded, add one line here in both the English and Chinese READMEs on the same day.

## Author / 作者

- Douyin / 抖音：神探夏洛克𝕏（抖音号：zayshzh）
- Demo videos for this project may be posted on that Douyin account. For setup and safety details, please use this repo, not private messages.

## License

MIT License. See `LICENSE`.
