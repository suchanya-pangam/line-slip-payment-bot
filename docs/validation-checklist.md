# Validation Checklist

Use a separate test ledger and controlled, non-personal inputs. Record the result of each check.

## Import and configuration

- Import `workflow/payment-collection-bot.json`; confirm all 28 nodes are available in your n8n version.
- Replace every `REPLACE_ME` credential and `YOUR_GOOGLE_SHEET_ID` value, and bind the model credential.
- Check LINE HTTP authentication, the raw webhook body used for HMAC, SHA-256 hashing, and binary-image input to the vision node.
- Confirm the roster, payment-round tabs, history tab, and review tab use the documented columns.

## Branch checks

| Input / condition | Expected route | What to inspect |
|---|---|---|
| Invalid LINE signature | Reject Invalid Signature | No payment write or reply |
| Non-image message | Compose Reply Message → LINE Reply | Request for a slip image |
| Image hash already in history | Duplicate reply | No payment update or new history record |
| Unreadable / incomplete AI response | Direct retry reply | No roster matching; no review-queue append |
| Multiple or no roster matches | Review queue and review reply | Reason and candidate values |
| Unique exact or surname-prefix match | Choose Payment Round | Selected student and matching method |
| Missing round tab / student row | Review queue and review reply | No payment status update |
| Student already marked paid | Review queue and review reply | Existing payment remains intact |
| Amount differs from amount due | Review queue and mismatch reply | No payment status update |
| Valid unpaid row and matching amount | Update Payment Status → Log Transaction → Restore Payment Reply Context → reply | Payment fields, history entry, and success reply |

The amount comparison in `Confirm Round Record` allows an absolute difference of up to **0.005 THB**. `Parse Slip Data` checks required values but does not validate a receiver account or impose a confidence threshold.

## Integration and failure checks

- Confirm one event and multiple image events retain the correct reply token, image, payer, and message identifiers through Sheets and AI nodes.
- Check successful and failed writes separately: the update and log are not an atomic transaction.
- Test service errors, redelivery, simultaneous submissions, and LINE reply failures. Do not assume the hash lookup provides a concurrency lock.
- Verify what happens when a round tab is missing; `Read Round Sheet` uses `continueRegularOutput`.
- Confirm the stored review reason is meaningful for each branch.

## Evidence for a future portfolio update

Record the n8n version, test date, configuration, expected result, observed result, and any failed cases. Use anonymised screenshots; remove credentials, names, student IDs, slips, and production Sheet URLs. Add performance claims only when supported by recorded measurements.

## Known Limitations

- Slip reading does not confirm settlement, authenticity, or the recipient account with a bank.
- Extracted receiver and confidence values are not acceptance gates. The AI prompt asks the model not to guess; the parser checks required fields rather than verifying their truth.
- Duplicate detection compares image hashes, not transaction reference numbers. Edited or re-exported images may have different hashes. Concurrent submissions are not locked.
- Matching can accept a unique first-name and surname-prefix match; it does not require a student ID from the sender.
- Payment rounds use the exact transfer date as `dd/MM/yy`. A slip from a different date needs a corresponding round tab or goes to review.
- Amount comparison allows an absolute difference of up to 0.005 THB. A blank amount-due cell goes to review.
- Payment-row updates and transaction logging are separate writes. A log failure can leave the payment row updated without a success reply.
- Unreadable slips receive a retry reply and are not appended to the review queue. Not every external-service error has a review route.
- JSON structure, node references, connections, and Code-node syntax have been checked locally. The Cloud connections and Code-node sources match the repository. Live integration testing requires configured LINE, OpenAI, and Google Sheets credentials.
