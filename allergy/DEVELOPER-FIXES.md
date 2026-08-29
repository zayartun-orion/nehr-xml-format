# Allergy / Drug Reaction (putAllergyADR) — What To Build

**For the developer.** Simple English.

| | |
|---|---|
| Example file to copy | `putAllergyADR-reference.xml` — in this folder. |
| More detail (harder English) | `FIXES.md` — in this folder. |
| Check your file | `python3 tools/check_xml.py your_file.xml` |

---

## ⚠️ This one is different from the other 6

**There is nothing to fix here. There is nothing to fix because there is nothing yet.**

The web address `/nehr/xml/allergies` has **never given any data**. We tried again on 9 Aug — still nothing.

So this file is **not** a list of corrections. It is a **build guide**: it shows the exact shape you must make.

### 🛑 Ask this question before you start

**Is this service not built yet? Or is it built but has no data?**

We cannot tell from outside. When a service has no data, it says `404 There is no data provided.` — and **that is the same message** the working services give when they are empty. So the message tells us nothing.

**Please tell the clinic which one it is.** If it is not built, this is new work and needs planning time.

---

## What is real and what is not in the example file

| In `putAllergyADR-reference.xml` | Real? |
|---|---|
| The shape and the order of tags | ✅ real — from NEHR's own schema file |
| The codeset names | ✅ real |
| Every code value (`A`, `Y`, `03`, `2`…) | ✅ real — from NEHR's codeset file |
| The patient (marked `TODO_`) | ❌ not real — put your data here |
| The Penicillins / Rash allergy | ❌ not real — it is an **example** to show every required field |

---

## The list

| # | What to do | How hard |
|---|---|---|
| 1 | **Code fields have a different shape here** ⭑⭑ | Important |
| 2 | The text must match the codeset **exactly**, including big/small letters | Important |
| 3 | Three fields have a strange number format | Easy to get wrong |
| 4 | Fill every required field | Medium |
| 5 | Follow 5 rules that come from the clinic system, not the XML | Need clinic |
| 6 | Old codes must not be used | Medium |
| 7 | Same basics as the other services | Easy |

---

## 1. ⭑⭑ Code fields have a DIFFERENT shape here

**This is the one thing that will break your code.** If you copy your code-field function from any other service, every code field in this service will be wrong.

**In the other 6 services, a code field looks like this — flat:**
```xml
<status>
  <code>Final</code>
  <codingSchemeName>Document_Status_(NEHR)</codingSchemeName>
  <codingSchemeVersion>...</codingSchemeVersion>
  <textDescription>Finalized</textDescription>
</status>
```

**In THIS service it looks like this — inside a `<coding>` box:**
```xml
<verificationStatus>
  <coding>
    <system>CMIS_DRUG_ALLERGY_INDICATOR_(NEHR)</system>
    <version>...</version>
    <code>Y</code>
    <display>Yes</display>
  </coding>
</verificationStatus>
```

**The names change too:**

| Other 6 services | **This service** |
|---|---|
| `codingSchemeName` | **`system`** |
| `codingSchemeVersion` | **`version`** |
| `textDescription` | **`display`** |
| (flat) | **everything inside `<coding>`** |

There is also a `<text>` tag which is a **brother** of `<coding>`, not inside it.

> **What to do:** write a **second** function just for this service. Do not try to make one function do both.
>
> Two other services (`putLabResult`, `putComposition`) may also use this shape. Check them the same way when you reach them.

---

## 2. The text must match the codeset EXACTLY

NEHR's template repeats this on almost every code field:

> *All codified fields will be validated against CMIS codes and coded description, **including case-sensitivity checks**.*

So `display` is **not free text**. It must match the codeset word for word, and big letter for small letter.

**Example:**
```xml
<display>PENICILLINS</display>   <!-- ✅ correct — CMIS uses big letters -->
<display>Penicillins</display>   <!-- ❌ rejected -->
```

**What to do:** copy the text straight from the codeset. Do not re-type it. Do not make it nice-looking.

---

## 3. Three fields have a strange number format ⚠️

These are easy to get wrong because they look like normal numbers, but they are not.

**a. `informationSource > code` — has a zero in front**

```xml
<code>03</code>   <!-- ✅ correct -->
<code>3</code>    <!-- ❌ wrong -->
```
The codes are `01`, `02`, `03`, `04`, `10`.

**b. `exposureRoute > code` — always 3 digits**

```xml
<code>038</code>  <!-- ✅ correct — Oral -->
<code>38</code>   <!-- ❌ wrong -->
```

**c. `onset > onsetDateTime` — NOT a normal date**

```xml
<onsetDateTime>20260807</onsetDateTime>   <!-- ✅ YYYYMMDD, no dashes, no time -->
<onsetDateTime>2026-08-07T10:13:11+08:00</onsetDateTime>   <!-- ❌ wrong -->
```

**And when the date is unknown:**
```xml
<onsetDateTime>00000000</onsetDateTime>   <!-- ✅ eight zeros -->
```
Unknown is normal here. Most allergies were found long ago and nobody wrote the date.

> ⚠️ **Careful:** every other date in NEHR uses `2026-08-07T10:13:11+08:00`. **This one field is different.** Do not use your normal date function here.

---

## 4. Fill every required field

When you send one allergy, these fields **must** all be there:

`identifier` · `reportingFlag` · `creator > practitioner > name` · `verificationStatus` · `adverseReaction` · `encounter > serviceProvider` (both `identifier` and `name`) · `onset > onsetDateTime` · `recorder > practitioner` (`identifier`, `name`, `profession > code`) · `reportedDate` · `informationSource` · `reaction` · `drugInformation` (1 or more, with `drugType`, `substance` and `drugProbability`)

**Note:** a message may have **zero** allergies. That is allowed. But if there is one, all the fields above are needed.

**The order of the tags is fixed.** Copy the order from the example file.

### The codesets you need

All of them are in `Onboarding Pack/04. Onboarding Codeset Inventory/INT_2_1 Onboarding_CodeSetInventory_v2.29_20251224.xlsx`.

| Field | Codeset name | How many codes |
|---|---|---|
| `reportingFlag` | `CMIS_REPORTING_FLAG_(NEHR)` | 3 — `A` add, `U` update, `D` delete |
| `verificationStatus` | `CMIS_DRUG_ALLERGY_INDICATOR_(NEHR)` | 3 — **`Y` = allergy; `N` or `U` = drug reaction** |
| `informationSource` | `CMIS_INFORMATION_SOURCE_(NEHR)` | 5 |
| `recorder > profession` | `CMIS_REPORTER_PROFESSION_(NEHR)` | 4 — 1 Dentist, 2 Doctor, 3 Pharmacist, 4 Nurse |
| `reaction > ADRAlertIndicatorCode` | `CMIS_ADR_ALERT_INDICATOR_(NEHR)` | 3 |
| `reaction > severity` | `CMIS_REACTION_SEVERITY_(NEHR)` | 3 |
| `reaction > ADROutcome` | `CMIS_ADR_OUTCOME_(NEHR)` | 6 |
| `reaction > reactionOutcome` | `CMIS_REACTION_OUTCOME_(NEHR)` | 6 |
| `drugInformation > substance` | `CMIS_DRUG_(NEHR)` | **2,529** |
| `drugInformation > exposureRoute` | `CMIS_ADMINISTRATION_ROUTE_(NEHR)` | 69 |
| `drugInformation > drugProbability` | `CMIS_DRUG_PROBABILITY_(NEHR)` | 5 |
| `drugInformation > productName` | `CMIS_BRAND_(NEHR)` | **6,314** |

**`adverseReaction` is special.** It is **free text**, not a code. There is a list of 49 words (`CMIS_ADVERSE_REACTION_(NEHR)`), but that list is for the **clinic screen** — so the doctor can pick from it. What you send is just the text.

**These fields have NO codeset** — send only `text`, or do not send them: `type`, `clinicalStatus`, `category`, `criticality`, `code`, `reaction > manifestation`.

---

## 5. Five rules from NEHR — these are about the clinic system, not the XML

Please read these with the clinic before you build.

| Rule | What it means |
|---|---|
| **Drug reaction needs NRIC or FIN.** Passport is **not** accepted. | A patient with only a passport **cannot** have a drug reaction sent to NEHR at all. Several of this clinic's patients are foreign. **The clinic must decide what to do.** |
| **Only ONE suspected drug per record** (`drugType` = `S`). | Other drugs the patient was taking are `drugType` = `C`, and there can be many. |
| `drugProbability` is **required** when `drugType` = `S`. | |
| `reasonForChange` is **required** when `reportingFlag` is `U` or `D`. | Only when updating or deleting. |
| **Medical Alert and Drug Reaction cannot both be in one message.** | This service sends the drug reaction part only. |

**Also:** `drugType` uses big letters — `S` and `C`. Not `s` and `c`.

---

## 6. Old codes must not be used

The template has an **Expired Codes** sheet.

CMIS drug codes, route codes and brand codes each have a **start date** and an **end date**. NEHR only accepts a code if today is inside those dates.

**What to do:** when the clinic screen shows the drug list, it must **hide** the codes that have expired. It is not enough to check that the code exists.

---

## 7. Same basics as the other services

These are the same everywhere. Do them here too:

- One `<putAllergyADR>` per message. No box holding many patients.
- `xmlns="http://www.mohh.com/nehr"` on the top tag, with no `:NEHR` part.
- Small letter at the start of tag names.
- No empty tags. No value → do not write the tag.
- `msgID` — full ID, no `MDX-GCMS-` in front, do not cut it.
- Doctor number with `M` in front — `M03009J`.
- `<name><value>...</value></name>`, not `<name><name>`.
- **No `msgType`** — that field is only in putEvent.
- `encounter/identifier` must be the same event id as putEvent.

---

## One thing the clinic must answer first

**Does the clinic record allergies today at all?**

If allergies are only written as free text in the doctor's notes, then the clinic system must be changed **before** any XML work can start. You cannot make coded XML from free text.

Please check this before planning the work.

---

## One more note (for the clinic, not for you)

NEHR only allows 11 systems to send this service, and `MDX-GCMS` is not one of them. The list here is also **not the same** as the list for the Medication List service.

So one registration with NEHR may not be enough — it may be needed **per service**. The clinic is asking NEHR about this.

---

## Check your work

```bash
python3 tools/check_xml.py your_file.xml
```

Run it after every change. When you see `RESULT: PASSED`, that file is good.
