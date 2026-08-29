# Cardiology (putCardiology) — What To Fix

**For the developer.** Simple English. Fix number 1 first, then 2, then 3.

| | |
|---|---|
| Correct example file | `putCardiology-corrected.xml` — in this folder. Copy the shape from it. |
| More detail (harder English) | `FIXES.md` — in this folder. |
| Check your file | `python3 tools/check_xml.py your_file.xml` |

⚠️ **Do putEvent first.** This service needs the event id that putEvent makes.

---

## ✅ First, good news

**The PDF attachment part is the best work in any of the 7 services.** We checked all 10 records:

- The PDF turns back into a real PDF file every time. No broken files.
- `size` is the size of the **real PDF**, not the size of the encoded text. This is the easy thing to get wrong, and you got it right in all 10.
- `attachmentCategory` `1`, `contentType` `1`, `language` `EN` — all correct codes.

**Do not change the attachment encoding.** The problems in this service are names, order, times, and one code.

---

## The list

| # | What to fix | How hard |
|---|---|---|
| 1 | One `<putCardiology>` per message | Easy |
| 2 | Change the namespace | Easy |
| 3 | ⭑ Two tag names are spelled wrong | Easy |
| 4 | Small letter at the start of other tag names | Easy |
| 5 | Delete `msgType`, `patientMergeType`, `<Id>`, empty tags | Easy |
| 6 | `<name><name>` must be `<name><value>` | Easy |
| 7 | ⭑ `eventId` must be `eventID` — **big D** | Easy |
| 8 | ⭑ The reports must come LAST, not first | Medium |
| 9 | One code field uses the wrong codeset | Easy |
| 10 | All 3 times are the wrong time | Medium |
| 11 | `msgID` — send the full ID, do not cut it | Easy |
| 12 | Doctor number needs `M` in front | Easy |
| 13 | Add `primaryOperator/id` | Easy |
| 14 | `race` is empty | Need data |
| 15 | `gender` — **do not change yet** | Wait |

---

## 1. One `<putCardiology>` per message

**Now (wrong):**
```xml
<ArrayOfPutCardiology>
  <PutCardiology> ...report 1... </PutCardiology>
  <PutCardiology> ...report 2... </PutCardiology>
</ArrayOfPutCardiology>
```

**Change to:**
```xml
<putCardiology xmlns="http://www.mohh.com/nehr">
  ...report 1...
</putCardiology>
```

NEHR reads one report at a time. There is no box for many reports.

---

## 2. Change the namespace

**Now (wrong):** `xmlns:NEHR="http://www.synapxe.sg/nehr/putEvent"`

The address is wrong, and it says `putEvent` — a different service. Also `xmlns:NEHR=` is declared but `NEHR:` is never used on any tag.

**Change to:** `xmlns="http://www.mohh.com/nehr"` — with no `:NEHR` part.

---

## 3. ⭑ Two tag names are spelled wrong ⚠️

| You write | Correct spelling |
|---|---|
| `<Langauge>` | `<language>` |
| `<titile>` | `<title>` |

These are **typing mistakes**, not NEHR rules. NEHR spells both words correctly in its schema file **and** in its template. Please search your code for these two words and fix them.

**`title` is important.** This clinic sends a PDF with no report text. In that case, `title` is the name that the doctor sees on the NEHR screen. If the tag is spelled `titile`, **NEHR shows nothing** — the doctor sees an attachment with no name.

---

## 4. Small letter at the start of other tag names

**Now (wrong):** `ControlHeader` `Patient` `Document` `CardiologyReports` `CardiologyReport` `Name` `Type` `PrimaryOperator` `DocType` `Status` `Author` `FileAttachment` `AttachmentCategory` `ContentType` `ReportDateTime`

**Change to:** `controlHeader` `patient` `document` `cardiologyReports` `cardiologyReport` `name` `type` `primaryOperator` `docType` `status` `author` `fileAttachment` `attachmentCategory` `contentType` `reportDateTime`

---

## 5. Delete `msgType`, `patientMergeType`, `<Id>`, empty tags

| Delete this | Why |
|---|---|
| `<msgType>Clinical</msgType>` | `msgType` exists **only in putEvent**. Not here. |
| `<patientMergeType>OBSOLETE</patientMergeType>` | Does not exist in this service at all. |
| `<Id>` inside the control header | Your internal ID. NEHR has no such field. |
| `<contactDetails/>` `<race/>` `<language/>` `<occupation/>` — **inside `<patient>` only** | Empty tags are not allowed. No value → do not write the tag. |

> ⚠️ **Careful — there are TWO tags called `language` in this message.**
>
> | Where | What to do |
> |---|---|
> | Inside `<patient>` | It is empty → **delete it** |
> | Inside `<fileAttachment>` | It holds `EN` → **keep it**. This one is correct. |
>
> Do not delete both. Check the parent tag first.

---

## 6. `<name><name>` must be `<name><value>`

```xml
<name><value>TAN AH KOW</value></name>
```

---

## 7. ⭑ `eventId` must be `eventID` — big D ⚠️

**Now (wrong):**
```xml
<eventId>evt-0708202610475856713032562</eventId>
```
**Change to:**
```xml
<eventID>0708202610475856713032562</eventID>
```

Two things: **big `D`** in this service, and **remove `evt-`**. The value must match the `event/id` from putEvent letter by letter, or the ECG links to no visit.

---

## 8. ⭑ The reports must come LAST, not first ⚠️

**Now (wrong):** `cardiologyReports` is the **first** tag inside `document`.

**Correct order inside `<document>`:**
```
id, lastUpdatedTime, eventID, institution, docType, status, author, cardiologyReports
```

So `cardiologyReports` goes **last**. All the information about the document comes first, then the reports.

Easiest way: copy the order straight from the example file.

---

## 9. One code field uses the wrong codeset

**Now (wrong):**
```xml
<cardiologyReport>
  <type>
    <code>Cardiology</code>
    <codingSchemeName>Document_Type_(NEHR)</codingSchemeName>
```

**Change to:**
```xml
<cardiologyReport>
  <type>
    <code>2</code>
    <codingSchemeName>Cardiology_Procedure_Category_(NEHR)</codingSchemeName>
    <textDescription>Non Invasive</textDescription>
```

**Why:** `type` here means **what kind of heart test**, not what kind of document. You used the document codeset in the wrong place.

The 4 allowed codes are:

| Code | Meaning |
|---|---|
| 1 | Invasive |
| **2** | **Non Invasive** |
| 3 | Nuclear |
| 4 | Peripheral |

A 12-lead ECG is **non-invasive**, so code `2`. **Please ask the clinic** to confirm, and to tell you the code for any other heart test they do.

**Careful:** there is another field called `docType` in the same message. **That one is correct** and really does use `Document_Type_(NEHR)` with the code `Cardiology`. Do not change `docType`. Only `cardiologyReport/type` is wrong.

---

## 10. All 3 times are the wrong time ⚠️

`lastUpdatedTime`, `startDateTime` and `reportDateTime` all carry the time your program ran — about **22:40 at night** — instead of the real test time.

Look at the pattern. Real clinic tests all over the day, but all stamped within 33 seconds at night:

| Record | Time you send | Real ECG time |
|---|---|---|
| 1 | 22:40:39 | **10:52:16** |
| 2 | 22:40:43 | 10:14:47 |
| 3 | 22:40:45 | 14:18:57 |
| 9 | 22:41:10 | 09:12:45 |

**Where we found the real time:** it is inside the attachment file name — `UPG_20260807105216403` means 2026-08-07, 10:52:16.403.

We used the file-name time in the example. **But please do not do it that way in your code.** Reading a time out of a file name is not safe. The clinic system knows the real time — send that.

**Also:** `startDateTime` (when the test was done) and `reportDateTime` (when the report was written) are two different moments. Today you send the same value for both.

**And the format:** `2026-08-07T10:52:16+08:00` — whole seconds only. Delete the numbers after the dot.

---

## 11. `msgID` — send the full ID, do not cut it

**Now (wrong):** `MDX-GCMS-fe01fe9c-74c5-46f1-8`

Do not put `MDX-GCMS-` in front. Do not cut at 29 letters (NEHR allows 50).

---

## 12. Doctor number needs `M` in front

**Now (wrong):** `03009J` in `author`
**Change to:** `M03009J`

---

## 13. Add `primaryOperator/id`

Today `primaryOperator` has only a name, no ID. The MCR number is already in `author` in the same record.

```xml
<primaryOperator>
  <id>M03009J</id>          <!-- ADD -->
  <name><value>PHILIP KOH</value></name>
</primaryOperator>
```

Optional, but you already have the value.

---

## 14. `race` — needs real data

Required by NEHR, but sent empty. We did not put a fake value. Someone must connect `race` from patient registration.

---

## 15. `gender` — do NOT change this yet ⚠️

Your system sends `C` / `D`. NEHR's template says `M`/`F`/`U`, but NEHR's own NHDD sample says `C` = Female, `D` = Male.

**So your value may already be correct.** Please wait for the answer. Do not convert.

---

## Do NOT change these — they are already correct

- `docType` code `Cardiology` — see the warning in number 9
- `status` code `Final` with the text `Finalized` — you used the codeset word, not the code word. This is correct.
- `attachmentCategory` `1`, `contentType` `1`, `language` `EN`
- The base64 encoding of the PDF
- `size` = the size of the real PDF
- `sequenceNo`
- `institution` = HCI code
- `cardiologyReport/name` sending only `textDescription` ("12-lead ECG")
- `document/id`

---

## Two things the clinic must answer (not your job)

**1. Is a PDF alone enough?**

Today the clinic sends a PDF with no report text (`<content>`). NEHR's template is not clear about this — one row says the text is required, another row says the `title` is used *when the text is missing*, and the schema file says the text is optional.

The clinic is asking NEHR. **If NEHR says the text is required**, this becomes a much bigger job — a full report structure must be built. Please wait for that answer before planning the work.

**2. One NEHR file is missing.**

NEHR did not include a schema file called `Section.xsd` in the pack. We checked everything. Without it, the report-text part of this service cannot be fully checked by the tool.

The clinic has asked NEHR to send it. This only matters if the answer to question 1 is "text is required".

---

## One note about the source file

When we did this review on 9 Aug, the web address gave no data (`404 There is no data provided.`). So we used the copy saved on 7 Aug: `xml-fixes/source-xml/cardiology.xml` (10 records).

Please check the tag names again when the address is working.

---

## Check your work

```bash
python3 tools/check_xml.py your_file.xml
```

Run it after every change. When you see `RESULT: PASSED`, that file is good.
