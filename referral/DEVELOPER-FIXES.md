# Referral Notes (putComposition) — What To Fix

**For the developer.** Simple English. Fix number 1 first, then 2, then 3.

| | |
|---|---|
| Correct example file | `putComposition-RefNotes-corrected.xml` — in this folder. Copy the shape from it. |
| More detail (harder English) | `FIXES.md` — in this folder. |
| Check your file | `python3 tools/check_xml.py your_file.xml` |

⚠️ **Do putEvent first.** This service needs the event id that putEvent makes.

**The service name is `putComposition`.** There is no service called "putReferral". Referral notes are one kind of document inside `putComposition`. You already use the right name — good.

---

## ✅ First — thank you. Your new file is much better

We compared your old file (7 Aug) with your new one (19 Aug). You fixed a lot:

| | Old | New |
|---|---|---|
| Namespace address | wrong | ✅ correct |
| `licenseeNumber` | missing | ✅ added |
| `patientMergeType` | sent | ✅ removed |
| `<Id>` in the header | sent | ✅ removed |
| `msgType` | sent | ✅ removed (correct — this service has none) |
| Most tag names | Big letter | ✅ small letter |

**Only two small things went backwards:**

1. The namespace address is correct now, but it is still written the wrong way. See number 2.
2. `msgID` was cut at 29 letters before. Now it is cut at **20**. See number 12.

The rest of this list is mostly about **this service's own shape**, which is harder than the other six.

---

## The list

| # | What to fix | How hard |
|---|---|---|
| 1 | **The referral letter PDF is missing** 🛑 | Big job |
| 2 | Namespace: change `xmlns:NEHR=` to `xmlns=` | Easy |
| 3 | One `<putComposition>` per message | Easy |
| 4 | **All 5 sections use the same tag: `<section>`** | Medium |
| 5 | Delete the `<Others>` box | Easy |
| 6 | ⭑ Two tag names that look wrong but are correct | Important |
| 7 | ⭑ `compositionPractitioner` must be `practitioner` | Easy |
| 8 | ⭑ `requester` and `recipient` are not the same | Medium |
| 9 | Small letter at the start of tag names | Easy |
| 10 | ⭑ `<langauge>` is spelled wrong again | Easy |
| 11 | Wrong codeset on 4 sections | Easy |
| 12 | `msgID` — do not cut it | Easy |
| 13 | Delete extra tags and empty tags | Easy |
| 14 | 60 notes items are empty copies | Medium |
| 15 | `"N/A"` is not allowed | Need clinic |
| 16 | Attachment: wrong code and 2 empty texts | Easy |
| 17 | Date and time format | Easy |
| 18 | `encounter/identifier` must be the event id | Medium |
| 19 | Doctor number needs `M` in front | Easy |
| 20 | `identification/type` is wrong for foreign patients | Need clinic |
| 21 | `race` is empty | Need data |
| 22 | `gender` — **do not change yet** | Wait |

---

## 1. The referral letter PDF is missing 🛑 ⬅ the big job

**Now (wrong):**
```xml
<file />      <!-- EMPTY in all 20 records -->
```

Your message builds a full attachment box — it says the type is PDF, the language is English, the title is "Referral Letter" — and then sends **no letter inside**.

`file` is **required**. NEHR would get a referral that promises an attachment and gives nothing.

### ✅ You already know how to do this

**Your Cardiology service does attachments correctly.** We checked all 10 records there — every PDF was real, and the `size` was right. It is the best part of any file you have sent us.

**So copy that code here.** Two rules:

- `<file>` = the PDF turned into base64 text.
- `<size>` = the size of the **real PDF**, not the size of the base64 text.

---

## 2. Namespace: change `xmlns:NEHR=` to `xmlns=`

**Now (wrong):**
```xml
<PutComposition xmlns:NEHR="http://www.mohh.com/nehr">
```

**The address is correct now.** But it is written as a *prefix* called `NEHR`, and you never use `NEHR:` on any tag. So the tags still belong to nothing.

**Change to:**
```xml
<putComposition xmlns="http://www.mohh.com/nehr">
```

Delete the `:NEHR` part. That is the whole fix. Then every tag inside belongs to NEHR automatically.

---

## 3. One `<putComposition>` per message

**Now (wrong):**
```xml
<PutComposition>
  <putComposition> ...referral 1... </putComposition>
  <putComposition> ...referral 2... </putComposition>
</PutComposition>
```

**Change to (separate messages):**
```xml
<putComposition xmlns="http://www.mohh.com/nehr">
  ...referral 1...
</putComposition>
```

The inner name is already correct. Just remove the outer `<PutComposition>` box and send one at a time.

---

## 4. All 5 sections use the same tag: `<section>` ⚠️

This is the biggest shape change.

**Now (wrong)** — you made a different tag name for each section:
```xml
<CodeSection> ... </CodeSection>
<PatientInformationSection> ... </PatientInformationSection>
<ClinicalInformationSection> ... </ClinicalInformationSection>
<ReferralInformationSection> ... </ReferralInformationSection>
<AttachmentSection> ... </AttachmentSection>
```

**Change to** — all five are `<section>`:
```xml
<section> ... </section>
<section> ... </section>
<section> ... </section>
<section> ... </section>
<section> ... </section>
```

**Why:** NEHR has only **one** section tag. What makes a section different is the `<code>` and `<title>` **inside** it — not the tag name.

**The inside part too.** Every section holds exactly one `<content>`:

| You write | Change to |
|---|---|
| `<PatientInformationContent>` | `<content>` |
| `<clinicalInformationContent>` | `<content>` |
| `<referralInformationContent>` | `<content>` |
| `<attachmentContent>` | `<content>` |

**Each section = 1 title + 1 code + 1 content.** Nothing else.

> ### A hint about your code
>
> In the same file you write `<Other>` **120 times** and `<other>` **80 times**. Same tag, two spellings. Also `<Code>` 200 times and `<code>` 620 times.
>
> One shared function cannot do that. It looks like **each section is written by its own separate code block**.
>
> If you write **one** function that makes a `<section>`, and call it 5 times with different values, then this problem, the naming problem and the ordering problem all disappear together. That is the smallest amount of work with the biggest result.

---

## 5. Delete the `<Others>` box

**Now (wrong):**
```xml
<PatientInformationContent>
  <Others>
    <Other> ... </Other>
    <Other> ... </Other>
  </Others>
</PatientInformationContent>
```

**Change to:**
```xml
<content>
  <other> ... </other>
  <other> ... </other>
</content>
```

There is no `Others` level. And note you only use it in **one** section — the other sections already leave it out.

---

## 6. ⭑ Two tag names that LOOK wrong but are CORRECT ⚠️⚠️

**Please read this one carefully. It will waste your time if you miss it.**

| You write (looks correct) | NEHR wants (looks wrong) |
|---|---|
| `<practitioner>` | **`<practitoner>`** ← no second `i` |
| `<organization>` | **`<organisation>`** ← `s`, not `z` |

**These are not our mistakes.** We checked twice:

- NEHR's schema file spells them that way.
- NEHR's mapping template spells them that way too (rows 558, 562, 568, 572).

Two different NEHR files agree. So this is what NEHR wants.

It looks like you saw the strange spelling and "fixed" it to normal English. Please change it back.

### ⚠️ But `practitoner` is only in TWO places

```xml
<!-- Inside requester and recipient -> NO second "i" -->
<requester>
  <practitoner>              <!-- correct here -->
    <identifier>M03009J</identifier>
    <name>PHILIP KOH</name>
  </practitoner>
</requester>

<!-- Inside creator and author -> normal spelling -->
<creator>
  <practitioner>             <!-- correct here -->
    <identifier>M03009J</identifier>
    <name>PHILIP KOH</name>
  </practitioner>
</creator>
```

**So the same idea has two spellings in two places in the same message.** Do not use one shared function for both. Check the example file each time.

---

## 7. ⭑ `compositionPractitioner` must be `practitioner`

**Now (wrong):**
```xml
<author>
  <compositionPractitioner>
    <identifier>03009J</identifier>
    <name>PHILIP KOH</name>
  </compositionPractitioner>
</author>
```

**Change to:**
```xml
<author>
  <practitioner>
    <identifier>M03009J</identifier>
    <name>PHILIP KOH</name>
  </practitioner>
</author>
```

Inside `author` the tag is `practitioner` — exactly the same as inside `creator`. There is no tag called `compositionPractitioner`.

---

## 8. ⭑ `requester` and `recipient` are NOT the same ⚠️

They look the same. They are not.

| | `requester` | `recipient` |
|---|---|---|
| `identifier` | ✅ allowed | ✅ allowed |
| `name` | ✅ allowed | ✅ allowed |
| **`type`** | ❌ **not allowed** | ✅ allowed |

**Now (wrong)** — you send `<type>` inside requester:
```xml
<requester>
  <organization>
    <identifier>9403258</identifier>
    <name>The Heart Clinic</name>
    <type>                                        <!-- ❌ DELETE -->
      <code>Cardiology</code>
      <codingSchemeName>Document_Type_(NEHR)</codingSchemeName>
      <textDescription>Cardiology report</textDescription>
    </type>
  </organization>
</requester>
```

**Change to:**
```xml
<requester>
  <practitoner>
    <identifier>M03009J</identifier>
    <name>PHILIP KOH</name>
  </practitoner>
  <organisation>
    <identifier>9403258</identifier>
    <name>The Heart Clinic</name>
  </organisation>
</requester>
```

**Two problems there:** `type` is not allowed in requester at all, **and** the codeset was wrong anyway — `Document_Type_(NEHR)` describes a *document*, not a *clinic*.

**Your `recipient` is correct.** Do not change it — code `2` on `Organisation_Type_(NEHR)` is right.

**One more:** inside `referralRequest`, the `<encounter>` holds **only** `<identifier>`. You send `<period>` inside it. Delete that.

---

## 9. Small letter at the start of tag names

| Now (wrong) | Change to | How many |
|---|---|---|
| `<Other>` | `<other>` | 120 |
| `<Code>` | `<code>` | 200 |
| `<Extension>` | `<extension>` | 180 |
| `<NotesItem>` | `<notesItem>` | 180 |

**⚠️ Careful — do NOT write a rule that makes every first letter small.** This service has 3 tags that keep a big letter:

```
MRNNumber      VIPFlag      VVIPFlag
```

Copy the exact names from the example file instead.

---

## 10. ⭑ `<langauge>` is spelled wrong again

**Now (wrong):** `<langauge>`
**Change to:** `<language>`

**This is the same typo as in your Cardiology file.** Please search your whole project for `langauge` and fix every one. It is probably in a shared attachment function.

---

## 11. Wrong codeset on 4 sections

**Now (wrong)** — on Patient Information, Clinician Information, Referral Information, and Attachment:
```xml
<code>
  <code>Active</code>
  <codingSchemeName>Diagnosis_Status_(NEHR)</codingSchemeName>
  <textDescription>Patient Information</textDescription>
</code>
```

`Diagnosis_Status_(NEHR)` is for **diagnosis**. This is a **referral**. And look — the code says `Active` but the text says `Patient Information`. They do not even match each other.

**Change to** — send the text only:
```xml
<code>
  <textDescription>Patient Information</textDescription>
</code>
```

**Why only the text:** NEHR's template gives a code for the referral letter section (`4`), but gives no code for these three. The rule says send `codingSchemeName` **only when** you send a `code`. So send neither.

*(The clinic is asking NEHR whether code `1` "Referral Notes Item" should be used here instead. Wait for that answer before changing again.)*

**Your first section is correct** — code `4` on `Section_Code_(NEHR)`, "Referral Letter". Do not change it. It is also correct that this section has **no `<title>`**.

---

## 12. `msgID` — do not cut it

**Now (wrong):** `b8a1bdb8-6f42-4a3f-8` — this is only 20 letters.

NEHR allows **50**. A full ID is 36. So there is plenty of room.

**Change to:** the full ID, for example `b8a1bdb8-6f42-4a3f-8a1c-9f2e7d5b4c31`.

This got shorter than before (it was 29 letters). Please check where the cutting happens.

---

## 13. Delete extra tags and empty tags

| Delete this | Why |
|---|---|
| `<profession>` inside `creator/practitioner` | Not in this schema. A practitioner has only `identifier` and `name`. |
| `<system />` inside `creator` | Not in the schema, and empty. |
| `<address />` `<phone />` `<race />` `<language />` `<occupation />` `<reason />` | Empty. No value → do not write the tag. |

Also delete the whole `<contactDetails>` block when both `address` and `phone` are empty.

---

## 14. 60 notes items are empty copies

**Now (wrong):**
```xml
<NotesItem>
  <name>Full Name</name>
  <!-- no <text> ! -->
</NotesItem>
```

A notes item needs **both** `name` and `text`. Both are required.

**Look at the numbers.** In 20 records:

| Notes item name | How many times |
|---|---|
| **Full Name** | **80** ← 4 times per record! |
| NRIC | 20 |
| Gender | 20 |
| Referring Clinician | 20 |
| Clinic | 20 |
| Reason for Referral | 20 |

Every other name appears once per record. `Full Name` appears **4 times** — once with the real name, and 3 times empty.

**This is a loop bug.** Your loop writes the first field again and again instead of moving to the next one. It happens in two different sections.

Please find this loop. It may cause the same problem in other services.

---

## 15. `"N/A"` is not allowed

**Now (wrong):**
```xml
<notesItem>
  <name>Reason for Referral</name>
  <text>N/A</text>
</notesItem>
```

NEHR's rule book bans `NA` and `N/A` the same way it bans empty tags. A referral with no reason is not useful to the doctor who receives it.

**Please ask the clinic:** does the system record why the patient is being referred?

- **If yes** → send the real text here. Also fill `referralRequest/reason`, which takes a code.
- **If no** → this is a clinic system change, not an XML change.

---

## 16. Attachment: wrong code and 2 empty texts

**a. The code is wrong**

`Other_Code_(NEHR)` has 2 codes: `1` = Notes Item, `2` = File Attachment.

```xml
<code>1</code>                                   <!-- ❌ wrong -->
<textDescription>File Attachment</textDescription>
```
Your code says `1` (Notes Item) but your text says "File Attachment". **Change the code to `2`.**

**b. Two required texts are empty**

```xml
<contentType>
  <code>1</code>
  <textDescription />                            <!-- ❌ empty, required -->
</contentType>
```

**Change to:**

| Field | Code | Text to send |
|---|---|---|
| `contentType` | `1` | `PDF` |
| `language` | `EN` | `English` |

**c. The attachment title needs the date in front**

NEHR's template says the name should be `<DD-MMM-YYYY>_<Document Title>`.

```xml
<title>Referral Letter</title>                   <!-- now -->
<title>19-Aug-2026_Referral Letter</title>       <!-- change to -->
```

---

## 17. Date and time format

**Now (wrong):** `2026-08-19T18:28:58.6675902+08:00`
**Change to:** `2026-08-19T18:28:58+08:00`

Whole seconds only. Delete the numbers after the dot.

---

## 18. `encounter/identifier` must be the event id

**Now (wrong):** a GUID like `71b3b8e8-49e9-443d-81ce-4eaad906ef2a`

**Change to:** the same event id you sent in putEvent.

**Why:** this links the referral to the visit. A GUID links to nothing.

There are **two** places that need it — `composition/encounter/identifier` and `referralRequest/encounter/identifier`.

---

## 19. Doctor number needs `M` in front

**Now (wrong):** `03009J`
**Change to:** `M03009J`

Places to fix: `creator`, `author`, `requester/practitoner`, and inside the "Referring Clinician" notes text.

---

## 20. `identification/type` is wrong for foreign patients

**Now (wrong):**
```xml
<id>C8974431</id>
<type>SP</type>
```

`SP` means Singapore NRIC. But `C8974431` is not an NRIC, and this patient's nationality is `ID` (Indonesia). So this is almost certainly a **passport**, and needs `FP` or `MP`.

**Do not just change it to `FP`.** The right code depends on which document the patient actually showed at registration.

**Please ask the clinic** to set this from the real document type, instead of always sending `SP`.

---

## 21. `race` — needs real data

Required by NEHR, but empty in all 20 records. We did not put a fake value.

Someone must connect `race` from patient registration. This is a **data problem**, not an XML problem.

---

## 22. `gender` — do NOT change this yet ⚠️

Your system sends `C` and `D`. NEHR's template says `M`/`F`/`U`, but NEHR's own NHDD sample says `C` = Female, `D` = Male.

**So your value may already be correct.** Please wait for the answer. Do not convert.

---

## Do NOT change these — they are already correct

- `putComposition` as the inner tag name
- **No** `msgType` — correct, this service does not have it
- **No** `patientMergeType` — correct
- `licenseeNumber` — correct
- `composition/type` code `ReferralNote`
- `status` code `Final` with the text `Finalized`
- `composition/title`
- `custodian` — HCI code plus clinic name
- `referralRequest/status` code `1` (Authorized)
- `referralRequest/type` code `4` (Consultation)
- `specialty` `NEHR_SS_0002` / Cardiology
- `recipient/organisation/type` code `2` (Healthcare Provider)
- `serviceRequested`
- `contentType` code `1`, `language` code `EN`, `sequenceId`
- The first section having **no `<title>`** — that is the rule for the referral letter section
- The order of the tags inside `patient` and inside `referralRequest`

---

## Check your work

```bash
python3 tools/check_xml.py your_file.xml
```

Right now it reports **31 problems** on your file. When it prints `RESULT: PASSED`, that file is correct.

The tool now knows this service, and it knows the two strange spellings — so it will **not** tell you to change `practitoner` or `organisation`.
