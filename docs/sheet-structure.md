# Google Sheet Structure

The workflow reads and writes one spreadsheet with the tabs below. Column names must match exactly (they are in Thai because the bot was built for a Thai department).

## `รายชื่อ` (Student roster)

| Column | Description |
|---|---|
| `รหัสนักศึกษา` | Student ID |
| `ชื่อ` | First name (Thai) |
| `นามสกุล` | Surname (Thai) |
| `Name (Eng)` | Optional English full name, used as a second match |

## Payment round tabs (one per round)

Each round is its own tab named after the collection date in `dd/MM/yy` format (for example `03/03/25`). The workflow picks the tab from the slip's transfer date.

| Column | Description |
|---|---|
| `รหัสนักศึกษา` | Student ID (used to find the row) |
| `ยอดที่ต้องจ่าย (บาท)` | Amount due in THB |
| `ยอดเงิน (บาท)` | Amount paid (written by the bot) |
| `สถานะ` | Status, set to `ชำระแล้ว` (paid) by the bot |
| `เวลาที่จ่าย` | Transfer time (written by the bot) |
| `หมายเหตุ` | Note, the bot writes `Ref: <reference number>` |

## `_ประวัติรายการ` (Transaction history)

`timestamp`, `line_webhook_event_id`, `line_message_id`, `image_sha256`, `student_id`, `payer_name`, `amount`, `payment_round`, `reference_no`

## `_รอตรวจสอบ` (Manual review queue)

`timestamp`, `reason`, `line_webhook_event_id`, `line_message_id`, `line_user_id`, `image_sha256`, `payer_name`, `amount`, `reference_no`, `transaction_datetime`, `candidate_students`
