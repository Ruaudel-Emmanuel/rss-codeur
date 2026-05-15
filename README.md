Codeur.com RSS Alert — n8n Setup Guide
This workflow monitors the Codeur.com RSS feed every 30 minutes, filters projects in the €1,000–€10,000 budget range, deduplicates them via Airtable, and sends you an email alert with a ready-to-paste first message.
---
Workflow Overview
```
Schedule Trigger (30 min)
  → RSS Feed Read (codeur.com/projects.rss)
  → Code Node: Budget Filter (1k–10k€ or unknown budget)
  → Airtable Search: Check if guid already processed
  → IF Node: Not a duplicate?
      → YES → Airtable Create: Save project
               → Gmail: Send email alert
      → NO  → Stop (do nothing)
```
---
Prerequisites
Tool	What you need
n8n	Self-hosted or n8n Cloud account
Airtable	Free account + Personal Access Token
Gmail	OAuth2 credential configured in n8n
---
Step 1 — Create the Airtable Base
Create a new base called Codeur Projects with a single table named `Codeur Projects`.
Add the following fields:
Field name	Field type	Notes
`guid`	Single line text	Unique identifier from RSS — used for deduplication
`titre`	Single line text	Project title
`lien`	URL	Direct link to the project on Codeur
`date`	Date	Publication date from RSS feed
> **Important:** Note your **Base ID** (visible in the Airtable URL: `https://airtable.com/YOUR_BASE_ID/...`). You will need it in n8n.
---
Step 2 — Configure Credentials in n8n
Gmail OAuth2
In n8n → Credentials → New → search `Gmail OAuth2`
Follow the Google OAuth flow
Name it `Gmail OAuth2`
Airtable Token
Go to airtable.com/create/tokens
Create a Personal Access Token with scopes: `data.records:read`, `data.records:write`
Grant access to your Codeur Projects base
In n8n → Credentials → New → search `Airtable Token API`
Paste the token, name it `Airtable Token`
---
Step 3 — Import the Workflow
In n8n, click Workflows → Import from file
Select `codeur_rss_workflow.json`
Open the workflow
---
Step 4 — Update the Placeholders
Replace every `YOUR_*` value in the nodes:
Node	Field to update	Replace with
`Airtable — Check Duplicate`	`application`	Your Airtable Base ID
`Airtable — Check Duplicate`	Credential	Select `Airtable Token`
`Airtable — Save Project`	`application`	Your Airtable Base ID
`Airtable — Save Project`	Credential	Select `Airtable Token`
`Send Email Alert`	`fromEmail`	Your Gmail address
`Send Email Alert`	`toEmail`	Your Gmail address
`Send Email Alert`	Credential	Select `Gmail OAuth2`
---
Step 5 — Customize Your Template Message
In the Send Email Alert node, find the `✉️ Your template message` section in the HTML body and replace it with your own first-contact message.
The current template:
> Bonjour,
>
> J'ai bien lu la description de votre projet et je pense pouvoir vous apporter une solution efficace.
>
> Je suis développeur Python & automatisation (n8n, IA) basé à Rennes, avec une approche pragmatique : des outils qui fonctionnent, livrés dans les délais, sans jargon.
>
> Pourriez-vous me préciser vos contraintes de délai et le format de livraison attendu ? Je pourrai ainsi vous faire une proposition adaptée.
>
> Cordialement,
> Emmanuel Ruaudel — rennesdev.fr
---
Step 6 — Test the Workflow
Click Execute Workflow manually (do not activate yet)
Check the RSS node output — you should see recent Codeur projects
Check the Budget Filter node — verify items are passing through correctly
Check Airtable — a record should appear in your table
Check your inbox — the email alert should arrive
If the RSS URL returns no data, try these alternative URLs:
`https://www.codeur.com/projects.rss`
`https://www.codeur.com/projects.rss?category=developpement`
`https://www.codeur.com/projects.rss?category=automatisation`
---
Step 7 — Activate
Once the test passes, toggle the workflow to Active.
---
How the Budget Filter Works
The Budget Filter Code node scans the project title and description for euro amounts. Logic:
Amounts like `2000€`, `2 000 €`, `2k€`, `2.000€` are detected and parsed
If at least one amount is between €1,000 and €10,000 → project passes through
If no budget is mentioned → project passes through (unknown budget = potential opportunity)
If all amounts are clearly outside the range (e.g., `50€` or `50 000€`) → project is dropped
---
Email Format Received
```
Subject: 🆕 [Codeur] Automatisation Excel pour PME BTP

New project detected on Codeur.com

📌 Project: Automatisation Excel pour PME BTP
🔗 Link: [Open on Codeur]
📅 Published: Thu, 15 May 2026 10:34:00 +0200

Description:
[Full project description]

✉️ Your template message — copy/paste on Codeur:
Bonjour,
J'ai bien lu la description de votre projet...
```
---
Maintenance Notes
If Codeur changes their RSS structure, check field names in the RSS node output (`guid`, `link`, `title`, `pubDate`) and update the Airtable Create node mappings accordingly.
Airtable will grow over time. Consider adding an automated cleanup rule to delete records older than 90 days.
Multiple RSS categories: Duplicate the workflow and point each copy to a different RSS URL (e.g., one for `developpement`, one for `automatisation`).
---
Troubleshooting
Problem	Likely cause	Fix
No items from RSS node	Codeur URL changed	Test URL directly in browser
All items filtered out	Budget regex too strict	In Code node, temporarily set `budgetOk = true` to bypass filter
Duplicate emails	`guid` field empty in RSS	Use `link` as deduplication key instead of `guid`
Airtable error 422	Wrong Base ID	Copy Base ID from Airtable URL
Gmail not sending	OAuth token expired	Re-authenticate the Gmail credential in n8n
