# putEvent — Fixes To Make

**For the developer. Simple English.** Checked against the Excel mapping template `putEvent_v0.2_20251224.xlsx` — that file is the rule.

Fix 1 to 6 now. Then read "Wait for the codes" and "Please confirm."

---

## Fix now

**1. Add `msgType`. It is missing.**
The Excel marks it **Mandatory**. Every message must have it. For a normal visit the value is `Clinical`.
Put it in `controlHeader`, right after `msgDateTime`:
```xml
<msgType>Clinical</msgType>
```

**2. Do not send empty tags.**
The Excel Readme says: if there is no value, do not write the tag at all.
Now you send empty `<address/>`, `<phone/>`, `<race/>`, `<occupation/>`, `<language/>`, `<maritalStatus/>`.
Rule: no value → delete the whole tag. If a parent becomes empty too (`<contactDetails>`, `<nextOfKin>`), delete the parent as well.

**3. `race` must have a value.**
The Excel marks `race` **Mandatory**, but you send it empty. It cannot be blank.
It needs a real NHDD code — see "Wait for the codes" below. Do not send it empty, and do not put a fake value.

**4. Fix the movementType code name.**
Now: `<codingSchemeName>Movement_Type_(NEHR)</codingSchemeName>`
Change to: `<codingSchemeName>Visit_Type_(NEHR)</codingSchemeName>`
The Excel says: for an outpatient visit, use `Visit_Type_(NEHR)`. There is no code set called `Movement_Type_(NEHR)`.

**5. Move `movementType` to the end.**
Now it sits just after `eventType`. The Excel lists it **last** in the event block, after everything else.
Move the whole `movementType` block to the end of `<event>`, after `attendingClinician`.

**6. `type` must match the ID.**
Now every patient has `<type>SP</type>`. `SP` means Singapore NRIC.
But most IDs are passports (`X...`, `MF...`, `E...`), not NRICs. The Excel says the type must match the ID.
Set `type` from each patient's real document type. Do not hardcode `SP`.

---

## Wait for the codes — do NOT guess

`gender`, `race`, `nationality`, `language`, `marital status` must all use **NHDD codes**.
**We do not have the official NHDD code list yet** — the Excel points to it but does not contain it, and it was not on the NEHR portal. It is being requested from Synapxe.

Right now the data is mixed and unsafe to "correct" by guessing:
- `gender` uses one style (`C`/`D`), `marital status` uses another (`2`).
- `language` sends `Bahasa`, which is plain text, not a code.

**Do not change these five fields until we send you the real NHDD list.** Then map each field to it. Changing them now by guessing can corrupt every patient record.

---

## Please confirm (not visible in the file)

**A. `recordIdentifier`** — the Excel says this must stay the **same** if the visit is later changed or cancelled. Is it **saved** with the visit, or made new each time you send? It must be saved.

**B. `event > id`** — the Excel says the other data types (allergy, meds, etc.) must send this **same** event id for the same visit. Is it stored once and reused? It must be.

**C. Doctor number** — `03009J`. Singapore doctor numbers (MCR) start with `M`, e.g. `M03009J`. Please check the real number and add the `M` if it is missing. (This one is from the SMC list, not the Excel — please verify.)

---

## Smaller cleanups

- `title` — send `Mr`, not `Mr.` (no dot).
- `msgID` — send the full ID. It is being cut to 20 characters; send all of it so two messages can never clash.

---

## Important — not an XML issue

The endpoint is open on plain `http://` with **no login**, and it returns **real patient names, birth dates and ID numbers** to anyone who asks.
Please put it behind the secure channel and use fake test data. Fix this before more testing.