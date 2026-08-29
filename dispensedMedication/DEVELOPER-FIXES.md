# Dispensed Medications (putDispensedMedications) — What To Fix

**For the developer.** Simple English. Fix number 1 first, then 2, then 3.

| | |
|---|---|
| Correct example file | `putDispensedMedications-corrected.xml` — in this folder. Copy the shape from it. |
| More detail (harder English) | `FIXES.md` — in this folder. |
| Check your file | `python3 tools/check_xml.py your_file.xml` |

⚠️ **Do putEvent first.** This service needs the event id that putEvent makes.

🛑 **Three required things are missing completely.** See numbers 1, 2 and 3.

⚠️ **Do NOT copy your Ordered Medications code for the medicine part.** The field names are different. See the box after number 1.

---

## The list

| # | What to fix | How hard |
|---|---|---|
| 1 | **No medicines at all — add them** 🛑 | Big job |
| 2 | `dispensingInstitution` is missing | Easy |
| 3 | `dispensedBy` is empty | Easy |
| 4 | One `<putDispensedMedications>` per message | Easy |
| 5 | Root name has an extra word `Patient` | Easy |
| 6 | Change the namespace | Easy |
| 7 | `MedicationDispensed` must be `medicationDispense` | Easy |
| 8 | Delete the `<MedicationItems>` box | Easy |
| 9 | ⭑ `eventId` must be `eventID` — **big D** | Easy |
| 10 | Small letter at the start of other tag names | Easy |
| 11 | Delete `msgType`, `patientMergeType`, `<Id>`, empty tags | Easy |
| 12 | `<name><name>` must be `<name><value>` | Easy |
| 13 | Put the tags in the right order | Medium |
| 14 | Add `orderID` — links dispense to order | Medium |
| 15 | Add `orderingInstitution` and `orderedBy/id` | Easy |
| 16 | `msgID` — send the full ID, do not cut it | Easy |
| 17 | Date and time format | Easy |
| 18 | `dispenseDateTime` is the wrong time | Medium |
| 19 | Doctor number needs `M` in front | Easy |
| 20 | `race` is empty | Need data |
| 21 | `gender` — **do not change yet** | Wait |

---

## 1. There are NO medicines in this file 🛑 ⬅ the big job

**Now (wrong):**
```xml
<MedicationDispensed>
  <MedicationItems />      <!-- EMPTY in all 16 records -->
```

Same problem as Ordered Medications. NEHR requires **1 or more** `medicationItem`. Zero is not allowed.

**✅ You can build this now.** Your Medication List service (`listmed`) already sends real medicine data for the same clinic. The clinic system **has** the data — it is just not connected to this service.

> ### ⭑ Do NOT copy the Ordered Medications item code
>
> The medicine block has the **same name** in both services but **different fields inside**:
>
> | | Ordered Medications | **This service** |
> |---|---|---|
> | date field | `medicationItemOrderedDate` | **`medicationItemDispensedDate`** |
> | quantity | `quantityOrdered` | **`quantityDispensed`** |
> | quantity unit | `quantityOrderedUnits` | **`quantityDispensedUnits`** |
> | `medicationDiscontinuedDateTime` | yes | **not here** |
> | `additionalDosageInstruction` | yes | **not here** |
>
> **One extra rule only in this service:** when `medicationItemStatus` is `Returned`, `quantityDispensed` must have a **minus sign** in front. Example: `-2`.

---

## 2. `dispensingInstitution` is missing

**Required**, and you do not send it at all.

**Add:**
```xml
<dispensingInstitution>9403258</dispensingInstitution>
```

**Why `9403258`:** NEHR's rule says — if the clinic has its own pharmacy, put the clinic here. Use the HCI code first. If there is no HCI code, use the MOH licensee number.

---

## 3. `dispensedBy` is empty

**Now (wrong):**
```xml
<DispensedBy />      <!-- empty in all 16 records -->
```

**Required.** Both the ID and the name inside are required.

**Change to:**
```xml
<dispensedBy>
  <id>M03009J</id>
  <name><value>PHILIP KOH</value></name>
</dispensedBy>
```

**Why the doctor:** NEHR's rule says — *"must be a clinician who dispenses the medications. If a clinic assistant does the dispensing, still put the licensed clinician (MCR number and name)."*

For a small clinic that is the doctor who saw the patient. **Please ask the clinic** if a different licensed person does the dispensing.

---

## 4. One `<putDispensedMedications>` per message

**Now (wrong):**
```xml
<ArrayOfPutPatientDispensedMedications>
  <PutPatientDispensedMedications> ... </PutPatientDispensedMedications>
</ArrayOfPutPatientDispensedMedications>
```

**Change to:**
```xml
<putDispensedMedications xmlns="http://www.mohh.com/nehr">
  ...
</putDispensedMedications>
```

NEHR reads one at a time. There is no box for many records.

---

## 5. ⭑ Root name has an extra word `Patient`

**Now (wrong):** `PutPatientDispensedMedications`
**Change to:** `putDispensedMedications`

There is **no** `Patient` in this name.

---

## 6. Change the namespace

**Now (wrong):** `xmlns:NEHR="http://www.synapxe.sg/nehr/putEvent"`

The address is wrong, and it says `putEvent` — a different service. Also `NEHR:` is never used on any tag.

**Change to:** `xmlns="http://www.mohh.com/nehr"` — with no `:NEHR` part.

---

## 7. ⭑ `MedicationDispensed` must be `medicationDispense`

**Now (wrong):** `<MedicationDispensed>`
**Change to:** `<medicationDispense>`

Look carefully — **`Dispense`, not `Dispensed`**. Different word, and different first letter.

---

## 8. Delete the `<MedicationItems>` box

**Now (wrong):**
```xml
<MedicationDispensed>
  <MedicationItems />
```

**Change to:**
```xml
<medicationDispense>
  ...other fields first...
  <medicationItem> ...medicine 1... </medicationItem>
  <medicationItem> ...medicine 2... </medicationItem>
</medicationDispense>
```

There is no `MedicationItems` level. Same mistake as `Problems`, `Entries` and the same box in Ordered Medications.

---

## 9. ⭑ `eventId` must be `eventID` — big D ⚠️

**Now (wrong):**
```xml
<eventId>evt-0708202610131124913034023</eventId>
```
**Change to:**
```xml
<eventID>0708202610131124913034023</eventID>
```

Two things: **big `D`** in this service, and **remove `evt-`**. The value must match the `event/id` from putEvent letter by letter, or the dispense links to no visit.

---

## 10. Small letter at the start of other tag names

**Now (wrong):** `ControlHeader` `Patient` `DispensingLocation` `DispenseType` `DispenseStatus` `AuthorizedBy` `OrderedBy` `OrderingLocation`

**Change to:** `controlHeader` `patient` `dispensingLocation` `dispenseType` `dispenseStatus` `authorizedBy` `orderedBy` `orderingLocation`

---

## 11. Delete `msgType`, `patientMergeType`, `<Id>`, empty tags

| Delete this | Why |
|---|---|
| `<msgType>Clinical</msgType>` | `msgType` exists **only in putEvent**. Not here. |
| `<patientMergeType>OBSOLETE</patientMergeType>` | Does not exist in this service at all. |
| `<Id>` inside the control header | Your internal ID. NEHR has no such field. |
| `<contactDetails/>` `<race/>` `<language/>` `<occupation/>` | Empty tags are not allowed. |
| `<DispensedBy/>` | Do not send it empty — **fill it** instead. See number 3. |

---

## 12. `<name><name>` must be `<name><value>`

```xml
<name><value>TAN AH KOW</value></name>
```

---

## 13. Put the tags in the right order ⚠️

**NEHR checks the order.** Right tags in the wrong order = rejected.

**Inside `<medicationDispense>` — correct order:**
```
recordIdentifier, eventID, orderID, sourceGroupingID,
orderingInstitution, orderingLocation, orderedBy,
dispenseID, dispensingInstitution, dispensingLocation,
dispenseType, dispenseDateTime, dispenseStatus,
dispensedBy, authorizedBy, medicationItem
```

Two things to notice:

- **All the ORDER fields come first**, then all the DISPENSE fields.
- **The medicines go LAST**, not first. Today you put `medicationItems` first.

Easiest way: copy the order straight from the example file.

---

## 14. Add `orderID` — links dispense to order

Today `orderID` is missing. So NEHR sees an order, and sees a dispense, but cannot see that they belong together.

**We found the link for you.** We compared the two files:

| | orderedmed record 1 | dispensedmed record 1 |
|---|---|---|
| MRN | `MDX-GCMS^13034023` | `MDX-GCMS^13034023` — same |
| event | `evt-0708202610131124913034023` | same |
| `sourceGroupingID` | `visit-0708202610131124913034023` | same |
| `orderID` | `0708202610131124913034023` | **missing** |

So the two records are the same visit and the same medicine order.

**Add:**
```xml
<orderID>0708202610131124913034023</orderID>
```

This is **not invented** — we copied it from your own order file. But **please confirm** the clinic system can really find the order for a dispense. If it cannot, this field must stay empty rather than be guessed.

---

## 15. Add `orderingInstitution` and `orderedBy/id`

Both are optional, but you already know the values:

```xml
<orderingInstitution>9403258</orderingInstitution>
```
Same clinic ordered and dispensed.

```xml
<orderedBy>
  <id>M03009J</id>          <!-- ADD — today only the name is sent -->
  <name><value>PHILIP KOH</value></name>
</orderedBy>
```
The MCR number is already in `authorizedBy` in the same record.

---

## 16. `msgID` — send the full ID, do not cut it

**Now (wrong):** `MDX-GCMS-eb763d6b-fc51-4c34-a`

Do not put `MDX-GCMS-` in front. Do not cut at 29 letters (NEHR allows 50).

---

## 17. Date and time format

**Now (wrong):** `2026-08-07T22:40:35.1946075+08:00` and `...35.269+08:00`
**Change to:** `2026-08-07T22:40:35+08:00`

Whole seconds only. Your code sends 7 digits in some places and 3 digits in others — both are wrong.

---

## 18. `dispenseDateTime` is the wrong time

**Now (wrong):** `22:40` — the time your program ran, not the time the medicine was given.

The real time is `10:13`. It is inside your own event id:

```
0708202610131124913034023
        └── 101311249 = 10:13:11.249
```

**Also a rule to follow:** `dispenseDateTime` must be the **same or later** than the newest `medicationItemDispensedDate` inside. We cannot check this today because there are no medicines (number 1).

---

## 19. Doctor number needs `M` in front

**Now (wrong):** `03009J` in `authorizedBy`
**Change to:** `M03009J`

---

## 20. `race` — needs real data

Required by NEHR, but sent empty. We did not put a fake value. Someone must connect `race` from patient registration.

---

## 21. `gender` — do NOT change this yet ⚠️

Your system sends `C` / `D`. NEHR's template says `M`/`F`/`U`, but NEHR's own NHDD sample says `C` = Female, `D` = Male.

**So your value may already be correct.** Please wait for the answer. Do not convert.

---

## Do NOT change these — they are already correct

- `dispenseType` code `Outpatient` on `Med_Order_Dispense_Type_(NEHR)`
  → **the codeset name is spelled correctly here.** In Ordered Medications the same name is written wrong (`Med_Order_Med_Order_Dispense_Type_(NEHR)`). So fix that service, not this one.
- `dispenseStatus` code `Completed`
- `dispensingLocation` and `orderingLocation` sending only `textDescription`
- `dispenseID` with a prefix — that is your own number, so a prefix is fine
- `recordIdentifier` starting with `dsp-rec-`, `sourceGroupingID` starting with `visit-` — also your own numbers
- `sourceGroupingID` being the same as the order record for the same visit — that is exactly what it is for

---

## One length rule to watch later

NEHR adds these fields together and the total must be **1994 letters or less**:

`eventID` + `dispenseID` + `dispensingInstitution` + `dispensingLocation` + `itemId` + `sequenceNo` + `dispenseDateTime`

Today it is about 150 letters, so it is fine. But `dispenseID` is long. **Check this again after you add the medicines.**

---

## One note about the source file

When we did this review on 9 Aug, the web address gave no data (`404 There is no data provided.`). So we used the copy saved on 7 Aug: `xml-fixes/source-xml/dispensedmed.xml` (16 records).

Please check the tag names again when the address is working.

---

## Check your work

```bash
python3 tools/check_xml.py your_file.xml
```

Run it after every change. When you see `RESULT: PASSED`, that file is good.
