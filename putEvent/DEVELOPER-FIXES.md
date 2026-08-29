# Visit Diagnosis (putPatientProblemList) — What To Fix

**For the developer.** Simple English. Fix number 1 first, then 2, then 3.

| | |
|---|---|
| Correct example file | `putPatientProblemList-VD-corrected.xml` — in this folder. Copy the shape from it. |
| More detail (harder English) | `FIXES.md` — in this folder. |
| Check your file | `python3 tools/check_xml.py your_file.xml` |

⚠️ **Do putEvent first.** This service needs the event id that putEvent makes.

⚠️ **This service is NOT the same as putEvent.** Three fields are different. See number 5 and number 6 below. If you copy the putEvent code, it will be wrong.

---

## The list

| # | What to fix | How hard |
|---|---|---|
| 1 | One `<putPatientProblemList>` per message | Easy |
| 2 | Change the namespace | Easy |
| 3 | Small letter at the start of every tag name | Easy |
| 4 | Delete the `<Problems>` box | Easy |
| 5 | ⭑ Delete `msgType` — this service has none | Easy |
| 6 | ⭑ Delete `patientMergeType` — this service has none | Easy |
| 7 | `<name><name>` must be `<name><value>` | Easy |
| 8 | Delete empty tags and `<Id>` | Easy |
| 9 | `eventId` — remove `evt-` | Easy |
| 10 | `msgID` — send the full ID, do not cut it | Easy |
| 11 | Date and time format | Easy |
| 12 | List time must not be older than the problem time | Easy |
| 13 | Doctor number needs `M` in front | Easy |
| 14 | Delete `codingSchemeVersion` — but **keep one** | Easy |
| 15 | `sequenceNo` must start at 1 | Easy |
| 16 | **Diagnosis has no code — the big problem** | Need clinic |
| 17 | `race` is empty | Need data |
| 18 | `gender` — **do not change yet** | Wait |

---

## 1. One `<putPatientProblemList>` per message ⬅ fix this first

**Now (wrong):**
```xml
<ArrayOfPutPatientProblemList>
  <PutPatientProblemList> ...patient 1... </PutPatientProblemList>
  <PutPatientProblemList> ...patient 2... </PutPatientProblemList>
</ArrayOfPutPatientProblemList>
```

**Change to (separate messages):**
```xml
<putPatientProblemList xmlns="http://www.mohh.com/nehr">
  ...patient 1...
</putPatientProblemList>
```

**Why:** NEHR reads one patient at a time. There is no box for many patients. NEHR stops at the first line and reads nothing else.

---

## 2. Change the namespace

**Now (wrong):**
```xml
xmlns:NEHR="http://www.synapxe.sg/nehr/putEvent"
```

**Change to:**
```xml
<putPatientProblemList xmlns="http://www.mohh.com/nehr">
```

Note: your file says `putEvent` in the address, but this is **not** putEvent. The correct address is the same for all services: `http://www.mohh.com/nehr`.

---

## 3. Small letter at the start of every tag name

**Now (wrong):** `ControlHeader` `Patient` `ProblemList` `Problem` `Category` `ProblemName` `ConditionType` `Status` `State` `ServiceSpecialty` `ListType` `ListStatus` `LastUpdatedBy` `CreatedBy` `UpdatedBy`

**Change to:** `controlHeader` `patient` `problemList` `problem` `category` `problemName` `conditionType` `status` `state` `serviceSpecialty` `listType` `listStatus` `lastUpdatedBy` `createdBy` `updatedBy`

Big letter and small letter are different tags for NEHR.

---

## 4. Delete the `<Problems>` box

**Now (wrong):**
```xml
<ProblemList>
  <Problems>
    <Problem> ... </Problem>
  </Problems>
</ProblemList>
```

**Change to:**
```xml
<problemList>
  <problem> ... </problem>
  <problem> ... </problem>
</problemList>
```

**Why:** There is no `Problems` level in NEHR. Someone added it. When a patient has 3 diagnoses, write `<problem>` 3 times, one after another.

> Your code does this same wrong thing in 3 other services (`Entries`, `MedicationItems`). Fix the idea once: **a repeating item does not need a box around it.**

---

## 5. ⭑ Delete `msgType` — this service has none

**Now (wrong):**
```xml
<controlHeader>
  ...
  <msgType>Clinical</msgType>   <!-- DELETE -->
</controlHeader>
```

**Why:** `msgType` exists **only in putEvent**. This service does not have it. Sending it here is an extra tag and NEHR rejects the message.

---

## 6. ⭑ Delete `patientMergeType` — this service has none

**Now (wrong):**
```xml
<identification>
  <patientMergeType>OBSOLETE</patientMergeType>   <!-- DELETE -->
</identification>
```

**Why:** In putEvent this field exists. **Here it does not exist at all.** Not even with the right value.

Also: putEvent allows 2 `<identification>` blocks. This service allows only 1.

> ### Important for your code
>
> putEvent and this service look the same but they are **not** the same.
>
> | | putEvent | This service |
> |---|---|---|
> | `msgType` | yes | **no** |
> | `patientMergeType` | yes | **no** |
> | how many `identification` | 1 or 2 | **1 only** |
>
> So **one shared function for "control header" and "patient" will not work.** You need a small difference per service. Please check each service's own rule file before you write it.

---

## 7. `<name><name>` must be `<name><value>`

**Now (wrong):**
```xml
<name><name>TAN AH KOW</name></name>
```
**Change to:**
```xml
<name><value>TAN AH KOW</value></name>
```

---

## 8. Delete empty tags and `<Id>`

| Delete this | Why |
|---|---|
| `<Id>` inside the control header | Your internal ID. NEHR has no such field. |
| `<contactDetails/>` `<race/>` `<maritalStatus/>` `<occupation/>` | Empty tags are not allowed. No value → do not write the tag. |

---

## 9. `eventId` — remove `evt-`

**Now (wrong):** `<eventId>evt-0808202608252836813027674</eventId>`
**Change to:** `<eventId>0808202608252836813027674</eventId>`

**Why:** This number must match the `event/id` you sent in putEvent **exactly, letter by letter**. With `evt-` in front, it matches nothing. We checked all 7 records — **0 of 7** can be linked to a visit. So NEHR receives a diagnosis with no visit.

⭑ **Careful with the spelling here.** In this service the tag is `eventId` — **small `d`**. In 9 other services it is `eventID` — **big `D`**. This service is the only one with a small `d`. Please check each service's own rule file.

---

## 10. `msgID` — send the full ID, do not cut it

**Now (wrong):** `MDX-GCMS-09df7627-1b13-41eb-9`
**Change to:** `09df7627-1b13-41eb-9xxx-xxxxxxxxxxxx` (the full ID)

Two problems: do not put `MDX-GCMS-` in front (NEHR says no company name in the header), and do not cut at 29 letters (NEHR allows 50).

---

## 11. Date and time format

**Now (wrong):** `2026-08-08T08:25:28.3681234+08:00`
**Change to:** `2026-08-08T08:25:28+08:00`

Whole seconds only. Delete the numbers after the dot.

---

## 12. List time must not be older than the problem time

**Now (wrong):**
```xml
<lastUpdatedDateTime>2026-08-08T08:25:57.906</lastUpdatedDateTime>  <!-- the list -->
  <updatedDateTime>2026-08-08T08:25:57.928</updatedDateTime>        <!-- the problem inside -->
```

The list says it was updated at `.906`. The problem inside says `.928`. So the problem is **newer than the list that contains it**. That cannot be true.

**Fix in your code:** the list time must be the same as, or later than, the newest problem inside it.

Cutting to whole seconds (number 11) hides this one by luck. Please still fix the logic.

---

## 13. Doctor number needs `M` in front

**Now (wrong):** `03009J` in `lastUpdatedBy`, `createdBy`, `updatedBy`
**Change to:** `M03009J`

All three places.

---

## 14. Delete `codingSchemeVersion` — but KEEP one ⚠️

**Delete** `<codingSchemeVersion>2.29</codingSchemeVersion>` from `category` and `serviceSpecialty`.
`2.29` is the version of the Excel file we read codes from. It is not a code version.

**But KEEP it on `problemName`:**
```xml
<problemName>
  <code>...</code>
  <codingSchemeName>SNOMED-CT</codingSchemeName>
  <codingSchemeVersion>Jul2024</codingSchemeVersion>   <!-- KEEP THIS -->
  <textDescription>...</textDescription>
</problemName>
```
Here it is **required**, and `Jul2024` is a real SNOMED version.

---

## 15. `sequenceNo` must start at 1

**Now (wrong):** `<sequenceNo>2</sequenceNo>` — but there is only 1 problem in the list.
**Change to:** `1` for the first problem, `2` for the second, `3` for the third.

---

## 16. Diagnosis has no code — the big problem 🛑

**This is the blocker. It cannot be fixed in your code.** The clinic must do this part.

**Now (wrong):**
```xml
<problemName>
  <code>0</code>
  <textDescription>Controlled BP/DM/Lipids</textDescription>
</problemName>
```

`code` is `0` in 6 of 7 records. In the 7th record the whole `ProblemName` tag is empty. So **no record has a real diagnosis code.**

NEHR requires a **SNOMED CT** code for every diagnosis. Not ICD. Not clinic codes. Not free text.

### Problem A: nobody has made the codes yet

A doctor must choose the right SNOMED code for each diagnosis. In the example file we wrote `TODO_SNOMED_CT_CONCEPT_ID` on purpose, so nobody can send it by accident.

**Please do not guess a code.** A wrong SNOMED code means a wrong disease in the national record.

### Problem B: one text holds many diseases

| What the doctor typed | How many diseases |
|---|---|
| `Controlled BP/DM/Lipids` | **3** (high blood pressure, diabetes, high fat) |
| `Controlled BP/Lipids` | **2** |
| `Stable CAD s/p PCI` | 1 disease + 1 past operation |

SNOMED uses **one code for one disease**. So `Controlled BP/DM/Lipids` must become **3 separate `<problem>` blocks**:

```xml
<problem>
  <sequenceNo>1</sequenceNo>
  <problemName><code>...high blood pressure code...</code>...</problemName>
</problem>
<problem>
  <sequenceNo>2</sequenceNo>
  <problemName><code>...diabetes code...</code>...</problemName>
</problem>
<problem>
  <sequenceNo>3</sequenceNo>
  <problemName><code>...high fat code...</code>...</problemName>
</problem>
```

Each one needs its own `id` and its own number in `sequenceNo`.

For `s/p PCI` (past operation), the operation part probably belongs in `notes`, not in the diagnosis.

**Until this is done, this service cannot pass NEHR testing.** NEHR's test list checks that diagnoses have SNOMED codes and that free text is blocked.

---

## 17. `race` — needs real data

Required by NEHR, but sent empty. We did not put a fake value.

Someone must connect `race` from patient registration. This is a **data problem**, not an XML problem.

---

## 18. `gender` — do NOT change this yet ⚠️

Your system sends `C` and `D`. The NEHR template says `M`/`F`/`U`, but the NEHR NHDD sample list says `C` = Female, `D` = Male.

**So your value may already be correct.** Please wait until the clinic gets the answer from NEHR. Do not convert all of them.

---

## Do NOT change these — they are already correct

- `listType` = `Visit Diagnosis List` (exact spelling and letters)
- `listStatus` code `Final`
- `category` code `2`
- `conditionType`, `status`, `state` — all on the right codesets
- `serviceSpecialty` `NEHR_SS_0002` / Cardiology
- `problemName/codingSchemeName` = `SNOMED-CT` (with the dash) and version `Jul2024`
- `userInstitution` = HCI code
- `recordIdentifier`
- The **order** of the tags is already correct

---

## Check your work

```bash
python3 tools/check_xml.py your_file.xml
```

Run it after every change. When you see `RESULT: PASSED`, that file is good.
