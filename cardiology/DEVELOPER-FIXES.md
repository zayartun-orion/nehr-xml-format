# putCardiology — Fixes To Make

**For the developer. Simple English.** Checked against the Excel mapping template `putCardiology_v0.2_20251224.xlsx` — that file is the rule.

Endpoint checked: `http://ec2-47-131-196-190.ap-southeast-1.compute.amazonaws.com/nehr/xml/cardiology` (PUT). 16 messages, each with an attached PDF report.

Good news first: the codes are in good shape here. All six code-set names are correct (`Document_Type_(NEHR)`, `Document_Status_(NEHR)`, `Cardiology_Procedure_Category_(NEHR)`, `Attachment_Category_(NEHR)`, `Attachment_Content_Type_(NEHR)`, `Attachment_Language_(NEHR)`), each with a real code. The `eventId` value matches putEvent on all 16. The main problems are two spelling mistakes, one capital letter, field order, and a few empty fields.

---

## Fix now

**1. Spelling: `langauge` → `language`.**
Inside `<fileAttachment>` you send `<langauge>` on all 16 messages. The Excel (row 478) spells it `language`. NEHR will not recognise the misspelled tag, and this field is **Mandatory**. Fix the spelling.

**2. Spelling: `titile` → `title`.**
Inside `<fileAttachment>` you send `<titile>` on all 16 messages. The Excel (row 486) spells it `title`. This one matters extra: because you send the report as an attachment only (no discrete text — see "Please confirm"), the Excel says the `title` is what NEHR shows on screen for the attachment. With the wrong spelling, that title is lost. Fix it.

**3. Capital letter: `eventId` → `eventID`.**
Inside `<document>` you send `<eventId>`. The Excel (row 70) names it `eventID` (capital ID). The value is correct — just fix the tag name. XML tags are case-sensitive.

**4. `document` field order is wrong.**
You send `<cardiologyReports>` as the **first** child of `<document>`. The Excel puts it **last**, after `author`. The Excel order is:
id, lastUpdatedTime, eventID, remarks, institution, accessionNumber, docType, status, author, cardiologyReports.
Move `cardiologyReports` to the end of `<document>`. (NEHR most likely checks this order through the XSD.)

**5. `lastUpdatedTime` is empty but Mandatory.**
You send `<lastUpdatedTime />` on all 16. The Excel (row 69) marks it **Mandatory**. Put the real last-updated time, in the format `CCYY-MM-DDThh:mm:ss`.

**6. `cardiologyReport > type > textDescription` is empty.**
You send `<textDescription />` while the `type` code is filled (code `2`). When the code is present, the Excel (row 104) makes the description Mandatory. Put the description text (e.g. the procedure category name), or drop the whole `type` block if there is nothing to send.

**7. Date-time format on the report dates.**
`startDateTime` and `reportDateTime` are sent as `2026-09-01T19:02:04.2381552+08:00` — with fractional seconds and a timezone. The Excel (rows 92, 93) format is `CCYY-MM-DDThh:mm:ss` — no fractional seconds, no timezone. Remove the `.2381552` and the `+08:00`. Also check these should be the **real** report times, not the send time (all currently show `19:02`).

**8. Do not send empty tags.**
Empty tags to remove: `<lastUpdatedTime />` (fix 5 — fill instead), `<textDescription />` in the report type (fix 6), and the patient block empties below. The Readme says: no value → delete the tag (unless Mandatory, then fill it).

---

## Same as putEvent (the patient block is identical)

The `<patient>` block is the same as in putEvent, with the same problems. Apply the putEvent fixes here too:

- `msgType` missing (Mandatory) — add `<msgType>Clinical</msgType>`.
- Empty `<address/>`, `<phone/>`, `<race/>`, `<language/>`, `<occupation/>` — delete them.
- `race` is Mandatory but empty — needs a real NHDD code (wait for the NHDD list).
- `gender`, `nationality`, `maritalStatus` — waiting for the official NHDD code list. Do not guess.
- `type` hardcoded `SP` but most IDs are passports — set from the real document type.
- `title` — send `Mr`, not `Mr.`.
- `msgID` — send the full id, not cut to 20 characters.
- Doctor number `03009J` (used in `author > id` and `primaryOperator > id`) — Singapore MCR starts with `M` (e.g. `M03009J`). Check and add the `M`.

See `putEvent_Developer_Fixes_2026Sep01.md` for detail.

---

## Please confirm

**A. Attachment-only documents (no `<content>` / `<Composition>`).**
You send the report as a PDF attachment only — there is no `<content>`/`<Composition>` segment with discrete text. The Excel allows this, but only if the attachment `title` is present so NEHR can render it. So this is fine **once Fix 2 (the `titile` spelling) is done**. Please confirm with NEHR that attachment-only cardiology documents are accepted for this clinic.

**B. `document > id`** — this is the document's unique record id (a UUID, e.g. `121685dd-...`). The Excel (row 68) says it must be unique and stay the **same** if the document is later amended or cancelled. Confirm it is saved and reused.

---

## Important — not an XML issue (same as putEvent)

This endpoint is open on plain `http://` with **no login** and returns **real patient records and their attached cardiology report PDFs**. Please put it behind the secure channel and use fake test data before more testing.