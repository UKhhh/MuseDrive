---
name: tesla-fleet-integration
description: Connect a personal Tesla to Muse through Tesla's official Fleet API (China region) for read-only queries and signed vehicle commands. Use when the user asks about their Tesla's battery, range, tire pressure, location, charging, or wants Muse to flash lights, lock/unlock, run climate, or add a scope such as charging management. Covers China developer-account registration, the China developer app, GitHub Pages public-key hosting, partner registration, local-browser OAuth, token refresh, and tesla-control signed commands.
---

# Tesla Fleet API integration (China)

Project: MuseDrive for Tesla. Brand name is MuseDrive; "for Tesla" is a descriptive subtitle. This is an unofficial personal project, not affiliated with or endorsed by Tesla.

This skill records the working method, not a specific account. Never put a client secret, private key, access token, refresh token, authorization code, VIN, or callback URL containing a personal domain into this file, chat, a browser task, or a public repo. Personal values live only in the user's developer portal and in 600-permission files under `~/workspace/tesla-fleet/`.

## Step -1: Safety gate — mandatory before any installation (main-skill entry gate)

This gate applies to the main skill file, `SKILL.md` in this project (the main-skill entry point; see the naming note below). It is not optional, and it is not a formality.

Before Step 0, before any developer-account, app, key, domain, OAuth, token, or pairing step, and before asking the user for any login, code, secret, or callback URL, the assistant MUST:

1. Present the full security reminder in `SECURITY.md` and the disclaimer in the README, in the user's own language, in full — not a one-line summary, not a link alone.
2. Ask the user to explicitly confirm, in their own words, that they have read it, understand it, and accept it, including: this is an unofficial personal project not affiliated with Tesla; it is for their own vehicle and personal use only; a leaked private key plus token can allow control of the car within the approved scopes; they will keep all secrets private and use their own developer app, domain, and key pair; and they accept the project as-is with no warranty.
3. Wait for that explicit confirmation. If the user declines, answers vaguely, asks to skip it, or gives only implied consent, STOP. Do not start Step 0, do not create an app, do not generate a key, do not ask for a login or authorization code, and do not proceed on the assumption that they will confirm later.

Only after an explicit confirmation may the assistant move to the preparation checklist and Step 0. Record that confirmation happened in the conversation; do not record the user's secrets, and do not treat a previous session's confirmation as confirmation for a new user or a new vehicle.

Naming note: the main skill entry file must stay named `SKILL.md`, because skill loaders discover a skill by that exact filename. Renaming it to `main-skill.md` would break auto-discovery, so the main-skill identity is carried by the project and by this gate instead: this file IS the main skill, and any copy, fork, or install package should be labelled `main-skill` (for example a `main-skill` folder or a `main-skill.zip`) while keeping `SKILL.md` inside it unchanged.

## Support scope — supported vehicles and other AIs

Follow Tesla's official Fleet API and Vehicle Command documentation for this section. This skill uses only `vehicle_data` queries and signed commands via `tesla-control`; it does not use Fleet Telemetry streaming.

- Supported by the official Vehicle Command Proxy: Model 3 (all years), Model Y (all years), Model S / X from mid-2018 onwards, and Cybertruck (officially supported, not tested in this project). Older Model S / X only work with an MCU2 retrofit; without it they use the old Owner API and do not fit this method. Pre-2021 Model S / X must be checked individually via `fleet_status` / `vehicle_command_protocol_required`, because some newer flows exclude them. Semi is officially listed for the proxy but out of scope for this personal project. Other or future models are unsupported until Tesla lists them.
- Firmware: vehicles delivered from November 2023 only support the Vehicle Command Protocol, and from January 2024 all vehicles (except the pre-2021 Model S / X exclusion in Tesla's announcement) require it. Firmware 2023.38+ requires explicitly requesting `location_data` for location.
- Account: Tesla owner account only, vehicle bound in the Tesla App, mobile access enabled, virtual key paired. A share account with `granular_access.hide_private` set to `true` receives no location even with the location scope granted.
- Region: this skill is written and tested for the China region (developer.tesla.cn, auth.tesla.cn, fleet-api.prd.cn.vn.cloud.tesla.cn). Global hosts are analogous but untested here; do not swap hosts blindly.
- Tested vs. officially supported: only a Model 3 has been tested end-to-end in this project. Other models above are officially supported but project-untested; verify every step on the user's own car.
- Other AIs: yes, in principle. The method is plain markdown plus curl, Go, `tesla-control`, and a local browser for OAuth, so any AI that can read the whole skill, run terminal commands, store secrets in 600-permission files, ask the user to complete OAuth locally, and verify each step can follow it. It does not work for a phone-only AI with no terminal or secure storage, or an AI that cannot keep secrets out of chat/repos or will skip Step -1. Muse-specific scheduling, memory, goals, and browser do not transfer automatically; another AI must re-implement them and re-run Step -1. Do not copy tokens or keys across AIs in chat; re-authorize for the new AI by default, or transfer 600-permission files only through a secure private channel. Each user must register their own developer app, domain, and key pair and pass Step -1 themselves.

## Before you start — preparation checklist

Do not start Step 0 until all of these are ready, and do not start Step 0 at all until the Step -1 safety gate above has been explicitly confirmed. Missing one will block a later step.

- Tesla owner account, with the vehicle already bound in the Tesla App, two-factor authentication on, and the Tesla App on the user's phone for pairing.
- GitHub account, with two-factor authentication on, able to create public repos and enable Pages (private repos do not publish Pages on a free account).
- A public domain the user controls (a GitHub Pages user site domain or their own HTTPS domain) that Tesla can fetch without login.
- A machine to run commands on (an assistant host with terminal, browser, and secure local storage, or the user's own Mac/Linux/Windows machine) with curl and a Go toolchain; a phone alone cannot do this step.
- A local browser (Safari or Chrome on the user's own phone or PC) for OAuth; do not use the Tesla App browser or an assistant's cloud browser for this step.
- No payment method and no third-party subscription is needed to start; keep usage light until Tesla's current China billing terms are confirmed.

## Safety tips and disclaimer

- This method is for your own vehicle, with your own Tesla account, for personal, non-commercial use only. Never use it on a vehicle you do not own or have not been explicitly authorized to control, and never offer it as a service to third parties.
- This skill is unofficial and is not affiliated with, endorsed by, or supported by Tesla. Tesla, its logos, and its API names belong to Tesla. Follow Tesla's current Developer Terms, Fleet API documentation, and acceptable-use rules; if they change, they override this skill.
- Do not abuse the API: no high-frequency polling, no continuous minute-level monitoring, no bulk data collection, and no automated command loops. Excessive wakes, queries, or commands can incur charges, trigger rate limits, or get the developer app restricted or revoked.
- Commands are the user's responsibility. Before any command that changes the car's state, verify the car is parked and the surroundings are safe. Never send lock, unlock, remote-start, climate, trunk, or charging commands while the vehicle is driving or occupied in a way that could create a hazard, and always verify the final state afterwards.
- Keep credentials private. A leaked private key, client secret, or token can allow control of the vehicle within the approved scopes. If you suspect a leak, revoke the Tesla authorization, delete the virtual key in the car, rotate or delete the developer credentials, and delete the local secret files immediately.
- Tesla may change, deprecate, or withdraw Fleet API endpoints, scopes, pricing, or the monthly credit at any time, and may require re-approval after a scope change. This skill does not guarantee continued access, approval, or a free allowance, especially for the China region where billing could not be verified.
- Use this skill at your own risk. The author provides the method as-is, without warranty, and is not liable for vehicle damage, loss, fines, charges, or account restrictions resulting from misuse, misconfiguration, or Tesla policy changes.
- Vehicle safety and in-car property theft risk: any method that can unlock a car remotely creates a theft risk if misused or leaked. If an attacker obtains both the access token and the private key, they could unlock the car, open the trunk or frunk, turn off sentry mode, and, with remote start, potentially drive it within the approved scopes. Location leaks can reveal where the car is parked and when it is unattended. Mitigations: never leave valuables in the car, always verify re-lock after any unlock, keep sentry on when parked somewhere unfamiliar, never post plate, VIN, home/work location, or live location in demos or issues, and treat any unfamiliar unlock, key, or third-party authorization as a possible theft attempt — check the car, revoke authorization, delete the virtual key, and change passwords before using the method again. The author is not liable for vehicle theft, break-in, or loss of property inside the car.
- Known limits and failure modes, disclosed so a reader cannot claim they were hidden: the vehicle may be asleep (vehicle_data returns 408 until woken), tokens expire (access 8 hours, refresh about 3 months and rotating), a scope change requires fresh re-authorization, the cloud browser may fail OAuth even when registration is correct (use a local browser), the REST command endpoint returns 403 and only signed commands work, and a lost network response can burn a single-use authorization code. Do not retry a failed command blindly without checking whether the first attempt already took effect. Location has its own trap, corrected 2026-11-03: for firmware 2023.38+, a plain vehicle_data call returns no coordinates — `location_data` must be requested explicitly (see Step 6), and `vehicle_data_cached` never returns location at all.
- Foreseeable misuse risks, explicitly not endorsed: accessing another person's vehicle, bypassing Tesla security, building a competing product, selling personal data, high-frequency polling, continuous monitoring, or automated command loops. Any of these can incur charges, trigger rate limits, or get an app restricted or revoked.

## Account security checklist (check before and after setup)

This integration touches three accounts, and the vehicle is only as secure as the weakest one. The assistant cannot verify an account's two-factor status from chat; the user must check each account's own security settings and confirm the result.

- Muse account: enable two-factor authentication in the Muse / Meta account security settings, preferably with an authenticator app rather than SMS alone. Keep a backup sign-in method. Never share the Muse login, and never paste a Muse sign-in code into another service.
- Tesla account: keep two-factor authentication on (Tesla sign-in already requires an authenticator-style verification for the developer portal and for sensitive actions). Review third-party authorizations and paired vehicle keys regularly, and remove anything unfamiliar.
- GitHub account: enable two-factor authentication in Settings -> Password and authentication, preferably with an authenticator app plus a passkey as a backup method. Store recovery codes offline, never in a repo and never in chat. Review Settings -> Sessions, Authorized OAuth Apps / GitHub Apps, and Personal Access Tokens, and revoke anything unused or unfamiliar.
- If the user also uses an Apple ID for related devices or accounts, its two-factor status is separate and must be checked in Apple ID settings; do not assume it from the Tesla or GitHub status.
- Confidentiality rules for all accounts: passwords, authenticator codes, client secrets, private keys, access and refresh tokens, and callback URLs carrying a fresh authorization code must never be pasted into chat, a public repo, a screenshot, or a forwarded message. Codes are for the single sign-in or exchange step that requested them, and then they are spent.
- If any account is suspected to be compromised, stop vehicle commands first, revoke that account's sessions and third-party access, change its password, re-check two-factor authentication, and only then re-authorize the Fleet API app with a fresh token.

## What Muse does with this skill — functions and linkage

- Chat queries (verified): battery, range, charging state and limit, location, lock/door/window state, tire pressure, odometer, climate temperatures, vehicle configuration, and software version.
- Chat commands (verified): flash lights, unlock, lock. Available in tesla-control but not yet tested: honk, climate on/off and set temperature, frunk/trunk, remote start, sentry, media, charge-port, and charging-* commands (charging commands require the charging scope and a fresh token after re-authorization).
- Linkage: the user talks to Muse in natural language; Muse translates the request into a Fleet API query or a signed command, checks parked/safe state, confirms sensitive commands, executes, and verifies the final state. Scheduled automations (morning report, low-battery/tire alerts, pre-trip range check, parking find, pre-cool/pre-heat, charge-complete notification) are possible through Muse's scheduled tasks, but only with the user's explicit approval, and none of them poll every minute.

## Personalized briefing and weather-linked tire-pressure alert (optional automation)

These are optional, require the user's explicit approval and chosen schedule and thresholds, and run at most once per morning or on a weather event — never minute-level polling.

- Personalized morning briefing: at a user-chosen morning time, wake the car once, pull charge_state, vehicle_state, climate_state, drive_state, and location in that single wake, combine them with that day's weather at the car's location, and give one short briefing with personalized car-use advice, not just raw numbers: range vs. today's planned driving, whether to charge tonight and whether scheduled charging is already set, whether tires need air before departure, and whether the car is locked and sentry is on. The user chooses the contents (workday commute vs. weekend drive, usual departure time, preferred charge limit) and can turn any part off.
- Weather-linked tire-pressure alert: tire pressure falls with temperature, roughly 0.1 bar for about every 10°C drop, measured cold. On a forecast sharp temperature drop (for example an overnight drop of 8–10°C or more, or a morning temperature much lower than yesterday's), check tire pressure once (one wake, one vehicle_state query). Alert only if a tire is below its recommended pressure or a TPMS soft/hard warning is present, naming the tire, its pressure vs. recommended, that cold pressure should be measured before driving, and that the user should inflate promptly and avoid high-speed long-distance driving until they do. If pressures are normal, stay silent. Do not run this check every hour, and do not use it to track the car continuously.

## How a new AI should onboard with this skill

If a user gives this skill to a new AI, that AI should: read the whole skill first, run the Step -1 safety gate (present `SECURITY.md` and the disclaimer in full, in the user's language, and obtain explicit confirmation) before anything else, then check the preparation checklist, and only then start at Step 0. Ask the user before every step that needs their login, confirmation, or a callback URL. Verify each stage before moving on (app valid, public-key URL 200 and matching, partner registration 200, token exchange succeeds, vehicle list works, wake works, read-only query works, pairing confirmed, signed command exits 0 and is verified). Do not jump to commands, do not paste secrets into chat, and do not set up briefings or alerts until read-only queries and at least one signed command are verified.

## When to use

- The user asks for their Tesla's battery, range, charging state, tire pressure, location, lock state, or other vehicle data.
- The user asks Muse to send a vehicle command: flash lights, honk, lock, unlock, climate, trunk/frunk, remote start, or a charging command.
- The user asks to add, remove, or explain a Fleet API scope, or asks why a command fails.
- Do not use this for another person's vehicle, for a commercial fleet, or for any scope the user has not approved.

## Safety rules

- Confirm with the user before any sensitive command: unlock, remote start, opening a trunk/frunk, or any charging change. Flash lights and honk are low-risk but still only send them when the user asked for that test or action.
- Before unlock or any command that changes the car's state, check that the car is parked (shift state P, speed 0) and report the final state afterwards (for example, re-check that it is locked again after an unlock-then-lock test).
- Scopes are minimal by default: vehicle information, vehicle location, and vehicle commands. Charging management is a separate scope and is not included unless the user explicitly asks for it.
- The public key may be published. The private key, client secret, tokens, and authorization codes must never be published, pasted into chat, or written into a repo, a browser task, or memory.
- Authorization codes are single-use and short-lived. If an exchange's network response is lost, that code is burned; get a fresh code from the user, do not retry the old one.
- Revocation is always available: remove the third-party authorization in the Tesla account, delete the virtual key in the car, and delete the local token, secret, and key files.

## China endpoints (do not substitute the global hosts)

- Developer portal: developer.tesla.cn
- Authorization and token host: https://auth.tesla.cn/oauth2/v3/
- Fleet API host: https://fleet-api.prd.cn.vn.cloud.tesla.cn
- Token audience for both partner tokens and user tokens: https://fleet-api.prd.cn.vn.cloud.tesla.cn
- The global hosts (auth.tesla.com, fleet-api.prd.na.vn.cloud.tesla.com) do not contain a China-registered app; a China app failing on the global host proves nothing about the China app.

## Step 0: Developer-account registration (before any app)

This step is separate from creating an app, and it is the part most guides skip. Do not skip it: without a developer account, there is nowhere to create an app.

- Go to the China developer portal, developer.tesla.cn, and sign in with the user's existing Tesla owner account. Sign-in goes through Tesla's China SSO (auth.tesla.cn) and will ask for an authenticator-style verification code. Never ask for, repeat, or store that code; it is for that single sign-in only.
- Agree to the Fleet API agreement when prompted. Read the current agreement in the portal; if its terms change, they override this skill.
- Choose the registration type for personal use only ("仅适用于我" / personal use, single owner, single vehicle), not a business or fleet registration, unless the user explicitly asks for a business setup. A personal registration matches this project's non-commercial scope.
- A developer account alone gives no car access. It only creates the account under which Step 1's app is created, and under which Step 3's partner registration is later verified.
- If sign-in, agreement, or registration fails, stop and report the portal's exact error. Do not create a second Tesla account, do not use another person's account, and do not proceed to app creation on a different account.

## Step 1: Developer app

- After Step 0, create a personal-use app in the China portal, using the same owner account from Step 0.
- Give the app an English name and a plain description of personal use: a single owned vehicle, occasional battery/range/location queries, and occasional commands, no commercial use, no high-frequency polling, no third-party sharing. A non-English app name may be rejected as invalid by the portal; if it is, switch to an English name and resubmit, keeping the same description and scopes.
- Choose only the scopes the user approved. Scope names: vehicle_device_data (vehicle information), vehicle_location (vehicle location), vehicle_cmds (vehicle commands), vehicle_charging_cmds (charging management, only on request).
- Choose the OAuth grant type for authorization code (and machine-to-machine where the portal offers it) as the portal presents it for a personal app.
- Register an allowed origin (bare domain only, no path, no trailing slash — the portal's origin field accepts a domain only) and an allowed redirect URI (the callback page, full HTTPS URL ending in the callback file, no trailing slash), plus an allowed return URL with the same callback address. Keep every registered entry; adding a lowercase-domain variant is safer than replacing an existing one, because OAuth checks the redirect URI character-for-character against the registration.
- First submission may be rejected with "source, redirect, or return URI invalid". If it is, do not change the app's purpose or scopes: re-check origin (bare domain only), redirect URI (exact callback URL), and return URL (same callback URL), re-save the origin in its lowercase domain form if needed, and resubmit. A successful registration shows the app as valid ("有效") in the dashboard, and Tesla sends an approval email to the owner.
- Get the app's client_id from the app details page. Get the client_secret from the app's credentials tab, and save it immediately into a 600-permission file under `~/workspace/tesla-fleet/` only. Never print the secret, never paste it into chat, a browser task, a repo, or memory, and never regenerate it unless the user asks — regenerating invalidates the old secret.
- App approval does not by itself give Muse access to the car. Partner registration, OAuth, and vehicle pairing must still happen, in that order.

## Step 2: Key pair and public-key hosting

- Generate an EC prime256v1 key pair locally. Store the private key in a 600-permission file under `~/workspace/tesla-fleet/` and never publish it.
- Host the public key at the domain root, at `/.well-known/appspecific/com.tesla.3p.public-key.pem`.
- GitHub Pages is only one example of a host, not a requirement. Any public HTTPS host works, as long as Tesla can fetch the public key from that exact path without login and the callback page is reachable from a normal browser: for example a GitHub Pages user site, Cloudflare Pages, Netlify, Vercel, a VPS or home server running Nginx, or any other static host you control. Choose the host you already have; do not set up GitHub Pages just because this skill uses it as its example.
- If the host you choose is GitHub Pages, add an empty `.nojekyll` file at the repo root, or GitHub Pages will not publish dot-directories and the public-key URL will 404. Other hosts do not need this file.
- Verify the public-key URL returns 200 and its content matches the generated public key exactly before proceeding.

## Step 3: Partner registration (before OAuth)

- Get a partner token with grant_type=client_credentials at the China token host, using the app's client_id and client_secret (read the secret from its 600 file, never print it) and the China Fleet API audience above.
- Register the bare domain with `POST https://fleet-api.prd.cn.vn.cloud.tesla.cn/api/1/partner_accounts` and the partner token as bearer. A 200 response confirms the app, domain, and public key.
- If the global token host returns client_not_found, that means the app lives only in the China database; switch to the China host above, do not treat it as a registration failure.

## Step 4: OAuth in the user's local browser (not the cloud browser)

- Build the authorize URL on auth.tesla.cn with the app's client_id, the registered redirect URI, response_type=code, the approved scopes plus openid and offline_access, and a state value the user can recognize.
- Give the URL to the user to open in their own phone or PC browser, never in the Tesla App and never in Muse's cloud browser. The cloud browser has repeatedly failed this step with a redirect_uri error even when the registration is correct; the local browser is the working path.
- The user logs in, confirms the app name and scopes, and sends back the callback URL. Extract the code from it without ever echoing the code in chat, memory, or files.
- Exchange the code immediately with curl (see the network note below) at the China token host, grant_type=authorization_code, with the same client_id, secret from the 600 file, redirect URI, and China audience. Save only the token response in a 600 file. Do not print tokens.

## Step 5: Tokens

- Access tokens last 8 hours (expires_in 28800). Refresh tokens last about 3 months and rotate: each refresh returns a new refresh token and invalidates the old one, so always overwrite the saved token response and never reuse an old refresh token.
- When an access token has expired, refresh it before querying; do not ask the user to re-authorize unless the refresh token itself has expired, been revoked, or the scopes changed.
- Adding a scope requires editing the app in the portal (the app may stay "valid" but will prompt that the vehicle owner must re-consent) and then a fresh OAuth authorization with the new scope. The old token does not gain the new scope automatically.

## Step 6: Read-only queries

- Vehicle list: `GET https://fleet-api.prd.cn.vn.cloud.tesla.cn/api/1/vehicles` with the access token. Use the vehicle id from this list for vehicle_data calls.
- Vehicle data: `GET /api/1/vehicles/{id}/vehicle_data?endpoints=...` with semicolon-separated endpoints, for example `charge_state;drive_state;climate_state` or `vehicle_state`. Comma-separated endpoints can return empty state sections. When combining endpoints, URL-encode the semicolons as `%3B`.
- Location — read this before troubleshooting a missing GPS position (corrected 2026-11-03, see the update note in the README): for vehicles running firmware 2023.38 or later, a plain vehicle_data call returns no location at all, even with the `vehicle_location` scope granted. You must explicitly request the `location_data` endpoint, for example `vehicle_data?endpoints=location_data` or `vehicle_data?endpoints=location_data%3Bcharge_state%3Bdrive_state%3Bvehicle_state`. Requesting it shows a location-sharing icon on the vehicle UI, per Tesla's documentation. Note two traps: the coordinates then appear in `drive_state` (`latitude`, `longitude`, `heading`, `shift_state`, `speed`), while the `location_data` block itself may stay null — that is not a failure. And do not use `vehicle_data_cached` for location: it never contains `drive_state` or location data. If location is still missing after requesting `location_data`, check the vehicle list first: a vehicle share with `granular_access.hide_private` set to `true` receives no location even when `vehicle_location` is granted, and an owner account should show `hide_private` as `false`.
- If the vehicle is offline, vehicle_data returns 408. Wake it with `POST /api/1/vehicles/{id}/wake_up`, wait, and retry.
- Tire pressure fields live in vehicle_state: tpms_pressure_fl/fr/rl/rr, tpms_rcp_front_value, tpms_rcp_rear_value, and tpms soft/hard warnings. Report pressures in bar against the recommended values, and note the tpms last-seen time; it may be older than the query time.
- Battery range from the API is in miles; convert to kilometres before reporting it to a user who expects kilometres, and say so.

## Step 7: Signed vehicle commands

- Do not use the REST command endpoint (`POST /api/1/vehicles/{id}/command/...`). It returns 403 "Tesla Vehicle Command Protocol required" for current vehicles.
- Commands must be signed with the private key using Tesla's official tesla-control tool, built from https://github.com/teslamotors/vehicle-command. The tool is tesla-control, with commands such as flash-lights, honk, lock, unlock, climate-on, climate-off, frunk-open, and charging-* (charging commands require the charging scope).
- Build it with Go, keeping the Go toolchain, module cache, and build temp directory under `~/workspace/`, not /tmp: /tmp on this VM is a small tmpfs and a Go build can fill it. Save the built binary under `~/workspace/tesla-fleet/tesla-control` (mode 700) so it survives a VM restart; a binary left in /tmp will not.
- Run it with `-key-file` pointing at the 600 private-key file, `-token-file` pointing at a temporary file containing only the plain access token (600, delete it after the command), `-vin` from the vehicle list (pass it as a shell variable, never print it), `-disable-session-cache`, and generous timeouts. A successful command exits 0 with no output; verify the result afterwards with a vehicle_data query where possible.
- Vehicle pairing is a prerequisite for signed commands: the user adds the virtual key in the Tesla App via https://tesla.com/_ak/<domain>. Pairing before a successful OAuth authorization fails with an "not yet authorized" error; pairing must come after OAuth. Do not consider pairing done until the user confirms it in the App or a signed command succeeds.

## Network notes for this VM

- Prefer curl over Python urllib for Tesla API calls through this VM's egress proxy; urllib has repeatedly failed here with RemoteDisconnected or IncompleteRead, while curl succeeds. A lost token-exchange response burns a single-use code, so always exchange codes with curl, immediately.
- If a vehicle list or vehicle_data call fails once with a connection error, retry once with curl before reporting a failure; do not retry a failed command blindly without checking whether the first attempt already took effect.

## Costs

- Fleet API costs are described in the README Costs section: Tesla's China subscription page currently lists a Discovery tier at $0 / vehicle / month with daily limits (data 200, commands 50, wakes 15, charging commands 5), but calls it temporary with a stale 2024 replacement note, so do not promise a free allowance for China. Tesla's global billing page separately describes pay-per-use pricing and a $10 / month individual-developer discount, which is not shown as applying to China accounts. No payment method should be added without the user's explicit approval.
- GitHub Pages and public repos used for the public key and callback page are free. This integration itself does not require a third-party subscription.
