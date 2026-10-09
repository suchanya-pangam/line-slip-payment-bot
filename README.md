# Department Payment Collection Bot (LINE Slip Automation)

An n8n workflow that automates department fee collection. Students send a bank transfer slip to a LINE Official Account; the bot reads the slip with an AI vision model, matches the payer to the student roster, verifies the amount, records the payment in Google Sheets, and replies to the student in LINE.

I built this as the department **treasurer** to replace manual slip checking, where each slip had to be opened, read, matched to a student, and recorded by hand.

![Workflow overview](assets/workflow-canvas.png)

## Problem

Collecting department fees through bank transfer meant:

- Opening every slip sent in chat and reading the name, amount, and date by hand
- Finding the matching student in the roster and updating a spreadsheet
- Catching mistakes such as the same slip being sent twice or a wrong amount
- Replying to each student to confirm the payment

## How It Works

| Step | What happens | n8n nodes |
|---|---|---|
| 1. Receive | LINE sends a webhook; the request signature is verified (HMAC) so only genuine LINE requests are processed | Webhook, Crypto, IF |
| 2. Filter | Keeps user messages and checks that an image was sent | Split Out, Filter, IF |
| 3. Fingerprint | Downloads the slip image and hashes it (SHA-256) | HTTP Request, Crypto |
| 4. Duplicate check | Rejects slips whose hash is already in the transaction history | Google Sheets, Code, IF |
| 5. Read slip | An AI vision model (GPT) extracts payer name, amount, reference number, and date from Thai bank slips, returning structured JSON | OpenAI, Code, Switch |
| 6. Match student | Compares the payer name with the roster; only one confident match is accepted | Google Sheets, Code, IF |
| 7. Verify | Finds the payment round from the transfer date and checks the student is listed, unpaid, and paid the exact amount due | Code, Google Sheets, IF |
| 8. Record | Marks the student as paid and appends the slip to the transaction history | Google Sheets |
| 9. Review | Uncertain cases (no match, wrong amount, unreadable slip) go to a manual review sheet with the reason | Google Sheets |
| 10. Reply | Sends a Thai confirmation or problem message back to the student in LINE | Code, HTTP Request |

## Design Decisions

- **Never guess.** The AI prompt forbids guessing numbers, and the workflow only auto-confirms when the name match is unique and the amount is exact. Everything else goes to a human.
- **Duplicate protection.** Image hashing stops the same slip from being recorded twice.
- **Verified webhooks.** LINE signature validation blocks forged requests.
- **Audit trail.** Every recorded payment and every review case is logged with a timestamp and LINE message ID.

## Tech Stack

n8n · LINE Messaging API · OpenAI API (GPT vision) · Google Sheets · JavaScript (n8n Code nodes)

## Setup

1. Import `workflow/payment-collection-bot.json` into n8n (**Workflows → Import from File**).
2. Create these credentials in n8n and attach them to the matching nodes:
   - **LINE Messaging API**: a header auth with your channel access token (`Authorization: Bearer <token>`)
   - **Crypto**: your LINE channel secret, used for HMAC signature checking
   - **OpenAI**
   - **Google Sheets** (OAuth2)
3. Create a Google Sheet with the structure in [`docs/sheet-structure.md`](docs/sheet-structure.md) and replace every `YOUR_GOOGLE_SHEET_ID` in the workflow with its ID.
4. Copy the production webhook URL from the **LINE Webhook** node into the LINE Developers Console and enable webhooks.
5. Activate the workflow and send a test slip to the LINE Official Account.

## Privacy Note

The real ledger contains student names, IDs, and payment records, so it is not included. The workflow file in this repository has all sheet IDs and credentials removed.
