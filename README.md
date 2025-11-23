📦 PHONE INVENTORY WORKFLOW — TECHNICAL DOCUMENTATION
📌 Purpose

This workflow manages the process of buying and selling phones, captures customer/device info via forms, stores the data in Google Sheets, and sends real-time notifications via Telegram.

The system automates:

Authentication

Data entry

Transaction classification (Buy / Sell)

Database storage

Messaging

1️⃣ Login Form — formTrigger (Node: Login)
Role

The entry point of the workflow.
Collects user credentials to allow workflow access.

Form Fields
Field	Type	Required
User Number	number	✔
Password	password	✔

Both fields must be filled before validation.

Submitted values are passed as:

$json['User Number']

$json['Password']

This ensures unauthorized users cannot continue to the product entry stage.

2️⃣ Login Validation — If (Node: If)
Purpose

Check if the user credentials are correct.

Conditions (AND logic)
Field	Operator	Value
$json['User Number']	equals	12345
$json.Password	equals	"password"
Result

TRUE → Redirect to product form

FALSE → Redirect to No Operation

If the user is invalid, the workflow stops immediately.

⚠️ Security Concern:
Storing passwords directly in the workflow is not secure.
Use credentials, DB lookup, or JWT instead.

3️⃣ Product & Customer Form — form (Node: Form)
Role

Collect complete transaction and customer information.

Form Fields
Field	Type	Required
imei	text	✔
Model	text	✔
Fiyat (Price)	number	✔
Durum (Operation Type)	dropdown	✔
İsim (Name)	text	✔
Soyisim (Surname)	text	✔
Kimlik No (ID)	text	✔
İletişim (Phone)	number	✔
Not	text	optional
Operation field options

SATIS (Sale)

ALIS (Purchase)

The user data is then passed to the Switch node.

4️⃣ Transaction Classification — Switch (Node: Switch)
Key Parameter

$json.Durum

This node directs the workflow depending on the selected transaction type.

Value	Direction
ALIS	→ buy_list
SATIS	→ sell_list

This allows using a single form to manage two different business processes.

5️⃣ Phone Purchases — Google Sheets (Node: buy_list)
Role

Append or update phone purchase records to Google Sheets.

Target Spreadsheet

telefon_kayit

Tab: buy

Write Mode

operation = appendOrUpdate

Columns Mapping
Sheet Column	Value
imei	{{ $json.imei }}
Model	{{ $json.Model }}
Fiyat	{{ $json.Fiyat }}
Durum	{{ $json.Durum }}
İsim	{{ $json["İsim"] }}
Soyisim	{{ $json.Soyisim }}
T.C. No	{{ $json["Kimlik No"] }}
İletisim	{{ $json["İletişim"] }}
NOT	{{ $json.Not }}
Tarih	JS formatted datetime
IMEI Matching

matchingColumns = ["imei"]

If IMEI exists → update row

If IMEI does not exist → append

This ensures inventory consistency.

6️⃣ Phone Sales — Google Sheets (Node: sell_list)
Role

Append or update phone sale records to another tab.

Target Spreadsheet

Same Sheet

Tab: sell

Mapping is identical to the purchase node.

—

Purpose

Separates buying and selling operations making analytics and reporting cleaner.

7️⃣ Merge Node (Node: Merge)
Mode

append

Purpose

Combine data from buy_list and sell_list

Create a unified dataset to be passed to Telegram

This ensures that Telegram receives a single data payload, regardless of transaction type.

8️⃣ Telegram Notification — Send a text message
Role

Send detailed notifications for each transaction.

chatId

1101442260

Message Template
İŞLEM = {{ $json.Durum }}
IMEI = {{ $json.imei }}
MODEL = {{ $json.Model }}
FİYAT = {{ $json.Fiyat }}
İSİM = {{ $json["İsim"] }}
SOYİSİM = {{ $json.Soyisim }}
İLETİŞİM = {{ $json["İletisim"] }}
NOT = {{ $json.NOT }}
TARİH = {{ $json.Tarih }}


📌 Purpose:

Immediate transaction log

Real-time remote inventory monitoring

Manager supervision

9️⃣ No Operation Node

Node: No Operation, do nothing

Purpose

If login is invalid:

Stop workflow execution

No logs

No data storage

Clean and secure termination policy.

🛡️ SECURITY RECOMMENDATIONS
🚫 Avoid plain-text credentials

Current workflow hardcodes:

User Number

Password

This is risky and can leak via:

UI forms

Workflow exports

Telegram messages

Better alternatives
✔ Credentials store
✔ Database validation
✔ OAuth / Token-based login
✔ JWT

🚫 Sensitive fields in Telegram

Currently sending:

Phone Number

ID (T.C. No)

Telegram chats are not a secure data vault.

Recommended

Mask personal data

Send only internal inventory data

Use admin-only groups

Example:

IMEI: 1234****
Customer: M.**** K.****

🔀 Data Model Overview
Sheet Tab	Description
buy	Purchase transactions
sell	Sale transactions
🔁 Workflow Diagram
Login Form
   → If (Validate Credentials)
       → Form (Customer + Phone)
            → Switch
                ├─ ALIS → buy_list → Merge
                └─ SATIS → sell_list → Merge
                     → Telegram Message
       → No Operation
