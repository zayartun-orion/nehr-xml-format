# putComposition (Referral Notes) — Fixes To Make

**For the developer. Simple English.** Checked against the Excel mapping template `putComposition_v0.2_(RefNotes)_20251224.xlsx` — that file is the rule.

Endpoint checked: `http://ec2-47-131-196-190.ap-southeast-1.compute.amazonaws.com/nehr/xml/referral` (PUT). 23 messages, each a referral letter with 5 sections and an attached PDF.

Good news first: the structured referral data is mostly right — the section codes are correct, and the composition-level `encounter` already carries the correct putEvent event id (all 23). The problems below are field order, a few element names, and one wrong block.

---

## Fix now

**1. `composition` field order is reversed.**
You send all 5 `<section>` blocks **first**, then the document metadata (identifier, date, createdDate, type, title, status, creator, author, custodian, encounter). The Excel puts the metadata **first** and `section` **last**.
All the metadata fields are present and correct — just move the 5 `<section>` blocks to the **end** of `<composition>`. (NEHR most likely checks this order through the XSD.)

**2. Inside `referralRequest`, use the exact element names from the Excel — even the ones that look misspelled.**
The Excel deliberately keeps two "wrong-looking" names so they match NEHR's existing system. It says so in the Remarks. **Do not correct them:**
- `requester > practitioner` must be **`practitoner`** — the Excel (row 558) says: *"Keeping 'practitoner' name to avoid breaking existing systems."*
- `requester > organization` must be **`organisation`** (British spelling, Excel row 562).
- `recipient > organization` must be **`organisation`** (Excel row 572).

Your code sends `practitioner` / `organization` everywhere. Inside `referralRequest` they must be `practitoner` / `organisation`.
**Leave the `practitioner` under composition `creator` and `author` as-is** — those are correctly spelled in the Excel (rows 93, 99). Only the ones inside `referralRequest` change.

**3. Remove the `<type>` block inside `requester > organisation`.**
The Excel's requester organisation has only `identifier` and `name` (rows 562-565) — there is no `type`. You send:
```xml
<type>
  <code>Cardiology</code>
  <codingSchemeName>Document_Type_(NEHR)</codingSchemeName>
  <textDescription>Cardiology report</textDescription>
</type>
```
This is wrong (wrong code-set, wrong value, and the field does not exist here). Delete it.
Note: the `type` inside `recipient > organisation` is correct (it uses `Organisation_Type_(NEHR)`) — keep that one.

**4. Spelling: `langauge` → `language`.**
In the section file attachment you send `<langauge>` (Excel row 630 spells it `language`). This is the same typo as in putCardiology, and the field is Mandatory. Fix it.

**5. `referralRequest > encounter > identifier` has the wrong value.**
You send a UUID (`870e645e-...`). The Excel (row 594) says it must be the **putEvent event id**. You already put the correct event id in the composition-level `encounter` — use the same value here, or drop this optional `encounter` block if you do not need it.

**6. Date-time format on `dateSent`.**
Sent as `2026-09-01T19:02:01.9313982+08:00` — with fractional seconds and a timezone. The Excel (row 596) format is `CCYY-MM-DDThh:mm:ss` — no fractional seconds, no timezone. Also check it should be the real send time, not the message time.

**7. Do not send empty tags.**
`<reason />` is sent empty on all 23 (Optional — delete it). Plus the patient block empties below. Readme rule: no value → delete the tag.

---

## Same as putEvent (the patient block is identical)

The `<patient>` block is the same as in putEvent, with the same problems:

- `msgType` missing (Mandatory) — add `<msgType>Clinical</msgType>`.
- Empty `<address/>`, `<phone/>`, `<race/>`, `<language/>`, `<occupation/>` — delete them.
- `race` is Mandatory but empty — needs a real NHDD code (wait for the NHDD list).
- `gender`, `nationality`, `maritalStatus` — waiting for the official NHDD code list. Do not guess.
- `type` hardcoded `SP` but most IDs are passports — set from the real document type.
- `title` — send `Mr`, not `Mr.`.
- `msgID` — send the full id, not cut to 20 characters.
- Doctor number `03009J` (used in `creator`, `author`, and `requester > practitoner`) — Singapore MCR starts with `M` (e.g. `M03009J`). Check and add the `M`.

See `putEvent_Developer_Fixes_2026Sep01.md` for detail.

---

## Please confirm

**A. `composition > identifier`** — the Excel (row 70) says it must be unique and stay the **same** if the referral is later amended or cancelled. Confirm it is saved and reused.

**B. Attachment-only** — the referral PDF is sent as a file attachment. Confirm with NEHR that this is accepted for this clinic (same as putCardiology).

---

## Important — not an XML issue (same as putEvent)

This endpoint is open on plain `http://` with **no login** and returns **real patient referral letters, names, ID numbers and attached PDFs**. Please put it behind the secure channel and use fake test data before more testing.