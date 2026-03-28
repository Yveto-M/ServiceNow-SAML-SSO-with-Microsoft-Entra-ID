# UVBANK – ServiceNow SAML SSO with Microsoft Entra ID

This project documents the integration of a **ServiceNow Zurich Personal Developer Instance (PDI)** with **Microsoft Entra ID** using **SAML**. The goal is to make ServiceNow a federated SaaS application in the UVBANK lab, authenticated centrally by Entra ID.

---

## Scope and objectives

- Configure ServiceNow as a SAML service provider using Multi‑Provider SSO.
- Configure Entra ID as the identity provider for ServiceNow.
- Validate end‑to‑end SSO (Entra authentication → ServiceNow landing page).
- Resolve a ServiceNow issue where the SAML IdP would not activate despite successful SSO.

All configuration and troubleshooting steps are illustrated with the screenshots listed below.

---

## Implementation overview

### 1. Enable SAML capability in ServiceNow

- Installed the **Integration – Multiple Provider Single Sign‑On** plugin in the Zurich PDI.  
  - `Screenshot-2026-03-27-183530-2.jpg` – Plugin installer.
  <img width="923" height="456" alt="Screenshot 2026-03-27 183505" src="https://github.com/user-attachments/assets/1a4ea26f-ddff-4b0e-af9e-9a5f9dfe9f22" />

  - `Screenshot-2026-03-27-183505.jpg` – Plugin installation progress.
- <img width="600" height="331" alt="Screenshot 2026-03-27 183530" src="https://github.com/user-attachments/assets/5153dfcb-9fa4-44c7-8e76-1cbde1f278d9" />


- Configured **Multiple Provider SSO Properties** to:
  - Allow multiple IdPs.
  - Auto‑import users from the IdP into `sys_user`.
  - Enable debug logging.  
  - `Screenshot-2026-03-27-185630-4.jpg` – SSO properties.
  - <img width="655" height="372" alt="Screenshot 2026-03-27 185630" src="https://github.com/user-attachments/assets/e5ebfcb7-c140-446d-8e73-b7ea09240f0d" />


- Created a new **Identity Provider** and imported Microsoft Entra metadata via URL.  
  - `Screenshot-2026-03-27-185323-3.jpg` – Import Identity Provider Metadata dialog.
<img width="860" height="421" alt="Screenshot 2026-03-27 185323" src="https://github.com/user-attachments/assets/d1bec4c0-3e86-424b-ad90-150a09d22a42" />

---

### 2. Configure Microsoft Entra ID for ServiceNow

- Used the ServiceNow enterprise application in Entra ID.
- Defined **Basic SAML Configuration**:
  - Identifier (Entity ID) – ServiceNow instance URL.
  - Reply URL (Assertion Consumer Service URL).
  - Sign‑on URL – ServiceNow login URL.  
  - `Screenshot-2026-03-28-160449-10.jpg` – Basic SAML configuration.
<img width="615" height="293" alt="Screenshot 2026-03-28 160449" src="https://github.com/user-attachments/assets/d446f4a7-9054-4680-80a1-5e89d7645010" />

- Validated the SSO flow:
  - User authenticates in Entra ID.
  - User is redirected to the ServiceNow home page.  
  - `Screenshot-2026-03-28-145239-5.jpg` – ServiceNow landing page after SSO.
<img width="885" height="456" alt="Screenshot 2026-03-28 145239" src="https://github.com/user-attachments/assets/c573cbfb-d65f-49a1-9010-2f4c3bce2804" />

---

### 3. ServiceNow IdP activation issue

- The SAML configuration was functional (users could sign in via Entra and reach ServiceNow), but the Identity Provider record could not be activated.
- ServiceNow displayed the banner:

  > “Before activating an IdP record, you must Test the connection to the IdP.”

  - `Screenshot-2026-03-28-150029-12.jpg` – Activation error banner.
<img width="915" height="189" alt="Screenshot 2026-03-28 150029" src="https://github.com/user-attachments/assets/e8d13806-2d0e-4c0d-8232-385ff8a07504" />

- Reviewed the Identity Provider record (URLs and Active flag) while troubleshooting.  
  - `Screenshot-2026-03-28-163844-7.jpg` – Identity Provider form.
<img width="927" height="427" alt="Screenshot 2026-03-28 163844" src="https://github.com/user-attachments/assets/eec7752d-4b70-49f1-9afa-5d87df81c19c" />

---

### 4. Resolution using a controlled system property

To address the stuck activation state, a documented system property was used:

- Created/updated the property:

  - `glide.authenticate.multisso.test.connection.mandatory = false`

- This temporarily removed the requirement for a recorded successful **Test Connection** before activation.

  - `Screenshot-2026-03-28-163522-9.jpg` – System Property configuration.
<img width="929" height="605" alt="Screenshot 2026-03-28 163522" src="https://github.com/user-attachments/assets/85de54e6-8592-480e-9076-74ee0a28c9f5" />

- With the property set to `false`:
  - Re‑ran **Test Connection**.
  - Successfully set the Identity Provider to **Active**.

  - `Screenshot-2026-03-28-164634-8.jpg` – Identity Providers list showing the Entra SAML IdP as Active.
<img width="927" height="357" alt="Screenshot 2026-03-28 164634" src="https://github.com/user-attachments/assets/d8f6d3a2-b000-47c4-8a9f-0b536fbbf763" />

- After activation, the property was returned to `true` (not shown in screenshots) so future IdPs must pass Test Connection before activation.

---

## Outcome

- ServiceNow now trusts Microsoft Entra ID via SAML 2.0.
- Users from the UVBANK Entra tenant authenticate in Entra and are redirected into the ServiceNow Zurich PDI.
- The environment demonstrates:
  - SaaS onboarding to Entra ID.
  - Practical understanding of ServiceNow Multi‑Provider SSO.
  - Use of a documented system property to resolve an activation issue while maintaining security guardrails.

This integration is used in the UVBANK lab as a realistic example of enterprise ITSM onboarding into a centralized identity platform.
