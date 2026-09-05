# putPatientProblemList (Visit Diagnosis) — Fixes To Make

**For the developer. Simple English.** Checked against the Excel mapping template `putPatientProblemList_v0.2_(VD)_20251224.xlsx` — that file is the rule.

Endpoint checked: `http://ec2-47-131-196-190.ap-southeast-1.compute.amazonaws.com/nehr/xml/problemlist` (PUT). 23 messages, 1 diagnosis each.

Good news first: the `eventId` on every message **matches** the event id from putEvent for the same visit. That is correct — keep it that way.

Fix 1 to 6 now. Then read "Same as putEvent" and "Please confirm."

---

## Fix now

**1. `ListType` has the wrong capital letter.**
Now you send `<ListType>Visit Diagnosis List</ListType>` (capital L) in all 23 messages.
The Excel names this element `listType` (small l). Change it:
```xml
<listType>Visit Diagnosis List</listType>
```
XML tag names are case-sensitive, so `ListType` and `listType` are not the same tag.

**2. The diagnosis has no SNOMED code. This is the big one.**
The Excel marks `problemName > code` **Mandatory** and says: *"Diagnosis must be coded in SNOMED-CT."*
Right now:
- 19 messages send `<code>0</code>`. `0` is not a real SNOMED code.
- 4 messages (see Fix 3) send no code at all.

The real diagnosis is only in the text (e.g. "Controlled BP/Lipids"). NEHR needs the **SNOMED-CT code**, not just text.
You need a way to map each diagnosis to a SNOMED-CT code. If NEHR has agreed a placeholder in writing, send us that agreement — otherwise `0` will be rejected.

**3. Four messages are broken (the non-diagnostic visits).**
Messages for these visits — MRN `13003731`, `13012664`, `13034059`, `13002398` — are missing required parts:
- `problemName` is empty (no `code`, no `textDescription`). Both are **Mandatory**.
- `sequenceNo` is sent as `nil`. It is **Mandatory** — send a real number.
- `updatedDateTime` is **missing**. It is **Mandatory**.

The Excel says: even a non-diagnostic visit must send an administrative diagnosis with a proper non-diagnostic SNOMED-CT code and description. Do not send an empty `problemName`.

**4. `createdDateTime` is wrong — it uses the send time.**
On every message `createdDateTime` is stamped `2026-09-01T19:02:xx` — that is the moment you sent the message, not when the record was made.
Because of this, `createdDateTime` is **later** than `updatedDateTime` on all 19 messages.
The Excel says (row 184): `updatedDateTime` must be the **same or later** than `createdDateTime`. Right now it is earlier, so every message breaks the rule.
Fix: put the **real** record creation time in `createdDateTime`.

**5. Check the date-time format on the record dates.**
`createdDateTime`, `updatedDateTime` and `lastUpdatedDateTime` are sent with a timezone: `...+08:00`.
In the Excel, these three use the format `YYYY-MM-DdTHH:mm:sszzz` — **no timezone**. (Only `msgDateTime` keeps the `+08:00`.)
Please check with NEHR and, if needed, drop the `+08:00` from these three fields.

**6. Do not send empty tags or `nil`.**
The Excel Readme says: do not send empty tags or `NA` unless NEHR agreed it first.
This covers the empty `<problemName>` and the `<sequenceNo nil="true">` in Fix 3. No value → leave the whole tag out (unless the field is Mandatory, in which case it needs a real value).

---

## Same as putEvent (the patient block is identical)

The `<patient>` block in this message is the **same** as in putEvent and has the **same problems**. Apply the putEvent fixes here too:

- `msgType` is missing (Mandatory) — add `<msgType>Clinical</msgType>`.
- Empty tags: `<address/>`, `<phone/>`, `<race/>`, `<language/>`, `<occupation/>` — delete them.
- `race` is Mandatory but empty — needs a real NHDD code (wait for the NHDD list).
- `gender`, `nationality`, `maritalStatus` — still waiting for the official NHDD code list. Do not guess.
- `type` is hardcoded `SP` but most IDs are passports — set it from the real document type.
- `title` — send `Mr`, not `Mr.`.
- `msgID` — send the full id, not cut to 20 characters.
- Doctor number `03009J` — Singapore MCR starts with `M` (e.g. `M03009J`). Please check and add the `M`.

See `putEvent_Developer_Fixes_2026Sep01.md` for the detail on each.

---

## Please confirm

**A. `recordIdentifier`** — the Excel says (row 68): a new one per **visit**, unique in your system, and it must stay the **same** if the diagnosis list is later changed or cancelled. Is it saved with the visit and reused? It must be.

**B. `lastUpdatedDateTime` order** — the Excel says (row 95) this must be the same or later than every `updatedDateTime` in the message. On a few messages it is earlier (e.g. MRN `13021934`, `13012858`). Please check the logic.

---

## Important — not an XML issue (same as putEvent)

This endpoint is open on plain `http://` with **no login** and returns **real patient names, birth dates and ID numbers**. Please put it behind the secure channel and use fake test data before more testing.