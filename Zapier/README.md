# Zapier Invoice Automation – Scheduling & Workflow Automation

## Project Overview

This project demonstrates an automated **invoice management system** built with **Zapier**, **Google Sheets**, and **Gmail**.

The system reduces manual effort in sending payment reminders and in updating invoice records after payment confirmations are received. It consists of **two separate Zaps**:

1. **Zap 1 – Payment Request Automation:** Runs on a schedule, finds pending invoices in Google Sheets, and sends a personalized payment reminder email for each one.
2. **Zap 2 – Automated Payment Confirmation & Reconciliation:** Monitors incoming Gmail replies, detects payment confirmations, and updates the matching invoice record in Google Sheets.

---

## Objectives

- Automate recurring payment reminders using a scheduled trigger.
- Use Google Sheets as a central invoice tracking database.
- Process each pending invoice individually using Looping by Zapier.
- Send personalized emails using dynamic field mapping in Gmail.
- Detect payment confirmation replies using Filter by Zapier.
- Extract the Invoice ID from an email subject using Formatter by Zapier.
- Update the invoice status automatically after payment confirmation.
- Support reconciliation by recording the Transaction ID / UTR.

---

## Technologies Used

| Technology / Component | Purpose |
|---|---|
| **Zapier** | Workflow automation platform used to build both Zaps |
| **Schedule by Zapier** | Triggers the reminder workflow at a fixed time every day |
| **Google Sheets** | Stores invoice data and payment status |
| **Looping by Zapier** | Handles each pending invoice as a separate item |
| **Gmail** | Sends reminder emails and receives payment replies |
| **Filter by Zapier** | Allows only valid payment confirmation emails to continue |
| **Formatter by Zapier** | Extracts the Invoice ID from the email subject |

---

## Sample Dataset

Spreadsheet: `SCIT_invoice` | Worksheet: `Invoice`

| Sr | Invoice ID | Type | Name | Amount | Account No | Bank Name | Status |
|---|---|---|---|---|---|---|---|
| 1 | 1001 | Maintenance | Madhav | 50000 | 21354964864 | SBI | Pending |
| 2 | 1002 | Project | Ram | 28000 | 12565647877 | ICICI | Pending |
| 3 | 1003 | Maintenance | Pavan | 94000 | 23134546008 | HDFC | Pending |
| 4 | 1004 | Project | Raj | 80000 | 66563745658 | Kotak | Completed |
| 5 | 1005 | Maintenance | Veeru | 85000 | 99697163008 | DBS | Completed |

Only invoices **1001, 1002, and 1003** are pending and should receive reminders. Invoices **1004** and **1005** are already completed.

---

## Workflow Architecture

### Zap 1 – Payment Request Automation

```text
Schedule by Zapier (Daily, 9:00 AM)
            ↓
Google Sheets – Lookup Spreadsheet Rows (Status = Pending)
            ↓
Looping by Zapier – Create Loop From Line Items
            ↓
Gmail – Send Payment Reminder Email
```

### Zap 2 – Automated Payment Confirmation & Reconciliation

```text
Gmail – New Email
            ↓
Filter by Zapier – Payment confirmation phrase (e.g., "Payment Done")
            ↓
Formatter by Zapier – Extract Invoice ID from subject
            ↓
Google Sheets – Lookup Spreadsheet Row (by Invoice ID)
            ↓
Google Sheets – Update Spreadsheet Row (Status = Completed)
```

---

## Zap 1: Payment Request Automation

### Step 1 – Schedule by Zapier
Triggers the workflow automatically at a predefined time (daily), so no manual start is needed.

### Step 2 – Google Sheets: Lookup Spreadsheet Rows
Looks up rows in the `Invoice` worksheet of `SCIT_invoice` where the **Status** column equals **Pending**.

### Step 3 – Looping by Zapier: Create Loop From Line Items
Maps Invoice ID, Type, Name, Amount, Account Number, Bank Name, and Status into a loop so each pending invoice is processed separately and keeps its own amount and bank details.

### Step 4 – Gmail: Send Payment Reminder Email
Sends a personalized reminder using dynamic values from the current invoice. The recipient is chosen by invoice type:

- **Project** invoices → designated Project recipient
- **Maintenance** invoices → designated Maintenance recipient

**Sample email format**

```text
Subject: Payment Reminder – Invoice [Invoice ID]

Dear [Recipient Name],

This is a friendly reminder that payment for the following invoice is currently pending.

• Invoice ID: [Invoice ID]
• Name: [Name]
• Type: [Type]
• Amount Due: ₹[Amount]
• Payment Status: [Status]

Bank Details:
• Bank Name: [Bank Name]
• Account Number: [Account Number]

Kindly review and process the payment. If the payment has already been completed,
please reply to this email with the Invoice ID and Transaction ID/UTR for reconciliation.

Regards,
Accounts Team
```

### Step 5 – Testing and Validation
Run a test to confirm that only pending invoices are selected, each is processed individually, and each email contains the correct values for its invoice.

---

## Zap 2: Automated Payment Confirmation & Reconciliation

### Step 1 – Gmail: New Email
Monitors the connected mailbox for incoming replies that may contain payment confirmations.

### Step 2 – Filter by Zapier: Identify Payment Confirmations
Continues only if the email contains a confirmation phrase such as **"Payment Done"**. Messages saying payment will be made later or is still processing are not treated as completed.

### Step 3 – Formatter by Zapier: Extract Invoice ID
Extracts the Invoice ID from the subject. For example, a reply to "Payment Reminder – Invoice 1002" gives **1002**.

### Step 4 – Google Sheets: Lookup Spreadsheet Row
Searches the `Invoice` worksheet using the **Invoice ID** column. If no record matches, no invoice is updated.

### Step 5 – Google Sheets: Update Spreadsheet Row
Changes the matching row's **Status** from Pending (or Reminder Sent) to **Completed**. In the extended version, the **Transaction ID / UTR** from the reply is also saved.

If the confirmation is ambiguous, the Invoice ID cannot be matched, or the Transaction ID is missing, the invoice stays unchanged until verified.

### Step 6 – Testing and Validation
Test with a real confirmation email (e.g., a reply of "Payment Done!" to the reminder for Invoice 1002) and confirm that the filter passes, the Invoice ID is extracted correctly, the right row is found, and the status becomes **Completed**.

---

## Edge Cases Handled

| Scenario | Expected Behavior |
|---|---|
| Invoice already marked Completed | No reminder is sent |
| Email does not contain a confirmation phrase | Filtered out, no update |
| Invoice ID does not match any record | No record is updated |
| Transaction ID / UTR missing or confirmation unclear | Invoice remains unchanged until verified |

---

## How to Set Up

1. Create a Google Sheet named `SCIT_invoice` with a worksheet `Invoice` using the columns shown above.
2. In Zapier, create **Zap 1** (Schedule → Google Sheets Lookup → Looping → Gmail Send Email) and map the fields into the email template.
3. Create **Zap 2** (Gmail New Email → Filter → Formatter → Google Sheets Lookup → Google Sheets Update Row).
4. Test each step, then **Publish** both Zaps.

---

## Conclusion

This project shows how Zapier can connect scheduling, spreadsheet management, and email communication to automate invoice follow-up and payment reconciliation. The integration of Google Sheets and Gmail reduces repetitive manual work, keeps a centralized record of invoice statuses, and improves consistency, traceability, and efficiency in invoice management.
