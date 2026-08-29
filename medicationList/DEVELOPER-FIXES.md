# Patient Medication List (putPatientMedicationList) — What To Fix

**For the developer.** Simple English. Fix number 1 first, then 2, then 3.

| | |
|---|---|
| Correct example file | `putPatientMedicationList-corrected.xml` — in this folder. Copy the shape from it. |
| More detail (harder English) | `FIXES.md` — in this folder. |
| Check your file | `python3 tools/check_xml.py your_file.xml` |

⚠️ **Do putEvent first.** This service needs the event id that putEvent makes.

⚠️ **Stop and read number 20 before you start.** The drug code and the drug name in your data are **two different medicines**. This is a patient safety problem. Please tell the clinic today.

---

## The list

| # | What to fix | How hard |
|---|---|---|
| 1 | One `<putPatientMedicationList>` per message | Easy |
| 2 | Change the namespace | Easy |
| 3 | Small letter at the start of tag names — **but not all** | Easy |
| 4 | `MedicationList` must be `list` | Easy |
| 5 | Delete the `<Entries>` box | Easy |
| 6 | ⭑ Delete `msgType` — this service has none | Easy |
| 7 | Delete `patientMergeType`, `<Id>`, empty tags | Easy |
| 8 | `<name><name>` must be `<name><value>` | Easy |
| 9 | Add `list/identifier` — missing | Easy |
| 10 | Add `CMISAvailability` — missing | Easy |
| 11 | `encounter/identifier` must be the event id | Medium |
| 12 | Put the tags in the right order (2 places) | Medium |
| 13 | Delete the whole `supportingInformation` block | Easy |
| 14 | Delete the whole `notesforHCP` block | Easy |
| 15 | `msgID` — send the full ID, do not cut it | Easy |
| 16 | Date and time format | Easy |
| 17 | `entry/date` must have a time, not only a date | Easy |
| 18 | Doctor number needs `M` in front | Easy |
| 19 | Add 4 missing codes (we found them for you) | Easy |
| 20 | **Drug code does not match drug name** 🛑 | Need clinic |
| 21 | Fix 4 wrong text values and 1 wrong codeset | Easy |
| 22 | Delete `strengthUnit` — it holds a strength, not a unit | Easy |
| 23 | Delete `codingSchemeVersion` `2.29` | Easy |
| 24 | `race` is empty | Need data |
| 25 | `gender` — **do not change yet** | Wait |

---

## 1. One `<putPatientMedicationList>` per message ⬅ fix this first

**Now (wrong):**
```xml
<ArrayOfPutPatientMedicationList>
  <PutPatientMedicationList> ... </PutPatientMedicationList>
</ArrayOfPutPatientMedicationList>
```

**Change to:**
```xml
<putPatientMedicationList xmlns="http://www.mohh.com/nehr">
  ...
</putPatientMedicationList>
```

NEHR reads one patient at a time. There is no box for many patients.

---

## 2. Change the namespace

**Now (wrong):** `xmlns:NEHR="http://www.synapxe.sg/nehr/putEvent"`

Two problems: the address is wrong, **and** it says `putEvent` — which is a different service. Also you write `xmlns:NEHR=` but never use `NEHR:` on any tag.

**Change to:** `xmlns="http://www.mohh.com/nehr"` — with no `:NEHR` part.

---

## 3. Small letter at the start of tag names — but NOT all ⚠️

**Change these to small first letter:**
`ControlHeader` `Patient` `Encounter` `Status` `Code` `Source` `Practitioner` `PractitionerRole` `ManagingOrganization` `ReviewedUpon` `SourceOfMedicationList` `Entry` `Item` `MedicationReference` `Product` `Form` `Dosage` `Route` `Timing` `QuantityRange` `Low` `Unit` `InformationSource`

**⭑ But KEEP this one with a big letter:**
```xml
<MedicationStatement>   <!-- correct, do NOT change -->
```

**Why:** NEHR really uses a big `M` for this one. It is one of the few. If you write a rule like "make every first letter small", you will break this tag.

**Please do not write a rule to change letters.** Copy the exact tag names from the example file.

---

## 4. `MedicationList` must be `list`

**Now (wrong):** `<MedicationList>`
**Change to:** `<list>`

This is a different name, not only different letters.

---

## 5. Delete the `<Entries>` box

**Now (wrong):**
```xml
<MedicationList>
  <Entries>
    <Entry> ...medicine 1... </Entry>
  </Entries>
</MedicationList>
```

**Change to:**
```xml
<list>
  <entry> ...medicine 1... </entry>
  <entry> ...medicine 2... </entry>
</list>
```

There is no `Entries` level in NEHR. Someone added it. Same mistake as `Problems` in Visit Diagnosis and `MedicationItems` in the two other medicine services.

---

## 6. ⭑ Delete `msgType` — this service has none

```xml
<msgType>Clinical</msgType>   <!-- DELETE -->
```

`msgType` exists **only in putEvent**.

> ### This service has the strictest header of all 15
>
> | | putEvent | Visit Diagnosis | **This service** |
> |---|---|---|---|
> | `msgType` | yes | no | **no** |
> | How many `srcApplication` values allowed | 27 | 39 | **only 11** |
>
> The 11 allowed values are all big public hospital systems. `MDX-GCMS` is not one of them, and there is **no private clinic system in the list at all**.
>
> The clinic has asked NEHR: is this service open to private clinics? **Wait for that answer before spending a lot of time here.**

---

## 7. Delete `patientMergeType`, `<Id>`, empty tags

| Delete this | Why |
|---|---|
| `<patientMergeType>OBSOLETE</patientMergeType>` | Does not exist in this service. |
| `<Id>` inside the control header | Your internal ID. NEHR has no such field. |
| `<contactDetails/>` `<race/>` `<occupation/>` `<MedicalAlert/>` `<DrugAllergy/>` `<ReasonForUseCodeableConcept/>` `<Value/>` | Empty tags are not allowed. No value → do not write the tag. |

---

## 8. `<name><name>` must be `<name><value>`

```xml
<name><value>TAN AH KOW</value></name>
```

---

## 9. Add `list/identifier` — missing

This field is **required** and you do not send it at all.

You **do** send a GUID, but in the wrong place — inside `encounter/identifier`. We think that GUID is really the list ID. It looks the same as the record IDs in putEvent and Visit Diagnosis.

**Change to:**
```xml
<list>
  <identifier>af6dbfee-....-9151</identifier>   <!-- move the GUID here -->
```

⚠️ **Please confirm this is right.** This number decides what NEHR overwrites. Same identifier + higher `msgSequenceID` = NEHR replaces the old list.

---

## 10. Add `CMISAvailability` — missing

**Required.** You do not send it.

```xml
<CMISAvailability>0</CMISAvailability>
```

`0` = CMIS was not available. `1` = it was available.

We used `0` because a private clinic probably does not check CMIS. **Please confirm with the clinic.**

---

## 11. `encounter/identifier` must be the event id

**Now (wrong):** a GUID
**Change to:** the same event id you sent in putEvent

```xml
<encounter>
  <identifier>0808202608252836813027674</identifier>
</encounter>
```

**Why:** This links the medication list to the visit. A GUID links to nothing.

---

## 12. Put the tags in the right order (2 places) ⚠️

**NEHR checks the order of tags.** Right order, wrong place = rejected.

**Inside `<list>` — correct order:**
```
identifier, code, source, encounter, status, date,
CMISAvailability, sourceOfMedicationList, reviewedUpon, entry
```
(Your order today: `encounter, date, status, code, reviewedUpon, sourceOfMedicationList, source, ...`)

**Inside `<MedicationStatement>` — correct order:**
```
identifier, sequenceNo, informationSource, dateAsserted, status,
supportingInformation, medicationReference, dosage
```
*(In the example file you will see only 7 of these — `supportingInformation` is deleted, see number 13. The order above is where it **would** go if the clinic ever sends it.)*

**Inside `<dosage>` — correct order:**
```
timing, route, quantityRange
```
(Your order today: `route, timing, quantityRange`)

Easiest way: copy the order straight from the example file.

---

## 13. Delete the whole `supportingInformation` block

This block is not real data. Please delete all of it.

**What is inside it now:** 6 different `unit` fields, all with the **same** value — `Non-reconciled PPL` on the codeset `Problem_List_Status_(NEHR)`. That is a **problem list** status code, put inside a **medicine quantity unit**, in a **medicine list**. It has no meaning.

It also has an impossible shape: `<DispenseRequest>` appears **inside `<Prescriber>`**. A prescriber can only hold an ID, a name and a role. Nothing can go inside it.

**What this means for your code:** somewhere you have a default object being written into every empty slot. Please find that and stop it. It will cause the same rubbish in other services.

Every field in this block is optional, and the clinic has no real order data behind this list. So delete the block. If the clinic wants this link later, build it from real data — do not bring this shape back.

---

## 14. Delete the whole `notesforHCP` block

Inside it, `medicationManagementIssues` sends the code `Active` on the codeset `Diagnosis_Status_(NEHR)`.

That is the wrong codeset completely. The correct codes are `1`, `2`, `3`, `999` on `Med_List_Mgmt_Issues_(NEHR)`.

The clinic has no such data, so delete the block.

---

## 15. `msgID` — send the full ID, do not cut it

**Now (wrong):** `MDX-GCMS-aa6d35bc-01d4-485e-9`

Do not put `MDX-GCMS-` in front. Do not cut at 29 letters (NEHR allows 50).

---

## 16. Date and time format

**Now (wrong):** `2026-08-08T22:42:12.1946075+08:00`
**Change to:** `2026-08-08T22:42:12+08:00`

Whole seconds only.

---

## 17. `entry/date` must have a time, not only a date

**Now (wrong):**
```xml
<date>2026-08-08</date>
```
**Change to:**
```xml
<date>2026-08-08T22:42:12+08:00</date>
```

**Why:** NEHR wants a date **and** a time here. A date alone is rejected.

---

## 18. Doctor number needs `M` in front

**Now (wrong):** `03009J` in `source` and in `informationSource`
**Change to:** `M03009J`

---

## 19. Add 4 missing codes — we found them for you ✅

You send only the text, with no code. **NEHR needs the code.** These 4 are checked and correct:

| Field | Your text | Add this code | Codeset name |
|---|---|---|---|
| `dosage/timing/code` | Twice a day | `229799001` | `Med_Frequency_(NEHR)` |
| `dosage/route` | Oral | `26643006` | `Med_RouteofAdmin_(NEHR)` |
| `dosage/quantityRange/unit` | Tab | `428673006` | `Med_List_Item_Dose_Unit_(NEHR)` |
| `medicationReference/product/form` | Tab | `421026006` | `Med_DoseForm_(NEHR)` |

**One rule to remember:** NEHR says `codingSchemeName` should only be sent **if there is a code**. Today you send the scheme name with no code. That is backwards. Either send both, or send only the text.

---

## 20. Drug code does not match drug name 🛑 PATIENT SAFETY

**Stop. Do not send this record.**

Your data says:

| Field | Value |
|---|---|
| `code` | `108401000133106` |
| `textDescription` | `Janumet XR 50/1000` |

We looked up that code in the official Singapore drug list:

| Code | Real medicine |
|---|---|
| `108401000133106` | JANUMET — **normal tablet** |
| `159576781000133104` | JANUMET **XR** — **slow release tablet** |

**So the code says one medicine and the name says a different medicine.** These two are not the same. They work differently in the body and the doctor gives them differently.

One of the two is wrong in the clinic system. **Until somebody decides which one, do not send this record.** A wrong medicine in the national record is worse than no record.

In the example file we wrote `TODO_SDD_CODE_CONFLICT` on purpose, so nobody can send it by mistake.

### This is not only one record

If the medicine picker in the clinic system can give a code and a name that do not match, then **the whole medicine list needs checking** — not only this patient. Please tell the clinic.

### It also changes the dose form

The example uses `421026006` (Tablet), because your text says `Tab`. But if the real medicine is XR, it must be `420627008` (Extended-Release Tablet). So fix the drug first, then the dose form.

---

## 21. Fix 4 wrong text values and 1 wrong codeset

| Field | Now (wrong) | Change to |
|---|---|---|
| `list/code` | code `PML` | code `1` |
| `list/code/textDescription` | `Patient Medication List` | `Patient's Medication List` |
| `sourceOfMedicationList/textDescription` | `Clinical Records (reconciled with patient self-report)` | `Clinical Records` |
| `reviewedUpon/textDescription` | `Outpatient (Post-Consultation)` | `Outpatient(Post-Consultation)` — **no space** |
| `quantityRange/unit` codeset name | `Med_UnitofMeasure_(NEHR)` | `Med_List_Item_Dose_Unit_(NEHR)` |

**Why:** the text must match the codeset word for word. `Med_UnitofMeasure_(NEHR)` belongs to the two other medicine services, not to this one.

---

## 22. Delete `strengthUnit` — it holds a strength, not a unit

**Now (wrong):**
```xml
<strengthUnit><textDescription>50/1000</textDescription></strengthUnit>
```

`50/1000` is a strength. It is not a unit like `mg`.

And it cannot go in `strength` either, because `strength` accepts only **one** number — but this medicine has **two** ingredients (50 mg + 1000 mg).

**Change to:** delete both `strength` and `strengthUnit`. Both are optional, and the two strengths are already inside the drug name.

*(The clinic is asking NEHR if this is acceptable for 2-ingredient medicines.)*

---

## 23. Delete `codingSchemeVersion` `2.29`

Delete it from `form`, `route`, `unit`, and everywhere else in this service.

`2.29` is the version of the Excel file we read the codes from. It is not a code version.

---

## 24. `race` — needs real data

Required by NEHR, but sent empty. We did not put a fake value. Someone must connect `race` from patient registration.

---

## 25. `gender` — do NOT change this yet ⚠️

Your system sends `C` / `D`. NEHR's template says `M`/`F`/`U`, but NEHR's own NHDD sample says `C` = Female, `D` = Male.

**So your value may already be correct.** Please wait for the answer. Do not convert.

---

## Do NOT change these — they are already correct

- `status` code `Final`
- `sourceOfMedicationList` code `2`
- `reviewedUpon` code `6`
- item `status` code `Active`
- `medicationReference/code/codingSchemeName` = `Singapore Drug Dictionary`
- The codeset names `Med_DoseForm_(NEHR)`, `Med_RouteofAdmin_(NEHR)`, `Med_Frequency_(NEHR)`
- `sequenceNo` = 1
- The nesting `source > practitioner > practitionerRole > managingOrganization > identifier`
- `MRNNumber` format `MDX-GCMS^13033541`
- `MedicationStatement` with a big `M` (see number 3)

---

## Check your work

```bash
python3 tools/check_xml.py your_file.xml
```

Run it after every change. When you see `RESULT: PASSED`, that file is good.
