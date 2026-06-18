# <img width="100" height="100" alt="logo (2)" src="https://github.com/user-attachments/assets/724f9942-52e9-4cc1-89aa-f83d48be5e18" />  Scriptora Developer Platform

> **Your own KeyAuth — self-hosted, MongoDB-backed, and built right into Scriptora.**
> Protect your software behind a login wall in minutes. No database setup, no server management.

---

## 📑 Table of Contents

1. [What is it?](#-what-is-it)
2. [Getting Access](#-getting-access-developer-slots)
3. [Core Concepts](#-core-concepts)
4. [Panels](#-panels)
5. [Users](#-users)
6. [License Keys](#-license-keys)
7. [Panel Variables](#-panel-variables)
8. [Panel Managers](#-panel-managers)
9. [Auth Endpoints](#-authentication-endpoints)
10. [New Auth Features](#-new-auth-features)
11. [Auth Logs & Sessions](#-auth-logs--sessions)
12. [File Downloads](#-file-downloads)
13. [Blacklist](#-blacklist)
14. [Webhooks](#-webhooks)
15. [Admin Panel UI Guide](#-admin-panel-ui-guide)
16. [Full API Reference](#-full-api-reference)
17. [ID & Key Formats](#-id--key-formats)
18. [Rate Limits & Security](#-rate-limits--security)

---

## 💡 What is it?

The **Scriptora Developer Platform** lets you protect your software — desktop apps, scripts, bots, APKs, cheats — behind a login wall. Your app calls one HTTP endpoint; Scriptora checks the credentials and returns a session token + variables. Zero infrastructure overhead.

**Perfect for:**
- 🎮 Cracked-game patchers, Roblox exploits, cheat menus
- 🤖 Discord bots and automation tools
- 📦 Selling software licenses (one-time, timed, multi-seat)
- 🔒 Gating file downloads behind payment or invite-only access
- 👥 Giving team members scoped panel access

---

## 🎟️ Getting Access (Developer Slots)

Every developer gets **2 free panel slots on first sign-in** — no payment required to get started.

- **Slot** = permission to own one panel.
- **First 2 slots** are free for life — just sign in with Discord.
- **Extra slots** can be purchased via Razorpay through the Developer tab (₹100/slot).
- A super admin can also grant you additional slots from the SuperAdmin panel.

> 💬 Sign in at **[scriptora.shubhamos.com](https://scriptora.shubhamos.com)** with Discord, then head to the **Admin panel → Developer tab**.

---

## 🧠 Core Concepts

| Concept | Description |
|---|---|
| 🖥️ **Panel** | Your "app". One panel = one product. Has a unique Panel ID and API key. |
| 👤 **User** | A person who authenticates into your panel with username + password. |
| 🔑 **License Key** | `XXXX-XXXX-XXXX-XXXX` format — an alternative to username/password. |
| 📦 **Variable** | Key-value pair stored on your panel, returned on every successful auth. |
| 🤝 **Manager** | A Discord user you trust to help manage your panel. |
| 🖱️ **HWID Lock** | Ties an account to a specific machine on first auth. |
| ⏳ **Subscription Expiry** | Each user/key can have an expiry date. After that, auth returns `subscription_expired`. |
| 🚫 **Blacklist** | Block specific HWIDs or values from ever authenticating. |
| 🎣 **Webhook** | Named HTTP callbacks fired on auth events. |
| 💸 **Revenue** | Per-panel revenue stats set by the Scriptora super admin. Visible in your developer dashboard. |

---

## 🖥️ Panels

A panel represents one application or product.

| Field | Details |
|---|---|
| `panel_id` | `PNL-` + 16 uppercase hex chars. **Public** — safe to embed in your app. |
| `api_key` | `shubhamos-` + 48 hex chars. **Keep this secret.** |
| `name` | Display name shown in the admin UI. |
| `version` | Current app version string (e.g. `1.0.3`). |
| `status` | `active` or `maintenance`. Maintenance mode rejects all auths. |
| `webhook_url` | Optional URL that receives a POST on every auth event. |
| `max_users` | `0` = unlimited. Otherwise, user creation is capped. |

**🔄 Rotating the API key:** Click "Rotate Key" in the admin panel. The old key is immediately invalidated.

---

## 👤 Users

Users authenticate with username + password. Each user belongs to exactly one panel.

### Fields

| Field | Details |
|---|---|
| `username` | Unique within the panel (case-sensitive). |
| `password` | Min 6 characters. Stored as scrypt hash — never exposed via API. |
| `hwid` | Hardware ID set on first auth. `null` = not locked yet. |
| `is_active` | `1` = enabled, `0` = disabled. |
| `expiry_date` | ISO 8601. If set and past, auth returns `subscription_expired`. |
| `metadata` | Arbitrary JSON — store plan tier, referral code, etc. |
| `last_auth_at` | Timestamp of last successful auth. |

### ⏳ Subscription Expiry

- Set expiry when creating a user or extend via the **+30d** button.
- Programmatic extension: `POST /developer/my/panels/:panelId/users/:username/extend`

### 🖱️ HWID Locking

- On first auth with a `hwid` field, the value is saved.
- All future auths **must** supply the same HWID.
- Reset via the ↺ icon in the admin UI or the reset-hwid endpoint.

---

## 🔑 License Keys

Keys are an alternative login method — no username or password needed. Great for one-time activations, giveaway codes, and invite-only access.

### Format

```
XXXX-XXXX-XXXX-XXXX   (e.g. A3KZ-7YQP-MNCF-B19T)
```

### Fields

| Field | Details |
|---|---|
| `key` | The key string itself. |
| `is_active` | `1` = enabled, `0` = disabled. |
| `max_uses` | `0` = unlimited. Otherwise capped at this number. |
| `uses_count` | How many times the key has been used. |
| `hwid` | Locked to a hardware ID after first use. |
| `expiry_date` | ISO date. After this date the key is rejected. |
| `assigned_username` | If the key was used to create a user, holds that username. |
| `note` | Free-text label (e.g. `"YouTube giveaway batch 1"`). |

### Generating Keys

In the **Keys tab** of your panel, click **Generate**:

- **Count** — up to 50 per batch
- **Max Uses** — `0` for unlimited, `1` for single-use
- **Expiry Date** — optional expiry for the batch
- **Note** — label to identify the batch

---

## 📦 Panel Variables

Variables are key-value pairs stored on your panel and **returned on every successful auth**. Use them to push dynamic config to your app without redeploying.

### Example Uses

| Key | Value |
|---|---|
| `version` | `1.2.0` |
| `discord_url` | `https://discord.gg/your-server` |
| `download_url` | `https://scriptora.shubhamos.com/api/download/TOKEN` |
| `motd` | `"Server maintenance Sunday 2AM UTC"` |
| `feature_dark_mode` | `true` |

Variables appear in the auth response:

```json
{
  "success": true,
  "token": "SESSION_TOKEN",
  "variables": {
    "version": "1.2.0",
    "download_url": "https://..."
  }
}
```

Manage them from the **Variables tab** — add, inline-edit, delete anytime.

---

## 🤝 Panel Managers

Managers are Discord users who can help run your panel without being the owner.

### Permissions

| Permission | What it allows |
|---|---|
| `users` | View, add, disable, delete users |
| `keys` | View, generate, toggle, delete license keys |
| `hwid` | Reset HWID for users and keys |
| `logs` | View auth logs |
| `variables` | View, add, edit, delete panel variables |
| `sessions` | View active sessions |

> ⚠️ Managers **cannot** rotate the API key, delete the panel, or modify other managers.

Add a manager by their Discord email in the **Managers tab**. The panel appears in their developer dashboard automatically.

---

## 🔐 Authentication Endpoints

All endpoints are under:
```
https://scriptora.shubhamos.com/api
```

### Standard Auth (username + password)

```
POST /developer/auth
```

**Request:**
```json
{
  "panelId":  "PNL-XXXXXXXXXXXXXXXX",
  "apiKey":   "shubhamos-...",
  "username": "johndoe",
  "password": "hunter2",
  "hwid":     "DESKTOP-ABC123",
  "version":  "1.2.0"
}
```

**Success `200`:**
```json
{
  "success":    true,
  "token":      "SESSION_TOKEN_UUID",
  "expiresAt":  "2026-06-19T20:27:00.000Z",
  "user": {
    "username":    "johndoe",
    "is_active":   1,
    "expiry_date": null,
    "hwid":        "DESKTOP-ABC123",
    "metadata":    {}
  },
  "variables": {
    "download_url": "https://...",
    "version":      "1.2.0"
  }
}
```

**Error Codes:**

| HTTP | `error` | Meaning |
|---|---|---|
| 400 | `missing_fields` | Required fields missing |
| 401 | `invalid_api_key` | Wrong API key |
| 404 | `panel_not_found` | Panel ID doesn't exist |
| 403 | `maintenance` | Panel under maintenance |
| 401 | `invalid_credentials` | Wrong username or password |
| 403 | `account_disabled` | User is disabled |
| 403 | `subscription_expired` | User's expiry has passed |
| 403 | `hwid_mismatch` | Different hardware ID than locked one |
| 403 | `version_mismatch` | App version doesn't match panel version |
| 403 | `blacklisted` | HWID is on the blacklist |

### License Key Auth

```
POST /developer/auth/license
```

Replace `username`/`password` with `licenseKey`:

```json
{
  "panelId":    "PNL-XXXXXXXXXXXXXXXX",
  "apiKey":     "shubhamos-...",
  "licenseKey": "A3KZ-7YQP-MNCF-B19T",
  "hwid":       "optional"
}
```

---

## ✨ New Auth Features

These endpoints match the KeyAuth feature set:

### 🆕 Register with Key

```
POST /developer/auth/register
```

Creates a new user by redeeming a license key. The key is consumed on success.

```json
{
  "panelId":    "PNL-...",
  "apiKey":     "shubhamos-...",
  "username":   "newuser",
  "password":   "securepass",
  "licenseKey": "A3KZ-7YQP-MNCF-B19T",
  "hwid":       "optional"
}
```

Returns the same response as standard auth on success.

---

### ⬆️ Upgrade with Key

```
POST /developer/auth/upgrade
```

Extends an existing user's subscription by redeeming an additional key.

```json
{
  "panelId":  "PNL-...",
  "apiKey":   "shubhamos-...",
  "username": "johndoe",
  "password": "hunter2",
  "licenseKey": "XXXX-XXXX-XXXX-XXXX"
}
```

---

### ✅ Check Session

```
POST /developer/auth/check
```

Validate a session token returned from a previous auth.

```json
{
  "panelId": "PNL-...",
  "apiKey":  "shubhamos-...",
  "token":   "SESSION_TOKEN_UUID"
}
```

Returns `{ "valid": true }` or `{ "valid": false, "reason": "expired" }`.

---

### 🚫 Check Blacklist

```
POST /developer/auth/checkblack
```

Check if a specific HWID or value is on the blacklist.

```json
{
  "panelId": "PNL-...",
  "apiKey":  "shubhamos-...",
  "hwid":    "DESKTOP-ABC123"
}
```

---

### 🔨 Ban / Disable User

```
POST /developer/auth/ban
```

Disable a user by username (requires API key).

```json
{
  "panelId":  "PNL-...",
  "apiKey":   "shubhamos-...",
  "username": "badactor"
}
```

---

### 📝 Log a Custom Event

```
POST /developer/auth/log
```

Write a custom entry to the panel's auth log.

```json
{
  "panelId":  "PNL-...",
  "apiKey":   "shubhamos-...",
  "username": "johndoe",
  "action":   "file_downloaded",
  "pcuser":   "DESKTOP-ABC123\\John"
}
```

---

### 📌 Set User Variable

```
POST /developer/auth/setvar
```

Store a per-user key-value pair server-side.

```json
{
  "panelId":  "PNL-...",
  "apiKey":   "shubhamos-...",
  "username": "johndoe",
  "varKey":   "high_score",
  "varValue": "99000"
}
```

---

### 📖 Get User Variable

```
POST /developer/auth/getvar
```

Retrieve a per-user variable.

```json
{
  "panelId":  "PNL-...",
  "apiKey":   "shubhamos-...",
  "username": "johndoe",
  "varKey":   "high_score"
}
```

Returns `{ "success": true, "value": "99000" }`.

---

### 📥 Download File

```
POST /developer/auth/download
```

Get a signed download URL for a panel file (authenticates the user first).

```json
{
  "panelId":  "PNL-...",
  "apiKey":   "shubhamos-...",
  "username": "johndoe",
  "password": "hunter2",
  "fileId":   "FILE-ID-HERE"
}
```

Returns `{ "success": true, "url": "https://...download/TOKEN" }`.

---

### 🎣 Fire Named Webhook

```
POST /developer/auth/webhook
```

Trigger a named webhook stored in your panel.

```json
{
  "panelId":     "PNL-...",
  "apiKey":      "shubhamos-...",
  "webhookName": "purchase_notify",
  "data":        { "amount": 299, "plan": "premium" }
}
```

---

## 📋 Auth Logs & Sessions

Every auth attempt (success or failure) is recorded in `auth_logs`.

**Fields logged:** `username`, `success`, `reason` (on failure), `ip`, `hwid`, `timestamp`.

**Sessions** — successful auths create a **4-hour session token**. Use it to validate subsequent requests without re-authenticating. Check validity with `POST /developer/auth/check`.

Visible in the **Logs** and **Sessions** tabs of your panel.

---

## 📁 File Downloads

Upload protected files (exe, apk, zip) and generate secure download URLs.

- **Max file size:** 100 MB per file
- Files are stored privately — not accessible by guessing paths
- Upload via **Admin → Developer → [Panel] → Files tab**
- Generated URL: `/api/download/TOKEN`
- Store the URL as a panel variable (`download_url`) so it's returned to authenticated users automatically
- View and delete all uploaded files from the admin panel

> 📷 Post images are separate — 15 MB limit.

---

## 🚫 Blacklist

Block specific HWIDs (or any string value) from ever authenticating.

- Auth with a blacklisted HWID returns `403 blacklisted`
- Manage from **Admin → Developer → [Panel] → Blacklist tab**
- Or check programmatically with `POST /developer/auth/checkblack`

---

## 🎣 Named Webhooks

Store named webhook URLs per panel. Fire them from your app or from server-side events.

| Action | Description |
|---|---|
| Add webhook | Admin → Developer → [Panel] → Webhooks tab → Add |
| Fire webhook | `POST /developer/auth/webhook` with `webhookName` |
| Update URL | Edit from the Webhooks tab |
| Delete | Remove from the Webhooks tab |

Each webhook receives a POST with the payload you send in `data`.

---

## 🛠️ Admin Panel UI Guide

Navigate to **[scriptora.shubhamos.com/shubhamosadmin](https://scriptora.shubhamos.com/shubhamosadmin)** → sign in with Discord → open the **Developer** tab.

### Panel List (left sidebar)
Shows all your panels with user count and status badge. Click a panel to select it.

### Panel Stats
After selecting a panel you'll see:

| Stat | Description |
|---|---|
| **Users** | Total user count (with cap if set) |
| **Sessions** | Currently active session tokens |
| **Auths** | Successful auth count |
| **Total Revenue** | Set by super admin — total earnings for your app |
| **This Week** | Set by super admin — earnings this week |

### Inner Tabs

| Tab | Contents |
|---|---|
| **Users** | List, add, toggle, reset HWID, extend subscription, delete users |
| **Keys** | Generate batches, filter/search, toggle, reset HWID, delete |
| **Vars** | Add/edit/delete panel variables inline |
| **Mgrs** | Add managers, set permissions toggles, remove |
| **Blacklist** | View and remove blacklisted HWIDs |
| **Sessions** | Active session tokens with IP and expiry |
| **Logs** | Auth history — 🟢 success / 🔴 failure with reason |
| **Files** | Upload and manage protected downloadable files |
| **Webhooks** | Named webhook URLs |
| **📖 Docs** | Live code snippets for Python / C# / C++ / Go / JS / Java / curl |
| **✍️ Post** | Rich-text post associated with the panel (shown on the public Scriptora page) |

---

## 📡 Full API Reference

Base URL: `https://scriptora.shubhamos.com/api`

### 🌐 Public Endpoints

| Method | Path | Description |
|---|---|---|
| `POST` | `/developer/auth` | Standard username/password auth |
| `POST` | `/developer/auth/license` | License key auth |
| `POST` | `/developer/auth/register` | Register new user with a key |
| `POST` | `/developer/auth/upgrade` | Extend subscription with a key |
| `POST` | `/developer/auth/check` | Validate session token |
| `POST` | `/developer/auth/checkblack` | Check if HWID is blacklisted |
| `POST` | `/developer/auth/ban` | Disable a user (requires API key) |
| `POST` | `/developer/auth/log` | Write a custom log entry |
| `POST` | `/developer/auth/setvar` | Set a per-user variable |
| `POST` | `/developer/auth/getvar` | Get a per-user variable |
| `POST` | `/developer/auth/download` | Get signed download URL |
| `POST` | `/developer/auth/webhook` | Fire a named webhook |
| `GET`  | `/developer/panel-info/:panelId` | Public panel metadata |

### 🔑 Developer Endpoints (Discord login required + developer slot)

| Method | Path | Description |
|---|---|---|
| `GET` | `/developer/my/slots` | Get slot usage |
| `POST` | `/developer/my/slots/buy` | Create Razorpay order for extra slot |
| `POST` | `/developer/my/slots/verify` | Verify slot payment |
| `GET` | `/developer/my/panels` | List owned + managed panels |
| `POST` | `/developer/my/panel` | Create a new panel |
| `PUT` | `/developer/my/panel` | Update panel settings |
| `POST` | `/developer/my/panel/rotate-key` | Rotate API key |
| `GET` | `/developer/my/panels/:panelId/users` | List users |
| `POST` | `/developer/my/panels/:panelId/users` | Create user |
| `PUT` | `/developer/my/panels/:panelId/users/:username` | Update user |
| `DELETE` | `/developer/my/panels/:panelId/users/:username` | Delete user |
| `POST` | `/developer/my/panels/:panelId/users/:username/reset-hwid` | Reset HWID |
| `POST` | `/developer/my/panels/:panelId/users/:username/extend` | Extend subscription |
| `GET` | `/developer/my/panels/:panelId/keys` | List keys |
| `POST` | `/developer/my/panels/:panelId/keys` | Generate key batch |
| `PUT` | `/developer/my/panels/:panelId/keys/:key` | Toggle key active |
| `DELETE` | `/developer/my/panels/:panelId/keys/:key` | Delete a key |
| `DELETE` | `/developer/my/panels/:panelId/keys` | Delete all keys |
| `POST` | `/developer/my/panels/:panelId/keys/:key/reset-hwid` | Reset key HWID |
| `GET` | `/developer/my/panels/:panelId/variables` | List variables |
| `POST` | `/developer/my/panels/:panelId/variables` | Create variable |
| `PUT` | `/developer/my/panels/:panelId/variables/:varKey` | Update variable |
| `DELETE` | `/developer/my/panels/:panelId/variables/:varKey` | Delete variable |
| `GET` | `/developer/my/panels/:panelId/managers` | List managers |
| `POST` | `/developer/my/panels/:panelId/managers` | Add manager |
| `PUT` | `/developer/my/panels/:panelId/managers/:email` | Update permissions |
| `DELETE` | `/developer/my/panels/:panelId/managers/:email` | Remove manager |
| `GET` | `/developer/my/panels/:panelId/sessions` | List active sessions |
| `GET` | `/developer/my/panels/:panelId/logs` | Auth log history |
| `GET` | `/developer/my/panels/:panelId/stats` | Panel stats |
| `GET` | `/developer/my/panels/:panelId/revenue` | Panel revenue (set by superadmin) |
| `GET` | `/developer/my/panels/:panelId/blacklist` | View blacklist |
| `POST` | `/developer/my/panels/:panelId/blacklist` | Add to blacklist |
| `DELETE` | `/developer/my/panels/:panelId/blacklist/:value` | Remove from blacklist |
| `DELETE` | `/developer/my/panels/:panelId/blacklist` | Clear blacklist |
| `GET` | `/developer/my/panels/:panelId/user-vars/:username` | List user variables |
| `DELETE` | `/developer/my/panels/:panelId/user-vars/:username/:varKey` | Delete user variable |
| `GET` | `/developer/my/panels/:panelId/files` | List panel files |
| `DELETE` | `/developer/my/panels/:panelId/files/:fileId` | Delete a file |
| `GET` | `/developer/my/panels/:panelId/webhooks` | List named webhooks |
| `POST` | `/developer/my/panels/:panelId/webhooks` | Create named webhook |
| `PUT` | `/developer/my/panels/:panelId/webhooks/:name` | Update webhook URL |
| `DELETE` | `/developer/my/panels/:panelId/webhooks/:name` | Delete webhook |

---

## 🧩 ID & Key Formats

| Type | Format | Example |
|---|---|---|
| Panel ID | `PNL-` + 16 uppercase hex | `PNL-3F9A1C72B40E8D56` |
| API Key | `shubhamos-` + 48 hex | `shubhamos-a1b2c3...` |
| License Key | `XXXX-XXXX-XXXX-XXXX` | `A3KZ-7YQP-MNCF-B19T` |
| Session Token | UUID v4 (4-hour lifetime) | `f47ac10b-58cc-4372-...` |

---

## 🛡️ Rate Limits & Security

| Protection | Details |
|---|---|
| 🚦 **Auth rate limit** | 20 requests / minute per IP (prevents brute force) |
| 🔐 **API key** | Keep server-side or inside obfuscated binaries. Never expose in client-side JS. |
| 🖱️ **HWID locking** | Strongly recommended for desktop apps to prevent account sharing |
| 🕐 **Session tokens** | 4-hour lifetime. Validate with `/auth/check` before sensitive operations. |
| 🎣 **Webhooks** | Use a secret header to verify authenticity on your receiving server |
| 👥 **Managers** | Principle of least privilege — only grant permissions a manager actually needs |
| 🚫 **Blacklist** | Block abusive HWIDs permanently with one click |

---

> 🏠 **Admin Panel:** [scriptora.shubhamos.com/shubhamosadmin](https://scriptora.shubhamos.com/shubhamosadmin)
> 📖 **Docs:** [scriptora.shubhamos.com/developer-docs](https://scriptora.shubhamos.com/developer-docs)
> 💬 **Discord:** Contact Jishan for access or support
