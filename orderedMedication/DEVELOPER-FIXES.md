# Ordered Medications (putOrderedMedications) — What To Fix

**For the developer.** Simple English. Fix number 1 first, then 2, then 3.

| | |
|---|---|
| Correct example file | `putOrderedMedications-corrected.xml` — in this folder. Copy the shape from it. |
| More detail (harder English) | `FIXES.md` — in this folder. |
| Check your file | `python3 tools/check_xml.py your_file.xml` |

⚠️ **Do putEvent first.** This service needs the event id that putEvent makes.

🛑 **The biggest problem: there are NO medicines in this file.** See number 1. Everything else is small compared to this.

---

## The list

| # | What to fix | How hard |
|---|---|---|
| 1 | **No medicines at all — add them** 🛑 | Big job |
| 2 | One `<putOrderedMedications>` per message | Easy |
| 3 | Root name has an extra word `Patient` | Easy |
| 4 | Change the namespace | Easy |
| 5 | `MedicationOrdered` must be `medicationOrder` | Easy |
| 6 | Delete the `<MedicationItems>` box | Easy |
| 7 | ⭑ `OrderBy` must be `orderedBy` | Easy |
| 8 | ⭑ `eventId` must be `eventID` — **big D** | Easy |
| 9 | Small letter at the start of other tag names | Easy |
| 10 | Delete `msgType`, `patientMergeType`, `<Id>`, empty tags | Easy |
| 11 | `<name><name>` must be `<name><value>` | Easy |
| 12 | Put the tags in the right order | Medium |
| 13 | One codeset name has a doubled word | Easy |
| 14 | `msgID` — send the full ID, do not cut it | Easy |
| 15 | Date and time format | Easy |
| 16 | `orderDateTime` is the wrong time | Medium |
| 17 | `orderID` is the same as `eventID` | Need clinic |
| 18 | Doctor number needs `M` in front | Easy |
| 19 | `race` is empty | Need data |
| 20 | `gender` — **do not change yet** | Wait |

---

## 1. There are NO medicines in this file 🛑 ⬅ the big job

**Now (wrong):**
```xml
<MedicationOrdered>
  <MedicationItems />      <!-- EMPTY in all 16 records -->
```

**This service exists to send the medicines.** Today it sends only the envelope: who ordered, where, when, for which patient. No medicine.

NEHR requires **1 or more** `medicationItem`. Zero is not allowed.

### Each medicine needs these 10 fields

`itemId` · `sequenceNo` · `dosageInstruction` · `doseQuantity` (both `lowValue` and `lowUnit`) · `medicationItemOrderedDate` · `medicationItemStatus` · `durationUOM` · `medicationName` · `frequency` · `routeOfAdministration`

### ✅ Good news — you can build this now

**Your Medication List service (`listmed`) already sends real medicine data** for the same clinic: drug code, frequency, route, dose form. So the clinic system **has** the data. It is just not connected to this service.

**So this is connecting work, not new work.** Take the same data, and use the field names from this service's template.

### What we put in the example file

We wrote a **skeleton** — the correct shape with `TODO_` in every place where your data must go. We did **not** invent medicines.

- Every value marked `TODO_` must come from the clinic system.
- The codeset names and the examples inside the comments **are correct** — you can use them as they are.
- Some numbers and dates have fake shapes (not `TODO_`), because NEHR rejects the word `TODO_` in a number field.

---

## 2. One `<putOrderedMedications>` per message

**Now (wrong):**
```xml
<ArrayOfPutPatientOrderedMedications>
  <PutPatientOrderedMedications> ...order 1... </PutPatientOrderedMedications>
</ArrayOfPutPatientOrderedMedications>
```

**Change to:**
```xml
<putOrderedMedications xmlns="http://www.mohh.com/nehr">
  ...order 1...
</putOrderedMedications>
```

NEHR reads one order at a time. There is no box for many orders.

---

## 3. ⭑ Root name has an extra word `Patient`

**Now (wrong):** `PutPatientOrderedMedications`
**Change to:** `putOrderedMedications`

There is **no** `Patient` in this name. Someone added it.

---

## 4. Change the namespace

**Now (wrong):** `xmlns:NEHR="http://www.synapxe.sg/nehr/putEvent"`

Two problems: the address is wrong, **and** it says `putEvent` — a different service. Also `xmlns:NEHR=` is declared but `NEHR:` is never used on any tag.

**Change to:** `xmlns="http://www.mohh.com/nehr"` — with no `:NEHR` part.

---

## 5. ⭑ `MedicationOrdered` must be `medicationOrder`

**Now (wrong):** `<MedicationOrdered>`
**Change to:** `<medicationOrder>`

Look carefully — **`Order`, not `Ordered`**. Different word, and different first letter.

---

## 6. Delete the `<MedicationItems>` box

**Now (wrong):**
```xml
<MedicationOrdered>
  <MedicationItems>
    <MedicationItem> ... </MedicationItem>
  </MedicationItems>
</MedicationOrdered>
```

**Change to:**
```xml
<medicationOrder>
  ...other fields first...
  <medicationItem> ...medicine 1... </medicationItem>
  <medicationItem> ...medicine 2... </medicationItem>
</medicationOrder>
```

There is no `MedicationItems` level in NEHR. Same mistake as `Problems` (Visit Diagnosis) and `Entries` (Medication List).

---

## 7. ⭑ `OrderBy` must be `orderedBy`

**Now (wrong):** `<OrderBy>`
**Change to:** `<orderedBy>`

This is a **different word**, not only a different first letter. `Order` + `ed` + `By`.

---

## 8. ⭑ `eventId` must be `eventID` — big D ⚠️

**Now (wrong):**
```xml
<eventId>evt-0708202610131124913034023</eventId>
```
**Change to:**
```xml
<eventID>0708202610131124913034023</eventID>
```

**Two things to fix here:**

1. **Big `D`.** In this service the tag is `eventID`. 9 of the 15 services use `eventID` with a big `D`. **Only `putPatientProblemList` uses `eventId` with a small `d`.** Your code uses the small `d` version everywhere — wrong here.
2. **Remove `evt-`.** The value must match the `event/id` from putEvent **letter by letter**. With `evt-` in front, the order links to no visit.

---

## 9. Small letter at the start of other tag names

**Now (wrong):** `ControlHeader` `Patient` `OrderingLocation` `OrderType` `OrderStatus` `AuthorizedBy`

**Change to:** `controlHeader` `patient` `orderingLocation` `orderType` `orderStatus` `authorizedBy`

---

## 10. Delete `msgType`, `patientMergeType`, `<Id>`, empty tags

| Delete this | Why |
|---|---|
| `<msgType>Clinical</msgType>` | `msgType` exists **only in putEvent**. Not here. |
| `<patientMergeType>OBSOLETE</patientMergeType>` | Does not exist in this service at all. |
| `<Id>` inside the control header | Your internal ID. NEHR has no such field. |
| `<contactDetails/>` `<race/>` `<language/>` `<occupation/>` | Empty tags are not allowed. No value → do not write the tag. |

---

## 11. `<name><name>` must be `<name><value>`

```xml
<name><value>TAN AH KOW</value></name>
```

---

## 12. Put the tags in the right order ⚠️

**NEHR checks the order.** Right tags in the wrong order = rejected.

**Inside `<medicationOrder>` — correct order:**
```
recordIdentifier, eventID, orderID, sourceGroupingID,
orderingInstitution, orderingLocation, orderType, orderDateTime,
orderStatus, orderedBy, authorizedBy, medicationItem
```

Two things to notice:

- **The medicines go LAST**, not first. Today you put `medicationItems` first.
- `orderDateTime` comes **after** `orderType`, not before.

Easiest way: copy the order straight from the example file.

---

## 13. ⭑ One codeset name has a doubled word

**Now (wrong):**
```xml
<codingSchemeName>Med_Order_Med_Order_Dispense_Type_(NEHR)</codingSchemeName>
```
**Change to:**
```xml
<codingSchemeName>Med_Order_Dispense_Type_(NEHR)</codingSchemeName>
```

Look: `Med_Order_` appears **two times**. This looks like a string-joining bug in your code — something adds the prefix twice.

**Note:** the Dispensed Medications service writes this name **correctly**. So the bug is only in this service's code path. Please find where the two paths differ.

---

## 14. `msgID` — send the full ID, do not cut it

**Now (wrong):** `MDX-GCMS-eb763d6b-fc51-4c34-a`

Do not put `MDX-GCMS-` in front. Do not cut at 29 letters (NEHR allows 50).

Note: the end of the real ID cannot be recovered from your file, because it was already cut before saving.

---

## 15. Date and time format

**Now (wrong):** `2026-08-07T22:40:35.1946075+08:00`
**Change to:** `2026-08-07T22:40:35+08:00`

Whole seconds only.

---

## 16. `orderDateTime` is the wrong time

**Now (wrong):** `22:40` — this is the time your program ran, not the time the doctor ordered.

The real time is `10:13`. You can see it inside your own event id:

```
0708202610131124913034023
        └── 101311249 = 10:13:11.249
```

**Also a rule to follow:** `orderDateTime` must be the **same or later** than the newest `medicationItemOrderedDate` inside the order. We cannot check this today because there are no medicines (number 1).

---

## 17. `orderID` is the same as `eventID`

Both fields carry `0708202610131124913034023`.

An order number should be different from a visit number. One visit can have many orders. NEHR's example is `OM^232324`.

We left your real value in the example file. **Please ask the clinic** whether the system can give a real order number.

---

## 18. Doctor number needs `M` in front

**Now (wrong):** `03009J` in `orderedBy` and `authorizedBy`
**Change to:** `M03009J`

---

## 19. `race` — needs real data

Required by NEHR, but sent empty. We did not put a fake value. Someone must connect `race` from patient registration.

---

## 20. `gender` — do NOT change this yet ⚠️

Your system sends `C` / `D`. NEHR's template says `M`/`F`/`U`, but NEHR's own NHDD sample says `C` = Female, `D` = Male.

**So your value may already be correct.** Please wait for the answer. Do not convert.

---

## Do NOT change these — they are already correct

- `orderStatus` code `New`
- `orderType` code `Outpatient`
- `orderingInstitution` = HCI code
- `orderingLocation` sending only `textDescription` (the code is optional here, and the codeset is empty)
- `recordIdentifier` starting with `ord-rec-` and `sourceGroupingID` starting with `visit-`
  → these are **your own** numbers, so a prefix is fine. **Only `eventID` must match putEvent exactly.**
- `sourceGroupingID` being the same for all items of one visit — that is exactly what it is for

---

## One note about the source file

When we did this review on 9 Aug, the web address gave no data (`404 There is no data provided.`). So we used the copy saved on 7 Aug: `xml-fixes/source-xml/orderedmed.xml` (16 records).

Please check the tag names again when the address is working, in case something changed.

---

## Check your work

```bash
python3 tools/check_xml.py your_file.xml
```

Run it after every change. When you see `RESULT: PASSED`, that file is good.
