# MuseDrive Security Reminder / 安全提醒

> **Read this before installing. Do not start installation until you have read it and explicitly confirmed it.**
> **安装前必读。没有读完并明确确认之前，不要开始安装。**

MuseDrive is an unofficial personal project. It is not affiliated with, endorsed by, or supported by Tesla.
MuseDrive 是非官方个人项目，与特斯拉无关，未获特斯拉认可或支持。

## 1. Only your own car, for personal use only / 只限自己的车、个人用

- Use this only on a vehicle you own, with your own Tesla account, for personal, non-commercial use.
- Do not use it on a vehicle you do not own or have not been explicitly authorized to control. Do not offer it as a service, do not collect other people's tokens, keys, or vehicle data, and do not use it for a commercial fleet.
- 只用于你自己的车、你自己的特斯拉账号、个人非商业用途。不要用在不属于你、没有明确授权的车上，不要对外提供服务，不要收别人的 token、密钥或车辆数据，不要用于商业车队。

## 2. Three accounts, the weakest one decides / 三个账号，最弱的那个决定安全

- Enable two-factor authentication on your Tesla account, your GitHub account, and the account running the AI assistant, preferably with an authenticator app rather than SMS alone. Store recovery codes offline, never in a repo or in chat.
- Regularly review third-party authorizations in your Tesla account and the key list in the car. Remove anything unfamiliar immediately.
- Tesla、GitHub、跑 AI 助手的账号，全部开双重验证，尽量用验证器 App 而不是只用短信，恢复码离线存好，不要放仓库、不要放聊天记录。
- 定期检查特斯拉账号里的第三方授权和车里的钥匙列表，看到不认识的，马上撤掉、删掉。

## 3. Secrets must never be shared / 密钥绝不能外泄

- Never publish or paste a private key, client secret, access token, refresh token, authorization code, VIN, license plate, or live location in a public repo, issue, screenshot, chat message, or forwarded message.
- Each person must register their own developer app, use their own domain, and generate their own key pair. Do not use a shared hosted app or someone else's key: whoever holds the private key and token can control the car within the approved scopes.
- If you suspect a leak, in this order: stop vehicle commands first, revoke the Tesla authorization, delete the virtual key in the car, change your passwords and re-check two-factor authentication, rotate or delete the developer credentials, and delete the local secret files.
- 私钥、client secret、access / refresh token、一次性授权码、车架号、车牌、实时位置，绝不要放进公开仓库、Issue、截图、聊天、转发消息。
- 每个人都必须用自己的开发者应用、自己的域名、自己的一对密钥，不要用别人托管好的现成应用：谁持有私钥和 token，谁就能在已批权限内控车。
- 怀疑泄露时按这个顺序处理：先停命令，再撤特斯拉授权、删车里虚拟钥匙、改密码并重查双重验证、换密钥、删本地 secret 文件。

## 4. Commands carry a real theft risk / 命令有真实的被盗风险

- A method that can unlock a car remotely creates a theft risk if misused or leaked. If an attacker obtains both your access token and your private key, they could unlock the car, open the trunk or frunk, turn off sentry mode, and, with remote start, potentially drive it, within the scopes you approved.
- Never leave valuables in the car. Always verify the car is locked again after any unlock. Keep sentry mode on when parked somewhere unfamiliar. Do not post your license plate, VIN, home or work location, or live location in demos or issues.
- Never send a command that changes the car's state while it is driving. Always verify the final state afterwards, and do not retry a failed command blindly: the first attempt may already have taken effect.
- 能远程开锁的方法，一旦被误用或泄露，就有被盗风险，而且是真实的：攻击者同时拿到 token 和私钥，就可能在已批权限内开锁、开后备箱、关哨兵、甚至用远程启动把车开走。
- 车里不要放贵重物品，每次开锁后都要确认重新锁好了，停在不熟悉的地方要开哨兵，演示和 Issue 里不要出现车牌、车架号、家里单位位置、实时位置。
- 车辆行驶时不要发任何改变车辆状态的命令，发完要核实最终状态，命令失败别盲目重发，第一次可能已经生效了。

## 5. Minimal scopes, light use, no surprise charges / 最小权限、轻量使用、别被收费打个措手不及

- Grant only the scopes you need. Vehicle information, location, and commands are the default; charging management is separate and should only be added if you explicitly need it.
- Do not poll at high frequency, monitor continuously, or run automated command loops. Excessive wakes, queries, or commands can incur charges, trigger rate limits, or get an app restricted or revoked.
- Tesla's China billing could not be fully verified as free, and Tesla may change pricing at any time. Do not add a payment method without your explicit approval.
- 只开你需要的最小权限：车辆信息、位置、车辆命令是默认，充电管理是单独的权限，没必要就别加。
- 不要高频轮询、连续监控、自动循环发命令，唤醒、查询、命令过多可能产生费用、触发限流、导致应用被限制或撤销。
- 中国区收费目前无法完全确认免费，特斯拉也随时可能改收费，没有你明确同意，不要绑支付方式。

## 6. If something looks wrong, treat it as a possible theft attempt / 发现异常，先当成可能的被盗企图处理

- An unfamiliar unlock, an unfamiliar key, or an unfamiliar third-party authorization is a warning sign. Check the car and its contents immediately, revoke the Tesla authorization, delete the virtual key, and change your passwords before using this method again.
- 不认识的开锁记录、不认识的钥匙、不认识的第三方授权，都是警告信号：马上检查车和车内物品，先撤授权、删钥匙、改密码，再考虑继续用。

## Disclaimer / 免责条款

This project is provided "as is", without warranty of any kind. The author is not liable for vehicle damage, loss, fines, charges, account restrictions, vehicle theft, break-in, or loss of property inside the car resulting from misuse, misconfiguration, a leaked credential, a mis-sent command, a failure to re-lock, or a third party's misuse. Tesla's current Developer Terms, Fleet API documentation, and acceptable-use rules override this project if they change. By installing and using MuseDrive, you accept all risks above.
本项目按“原样”提供，不附带任何保证。因误用、配错、凭证泄露、命令发错、没重新锁好、第三方滥用导致的车辆损坏、损失、罚款、费用、账号受限、车辆被盗、被撬、车内财物丢失，作者不承担责任。特斯拉现行开发者条款、Fleet API 文档和可接受使用规则如有变化，以特斯拉为准。安装并使用 MuseDrive，即表示你接受上面所有风险。

---

**Installation gate: an assistant installing MuseDrive for a user must present this reminder and the disclaimer in full, in the user's language, and must not start any installation step until the user has explicitly confirmed they have read it, understand it, and accept it. Presentation and confirmation must be two separate steps, not done in the same message or turn. A vague reply, a skipped step, or an implied consent does not count.**
**安装门：给用户安装 MuseDrive 的助手，必须先用用户语言完整告知本提醒和免责条款，并且在用户明确确认“已读、理解、接受”之前，不得开始任何安装步骤。告知和确认必须是两步，不能在同一条消息、同一轮里一边告知一边就开始安装。含糊的回复、跳过的步骤、默认同意，都不算确认。**
