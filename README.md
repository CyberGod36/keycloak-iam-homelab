# 🔐 Keycloak IAM Homelab

**Identity & Access Management lab built with Keycloak (v26.x) — covering realm administration, user lifecycle, RBAC, MFA, and application integration (OIDC & SAML).**

> Built by Deon | Complements my [Okta IAM Homelab](https://github.com/CyberGod36/okta-iam-lab) — demonstrating multi-IdP skills across Keycloak, Okta, Microsoft Entra ID, and Active Directory.

---

## 📋 Project Overview

This project simulates how an IAM Analyst would deploy and administer an open-source identity provider for a fictional company, **DeonTech Solutions**, with four departments. It demonstrates:

| Skill Area | What's Covered |
|---|---|
| Realm Administration | Custom realm creation, login policies, themes |
| User Lifecycle | Create, activate, disable, password reset, required actions |
| Groups & RBAC | Department groups, realm roles, group-role mapping |
| MFA | TOTP (Google Authenticator) enforcement |
| App Integration | OIDC client + SAML client configuration |
| Security Policies | Password policy, brute-force detection |
| Auditing | Login events & admin events logging |

---

## 🛠️ Lab Environment

- **Keycloak 26.x** running locally on Windows (dev mode)
- **Java 21** (required by Keycloak 26)
- Browser: any modern browser
- MFA app: Google Authenticator or Microsoft Authenticator (phone)

---

## Phase 1 — Install & Start Keycloak

### Steps

1. Download Keycloak from [keycloak.org/downloads](https://www.keycloak.org/downloads) (ZIP distribution).
2. Extract to a simple path, e.g. `C:\keycloak`.
3. Open **PowerShell** in the `bin` folder and start in dev mode:
   ```powershell
   cd C:\keycloak\bin
   .\kc.bat start-dev
   ```
4. Open `http://localhost:8080` in your browser.
5. Create the initial **admin** account (username: `admin`), then log in to the **Administration Console**.

### 📸 Screenshots to capture
- ✅ `01-keycloak-running.png` — PowerShell window showing Keycloak started ("Listening on http://0.0.0.0:8080")
- ✅ `02-admin-console-login.png` — The Keycloak admin login page
- ✅ `03-admin-dashboard.png` — Admin console home after login

---

## Phase 2 — Create a Custom Realm

A **realm** is an isolated tenant (like an Okta org). Never build labs in the `master` realm.

### Steps

1. In the top-left realm dropdown, click **Create realm**.
2. Realm name: `deontech`
3. Click **Create**.
4. Confirm the realm dropdown now shows **deontech**.

### 📸 Screenshots to capture
- ✅ `04-create-realm.png` — The "Create realm" form filled out with `deontech`
- ✅ `05-realm-created.png` — Realm dropdown showing you're inside the `deontech` realm

---

## Phase 3 — Create Groups (Departments)

### Steps

1. Go to **Groups** → **Create group**.
2. Create these four top-level groups:

| Group Name | Purpose |
|---|---|
| `IT-Department` | Admin-level access to apps |
| `HR-Department` | Access to HR systems |
| `Finance-Department` | Access to finance apps |
| `Sales-Department` | Access to CRM tools |

3. (Optional, stands out to employers) Inside `IT-Department`, create a **subgroup** called `IT-Admins` to show nested group structure.

### 📸 Screenshots to capture
- ✅ `06-groups-list.png` — Groups page showing all four department groups
- ✅ `07-nested-group.png` — `IT-Admins` subgroup inside `IT-Department`

---

## Phase 4 — Create Users

### Steps

1. Go to **Users** → **Add user**. Create each user below.
2. For each user: fill in **Username, Email, First name, Last name**, toggle **Email verified** ON, and under **Groups** click **Join Groups** to assign their department.
3. After creating each user, go to the **Credentials** tab → **Set password**. Set a temporary password (e.g. `Welcome2026!`) and leave **Temporary** ON so they must reset on first login.

### 👥 Your Lab Users

| Username | Name | Email | Department / Group |
|---|---|---|---|
| `dwilliams` | Deon Williams | dwilliams@deontech.com | IT-Department → IT-Admins |
| `tmokoena` | Thandi Mokoena | tmokoena@deontech.com | HR-Department |
| `jcarter` | James Carter | jcarter@deontech.com | Finance-Department |
| `lnaidoo` | Lerato Naidoo | lnaidoo@deontech.com | Sales-Department |
| `mbrown` | Marcus Brown | mbrown@deontech.com | IT-Department |
| `kvanwyk` | Kayla van Wyk | kvanwyk@deontech.com | Finance-Department |

### 📸 Screenshots to capture
- ✅ `08-create-user.png` — The "Add user" form for `dwilliams`
- ✅ `09-set-password.png` — Credentials tab with temporary password set
- ✅ `10-users-list.png` — Users page showing all six users
- ✅ `11-user-groups.png` — A user's Groups tab showing department membership

---

## Phase 5 — Create Realm Roles & Map to Groups (RBAC)

This is the heart of the project: **role-based access control via group inheritance** — users never get roles directly; they inherit them from their department.

### Steps

1. Go to **Realm roles** → **Create role**. Create:

| Role Name | Description |
|---|---|
| `app-admin` | Full administrative access to applications |
| `hr-user` | Access to HR applications |
| `finance-user` | Access to financial applications |
| `sales-user` | Access to sales/CRM applications |
| `standard-user` | Baseline access for all employees |

2. Map roles to groups: go to **Groups** → select a group → **Role mapping** tab → **Assign role**:

| Group | Assigned Roles |
|---|---|
| IT-Admins | `app-admin`, `standard-user` |
| IT-Department | `standard-user` |
| HR-Department | `hr-user`, `standard-user` |
| Finance-Department | `finance-user`, `standard-user` |
| Sales-Department | `sales-user`, `standard-user` |

3. **Verify inheritance:** Go to **Users** → `tmokoena` → **Role mapping** tab → toggle "Hide inherited roles" OFF. You should see `hr-user` and `standard-user` inherited from her group.

### 📸 Screenshots to capture
- ✅ `12-realm-roles.png` — Realm roles list showing all five roles
- ✅ `13-group-role-mapping.png` — HR-Department's Role mapping tab
- ✅ `14-inherited-roles.png` — `tmokoena`'s effective roles inherited via group (key screenshot — proves RBAC works)

---

## Phase 6 — Password Policy & Brute-Force Protection

### Steps

1. Go to **Authentication** → **Policies** tab → **Password policy**.
2. Add policies:
   - Minimum length: `12`
   - At least 1 uppercase, 1 lowercase, 1 digit, 1 special character
   - Not recently used: `5`
3. Go to **Realm settings** → **Security defenses** → **Brute force detection**:
   - Enable brute force detection
   - Max login failures: `5`
   - Wait increment: `60 seconds`

### 📸 Screenshots to capture
- ✅ `15-password-policy.png` — Configured password policy
- ✅ `16-brute-force.png` — Brute force detection settings enabled

---

## Phase 7 — Enforce MFA (TOTP)

### Steps

1. Go to **Authentication** → **Required actions** tab.
2. Set **Configure OTP** to **Enabled** and toggle **Set as default action** ON (forces all new users to enroll).
3. Test it: open a private/incognito window → go to the realm account console:
   `http://localhost:8080/realms/deontech/account`
4. Log in as `jcarter` → you'll be forced to reset the temp password, then scan the QR code with Google Authenticator and enter the OTP code.

### 📸 Screenshots to capture
- ✅ `17-required-actions.png` — Configure OTP set as default action
- ✅ `18-otp-enrollment.png` — The QR code enrollment screen during `jcarter`'s first login (blur the QR code before uploading!)
- ✅ `19-account-console.png` — `jcarter` logged into the account console showing "Two-factor authentication" configured

---

## Phase 8 — Integrate an Application (OIDC Client)

### Steps

1. Go to **Clients** → **Create client**.
2. Settings:
   - Client type: **OpenID Connect**
   - Client ID: `deontech-portal`
   - Name: `DeonTech Employee Portal`
3. Capability config: turn **Client authentication** ON (confidential client) → Next.
4. Login settings:
   - Valid redirect URIs: `http://localhost:3000/*`
   - Web origins: `http://localhost:3000`
5. Save, then open the **Credentials** tab to view the client secret (this is what an app would use).
6. (Bonus) Create a second client with type **SAML** named `deontech-saml-app` to show you understand both protocols — mirrors the Org2Org SAML work from your Okta lab.

### 📸 Screenshots to capture
- ✅ `20-oidc-client.png` — `deontech-portal` client settings page
- ✅ `21-client-credentials.png` — Credentials tab (blur/redact the actual secret!)
- ✅ `22-saml-client.png` — SAML client configuration (bonus)

---

## Phase 9 — Auditing & Event Logging

### Steps

1. Go to **Realm settings** → **Sessions/Login** area, then **Events** section (left menu in newer versions: **Realm settings → Events** or sidebar **Events**).
2. Under **User events settings**: toggle **Save events** ON. Set expiration (e.g., 7 days).
3. Under **Admin events settings**: toggle **Save events** ON.
4. Generate activity: log in/out as a user, fail a login on purpose, change a user's group.
5. View **Events** in the sidebar — you'll see `LOGIN`, `LOGIN_ERROR`, `UPDATE_PASSWORD`, etc.

### 📸 Screenshots to capture
- ✅ `23-events-config.png` — User & admin event saving enabled
- ✅ `24-login-events.png` — Event log showing LOGIN and LOGIN_ERROR entries (this is your "I understand audit trails" proof)

---

## Phase 10 — Lifecycle Management Demo

Show you understand joiner/mover/leaver (JML) — a favorite IAM interview topic.

### Steps

1. **Joiner:** You already did this — new user, temp password, group assignment, forced MFA enrollment.
2. **Mover:** Move `mbrown` from `IT-Department` to `Finance-Department`. Screenshot his role mapping before & after — his effective roles change automatically.
3. **Leaver:** Disable `kvanwyk` (Users → kvanwyk → toggle **Enabled** OFF). Attempt to log in as her — access denied. Check the event log for the failed attempt.

### 📸 Screenshots to capture
- ✅ `25-mover-before-after.png` — `mbrown`'s roles before and after group change
- ✅ `26-disabled-user.png` — `kvanwyk` disabled + the login failure screen
- ✅ `27-leaver-event-log.png` — Event log entry for the denied login

---

## 📁 Suggested Repo Structure

```
keycloak-iam-homelab/
├── README.md
├── screenshots/
│   ├── 01-keycloak-running.png
│   ├── ...
│   └── 27-leaver-event-log.png
└── docs/
    ├── rbac-design.md        (your role/group matrix + reasoning)
    └── lessons-learned.md    (what broke, how you fixed it)
```

---

## 🎯 Key Takeaways (for recruiters)

- Designed and administered a multi-department realm in Keycloak from scratch
- Implemented **RBAC via group-inherited realm roles** — no direct role assignment
- Enforced **MFA (TOTP)**, strong password policy, and brute-force protection
- Integrated applications using both **OIDC and SAML 2.0**
- Demonstrated full **user lifecycle (JML)** with audit evidence from event logs
- Multi-IdP experience: Keycloak (this repo) + [Okta](https://github.com/CyberGod36/okta-iam-lab) + Microsoft Entra ID / Active Directory

---

## ⚠️ Security Notes

- This lab runs in **dev mode** (`start-dev`) — never use dev mode in production
- All credentials in this repo are fictional lab values
- Client secrets and QR codes are redacted in screenshots

---

*Built as part of my IAM portfolio targeting IAM Analyst and DoD 8140-aligned cybersecurity roles.*

