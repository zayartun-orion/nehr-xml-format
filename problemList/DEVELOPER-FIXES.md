# putEvent (Visit / Encounter) — What To Fix

**For the developer.** Simple English. Fix number 1 first, then 2, then 3.

| | |
|---|---|
| Correct example file | `putEvent-corrected.xml` — in this folder. Copy the shape from it. |
| More detail (harder English) | `FIXES.md` — in this folder. |
| Check your file | `python3 tools/check_xml.py your_file.xml` |

**Fix this service first.** Every other service copies the patient part and the header part from here. And every other service needs the `event id` that this service creates.

---

## The list

| # | What to fix | How hard |
|---|---|---|
| 1 | One `<putEvent>` per message. Remove `<ArrayOfPutEvent>` | Easy |
| 2 | Change the namespace | Easy |
| 3 | Small letter at the start of every tag name | Easy |
| 4 | `<name><name>` must be `<name><value>` | Easy |
| 5 | Delete 5 fields that must not be sent | Easy |
| 6 | `msgID` — send the full ID, do not cut it | Easy |
| 7 | Date and time format | Easy |
| 8 | `eventStartDate` — send the real visit time | Medium |
| 9 | Doctor number needs `M` in front | Easy |
| 10 | Add 2 missing code fields | Easy |
| 11 | One codeset name is wrong | Easy |
| 12 | `race` is empty — needs real data | Need data |
| 13 | `gender` — **do not change yet** | Wait |

---

## 1. One `<putEvent>` per message ⬅ fix this first

Right now the file has one big box with 16 visits inside. NEHR does not accept this. **NEHR reads one visit at a time.** Send 16 separate messages.

**Now (wrong):**
```xml
<ArrayOfPutEvent>
  <PutEvent> ...visit 1... </PutEvent>
  <PutEvent> ...visit 2... </PutEvent>
</ArrayOfPutEvent>
```

**Change to (2 separate messages):**
```xml
<putEvent xmlns="http://www.mohh.com/nehr">
  ...visit 1...
</putEvent>
```

**Why:** NEHR's rule file says the message must start with `putEvent`. When it starts with `ArrayOfPutEvent`, NEHR stops at the very first line and reads nothing else. So this one problem hides all the others.

---

## 2. Change the namespace

**Now (wrong):**
```xml
<ArrayOfPutEvent xmlns:NEHR="http://www.synapxe.sg/nehr/putEvent">
```

**Change to:**
```xml
<putEvent xmlns="http://www.mohh.com/nehr">
```

**Why:** Two problems today. First, the address is wrong — NEHR uses `http://www.mohh.com/nehr`. Second, you wrote `xmlns:NEHR=` but then never used `NEHR:` on any tag. So all your tags belong to *no* group at all.

Use `xmlns=` with **no** `:NEHR` part. Then every tag inside belongs to NEHR automatically. Write it once on the top tag only.

> This is the same for all 7 services. Fix it one time in your shared code.

---

## 3. Small letter at the start of every tag name

**Now (wrong):** `<ControlHeader>` `<Patient>` `<Event>`

**Change to:** `<controlHeader>` `<patient>` `<event>`

**Why:** Big letter and small letter are different tags for NEHR. `<Patient>` is not the same as `<patient>`.

**Rule:** first letter small, later words keep a big letter — `<eventStartDate>`, `<attendingClinician>`.

---

## 4. `<name><name>` must be `<name><value>`

**Now (wrong):**
```xml
<name>
  <name>GATCHALIAN FERDINAND PAGUIO</name>
  <title>Mr.</title>
</name>
```

**Change to:**
```xml
<name>
  <value>GATCHALIAN FERDINAND PAGUIO</value>
  <title>Mr</title>
</name>
```

Two things: the inner tag is `value`, not `name`. And the title is `Mr` — **no dot**.

---

## 5. Delete 5 fields that must not be sent

| Delete this | Why |
|---|---|
| `<Id>` inside the control header | This is your own internal ID. NEHR has no such field. |
| `<patientMergeType>OBSOLETE</patientMergeType>` | Only allowed when joining two patient records. You send it on every message. Also the allowed words are `Obsolete` and `Survivor` — not `OBSOLETE` in big letters. **This alone makes NEHR reject the message.** |
| `<eventEndDate>` | Only for hospital discharge. This clinic is outpatient. |
| `<codingSchemeVersion>2.29</codingSchemeVersion>` (3 places) | `2.29` is the version of the Excel file we read the codes from. It is not a code version. Delete it everywhere in this service. |
| Empty tags: `<contactDetails/>` `<race/>` `<language/>` `<occupation/>` | NEHR's rule book says: if you have no value, **do not send the tag at all**. Also do not send `<occupation>NA</occupation>`. |

**Rule for empty tags — use this everywhere:** no value → do not write the tag.

---

## 6. `msgID` — send the full ID, do not cut it

**Now (wrong):**
```xml
<msgID>MDX-GCMS-eb763d6b-fc51-4c34-a</msgID>
```

**Change to:**
```xml
<msgID>eb763d6b-fc51-4c34-a471-9c2cebaa63d8</msgID>
```

Two problems:

1. **Do not put `MDX-GCMS-` in front.** NEHR's guide says the header must not contain the company name or the product name.
2. **Do not cut the ID.** Your code cuts at 29 letters. NEHR allows 50. When you cut it, two different messages can end up with the same ID.

---

## 7. Date and time format

**Now (wrong):** `2026-08-07T22:40:35.1946075+08:00`

**Change to:** `2026-08-07T22:40:35+08:00`

**Why:** NEHR wants `CCYY-MM-DDThh:mm:ss+08:00`. Whole seconds only. Delete the `.1946075` part.

Your code is printing the computer's full clock value. Cut it to seconds.

---

## 8. `eventStartDate` — send the real visit time

**Now (wrong):**
```xml
<eventStartDate>2026-08-07T22:40:35.269+08:00</eventStartDate>
```
This is 22:40 — the time your program **ran**. It is not the time the patient came.

**Change to:**
```xml
<eventStartDate>2026-08-07T10:13:11</eventStartDate>
```

**Why:** NEHR needs the real visit time. If you send the program time, every patient in the national record looks like they came at night.

**Good news — you already have the real time.** It is inside your own event id:

```
0708202610131124913034023
│      ││        ││
│      ││        └─ 13034023  = patient MRN
│      │└────────── 101311249 = 10:13:11.249  ← the real time
└──────┴─────────── 07082026  = 07 Aug 2026
```

So the source system knows the true time. It is just not being used here.

---

## 9. Doctor number needs `M` in front

**Now (wrong):** `<id>03009J</id>`
**Change to:** `<id>M03009J</id>`

**Why:** A Singapore doctor number (MCR) starts with `M`. NEHR's example is `M12345A`.

Please check the real number in the SMC register once, to be safe.

---

## 10. Add 2 missing code fields

**a. `serviceSpecialty` has no code**

```xml
<serviceSpecialty>
  <code>NEHR_SS_0002</code>
  <codingSchemeName>Service_Specialty_(NEHR)</codingSchemeName>
  <textDescription>Cardiology</textDescription>
</serviceSpecialty>
```
`code` is required. `NEHR_SS_0002` = Cardiology.

**b. `movementType` is missing completely**

```xml
<movementType>
  <code>1</code>
  <codingSchemeName>Visit_Type_(NEHR)</codingSchemeName>
  <textDescription>Primary Care Visit</textDescription>
</movementType>
```
NEHR's guide says: when `eventType` is `4` (outpatient visit), send `movementType` too.

---

## 11. One codeset name is wrong

**Now (wrong):** `<codingSchemeName>Event_Type_(NEHR)</codingSchemeName>`
**Change to:** `<codingSchemeName>Movement_Category_(NEHR)</codingSchemeName>`

**Why:** `Event_Type_(NEHR)` does not exist. The correct name is `Movement_Category_(NEHR)`.

The code `4` and the text you send are **already correct**. Only the name is wrong.

---

## 12. `race` — needs real data (not a code problem)

`race` is required by NEHR, but your system sends it empty.

We did **not** put a fake value in the example file. Someone must connect `race` from patient registration.

This is a **data problem**, not an XML problem. Please tell the clinic.

---

## 13. `gender` — do NOT change this yet ⚠️

Your system sends `C` and `D`. We are not sure yet if this is wrong.

| Where we looked | What it says |
|---|---|
| NEHR mapping template | `M` = Male, `F` = Female, `U` = Unknown |
| NEHR NHDD sample list | `C` = Female, `D` = Male, `E` = Unknown |
| Your system today | `C` and `D` |

**So your `C`/`D` may already be correct.** If you change all of them to `M`/`F` and your version was right, you will break every patient record.

**Please wait.** The clinic is asking NEHR which one is true. The example file shows `M` for now, but this is the value most likely to change.

The same warning is for `race`, `language`, `maritalStatus`, `occupation` and `title`.

---

## Do NOT change these — they are already correct

- `msgSequenceID`
- `dateOfBirth` format
- `institution` (HCI code)
- `cancellationNotice` = `false`
- `maritalStatus` code `2`
- `nationality`
- `MRNNumber` format `MDX-GCMS^13034023`
- `identification/type` = `SP` — **but see below**

---

## Two more things to check

**`identification/type`** — `SP` means Singapore NRIC. That is correct for `S...` and `T...` numbers. But 7 patients have IDs starting with `X`, `Y` or `B`. Those are foreign patients and need `FP` or `MP` instead. Please set this from the real document type, not fixed to `SP`.

**`recordIdentifier`** — you must **save** this number with the visit and send the **same** number again if the visit is changed or cancelled later. Do not make a new one each time you send.

---

## Check your work

```bash
python3 tools/check_xml.py your_file.xml
```

It tells you what is wrong in simple English. Run it after every change. When you see `RESULT: PASSED`, that file is good.
