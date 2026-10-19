# NEHR XML — Fix List for the Developer

**Date:** 14 Oct 2026
**Files checked:** allergy, cardiology, dispensedMedication, medicationList, orderedMedication, problemList, putEvent, referral (export of 13 Oct 2026)

## How to use this list

- The **Excel mapping files** (`put..._v0.2_20251224.xlsx`, "Request" tab) are the rule book. Every fix below comes from them.
- XML tag names are **case-sensitive**. `eventId` and `eventID` are different tags.
- The **order** of tags matters. Keep the same order as the Excel.
- Tick each box when it is done.

---

## Part 1 — Fix in ALL files

**1. Time zone — use Singapore time**
- [ ] Today the times end with `+06:30`. That is Myanmar time. It must be Singapore time.
- [ ] Always build the time in Singapore time (`Asia/Singapore`). Do not use the computer's clock setting.
- [ ] `msgDateTime` → `2026-10-13T14:21:09+08:00` (with `+08:00`)
- [ ] All other date-time fields → `2026-10-13T14:21:09` (no `+08:00`, no `.1234567`)

**2. Do not send empty tags**
- [ ] If there is no value, **do not write the tag at all**. No `<tag/>`, no `<tag></tag>`.
- [ ] If a group becomes empty because of this, remove the group too.
- [ ] Put this rule in one place, so every file uses it. putEvent already does this. The other 7 files do not.

**3. Doctor number**
- [ ] `03009J` → `M03009J` (please check with the clinic that this is Dr Koh's correct MCR number)
- [ ] Change it everywhere: attendingClinician, recorder, creator, author, orderedBy, authorizedBy, dispensedBy, lastUpdatedBy, createdBy, updatedBy, requester, primaryOperator.

**4. msgID**
- [ ] Today it is cut to 20 characters (`c463a98b-256b-4339-8`). Send the full ID (36 characters). The field allows 50.

**5. Title**
- [ ] `Mr.` → `Mr` (no full stop). Same for `Mrs.`, `Mdm.`.

**6. Patient details — do not use fixed default values**
All 3 test patients have the same race, marital status, nationality, language and occupation. One is a 9-year-old child marked "married". These look hardcoded. Read the real values from GCMS. Use the NHDD codes (not the "Sample Value" in the Excel):
- [ ] `gender` — `C` / `D` are correct. **Do not change.**
- [ ] `maritalStatus` — `2` is wrong. Married = `7`, Single = `6`, Widowed = `8`, Divorced = `9`.
- [ ] `language` — `EL` is wrong. English = `DA`. Indonesian = `DG`. Malay = `DJ`. Mandarin = `DI`. Do not send free text like `Bahasa`.
- [ ] `nationality` — Indonesian = `EM`, Malaysian = `ER`, Chinese = `EI`. For Singapore, wait — we are confirming the code.
- [ ] `race` — must be filled (Mandatory). Chinese = `HE`, Malay = `HL`, Indian = `HI`, Eurasian = `HG`, Indonesian = `HH`.
- [ ] If the value is blank in GCMS, leave the tag out (see fix 2).

**7. Patient ID type**
- [ ] Today every patient is sent as `SP` (pink IC), even passport holders.
- [ ] `SP` only for Singapore citizen pink IC. `SB` for PR blue IC. `SO` for FIN.
- [ ] Passport holders: we are waiting for NEHR's answer. We will tell you the code.

**8. Element order**
- [ ] Put tags in the same order as the Excel. The exact orders you need are given in each file section below.

---

## Part 2 — Fixes per file

### putEvent.xml

- [ ] Add `<msgType>Clinical</msgType>` in `controlHeader`, right after `msgDateTime`. It is Mandatory and still missing.
- [ ] Move `movementType` to the **end** of `<event>`. Order:
  `recordIdentifier, id, movementSequenceNo, eventStartDate, cancellationNotice, institution, serviceSpecialty, patientType, eventType, attendingClinician, movementType`

---

### allergy.xml (putAllergyADR)

**Fix the tag names:**
- [ ] `creater` → `creator`
- [ ] `practitoner` → `practitioner` (inside `creator` and inside `recorder`)
- [ ] `onSet` → `onset`
- [ ] `aDRAlertIndicatorCode` → `ADRAlertIndicatorCode`
- [ ] `aDROutcome` → `ADROutcome`
- [ ] `allergySeverity` → `severity`
- [ ] `allergyInformationSource` → `informationSource`

**Fix the content:**
- [ ] Delete `<profession>` inside `creator > practitioner`. Creator only has `identifier` and `name`.
- [ ] Add `<adverseReaction>` (Mandatory). It is the reaction text, for example `Rash, Itching`. Use the CMIS adverse reaction list as the pick-list.
- [ ] Information source: today `01` + `Medical records`. That is wrong. `01` means "Reporter observation". Change to `02` + `Medical records`. CMIS checks the code and the text together.
- [ ] Typo in system name: `CMIS_ADR_ALERT_INDICATOR_(NEHR` → `CMIS_ADR_ALERT_INDICATOR_(NEHR)` (closing bracket missing).
- [ ] Add a `<text>` tag next to each `<coding>`, with the same words as `display`. This is needed for: `verificationStatus`, `informationSource`, `recorder > profession > code`, `ADRAlertIndicatorCode`, `severity`, `ADROutcome`, `exposureRoute`, `drugProbability`.
- [ ] Delete `<manifestation>` and `<productName>` when they have no data. Today they are sent with an empty `<coding/>`.
- [ ] `encounter > serviceProvider` must be the **clinic**, not the doctor:
  ```xml
  <serviceProvider>
    <identifier>9403258</identifier>
    <name>The Heart Clinic</name>
  </serviceProvider>
  ```
- [ ] "Red Meat" is a food. This interface is for **drug** allergies only. Do not send food allergies.
- [ ] If a drug is not in the CMIS drug list, do not send `<coding>`. Send the name in `<substance><text>...</text></substance>` instead.
- [ ] `allergyIntolerance > identifier`: today it is the same ID as the visit. Use the allergy record's own ID. If not, two allergies in one visit will overwrite each other.

---

### problemList.xml (Visit Diagnosis)

- [ ] `ListType` → `listType` (small "l"). Still not fixed from last time.
- [ ] `createdDateTime` must be the time the diagnosis was **created** in GCMS. Today it is the time the file was made.
- [ ] `updatedDateTime` must be the **same or later** than `createdDateTime`. Today it is earlier in all 3 messages (one is 2025).
- [ ] `lastUpdatedDateTime` (list level) must be the **same or later** than the latest `updatedDateTime` of all problems.
- [ ] `sequenceNo` — number the problems 1, 2, 3 in each list (today: 3, 6, 10 on lists with one problem).
- [ ] Check the diagnosis is a real **diagnosis**. "Abbreviated Mental Test" is a test. "Administration of prophylactic antibiotic…" is a treatment. These are not diagnoses.

---

### medicationList.xml (putPatientMedicationList)

**Missing Mandatory tags (still missing from last time):**
- [ ] Add `<identifier>` as the first tag in `<list>`. One fixed ID for this medication list.
- [ ] Add `<CMISAvailability>0</CMISAvailability>` after `<date>`.

**Fix the tag names:**
- [ ] `practitoner` → `practitioner`
- [ ] `practitonerRole` → `practitionerRole`
- [ ] `medicationStatement` → `MedicationStatement` (capital M, this is how the Excel spells it)

**Fix the content:**
- [ ] `list > code`: today `PML` / `Patient Medication List`. Change to:
  ```xml
  <code>
    <code>1</code>
    <codingSchemeName>Med_List_Type_(NEHR)</codingSchemeName>
    <textDescription>Patient's Medication List</textDescription>
  </code>
  ```
- [ ] **Delete the whole `<supportingInformation>` block.** Today it is filled with Problem List codes (`Non-reconciled PPL`), which is wrong. The rule says: if you do not have the data, leave it out.
- [ ] **Delete the whole `<notesforHCP>` block.** Today it uses the diagnosis status code "Active", which is wrong.
- [ ] Delete empty `<medicalAlert/>`, `<drugAllergy/>`, `<strengthUnit/>`, `<reasonForUseCodeableConcept/>`.
- [ ] Dose unit: today `Med_UnitofMeasure_(NEHR)` + `Tab`. Change to the PML dose unit list:
  ```xml
  <unit>
    <code>428673006</code>
    <codingSchemeName>Med_List_Item_Dose_Unit_(NEHR)</codingSchemeName>
    <textDescription>tablet</textDescription>
  </unit>
  ```
- [ ] `entry > date`: today only a date (`2026-10-13`). Send date and time: `2026-10-13T14:05:34`.
- [ ] `dateAsserted`: remove the `.3047724` part.
- [ ] Drug codes are **gone**. Last time 8 of 11 drugs had an SDD code. Now 0 of 9. Please put them back (see "Medicine codes" below).

**Order of tags in `<list>`:**
`identifier, code, source, encounter, status, date, CMISAvailability, sourceOfMedicationList, reviewedUpon, entry`

**Order of tags in `<MedicationStatement>`:**
`identifier, sequenceNo, informationSource, dateAsserted, status, medicationReference, dosage`

**Order of tags in `<dosage>`:**
`timing, route, quantityRange`

---

### orderedMedication.xml (putOrderedMedications)

**Biggest problem — duplicate messages:**
- [ ] The file has 10 messages but only 3 are different. Each order is copied once for every medicine in it (4 medicines → 4 copies). The copies have the same msgID, so NEHR will reject them. Send **one message per order**. This looks like a JOIN in the query that repeats rows.

**Fix the tag names:**
- [ ] `medicationOrdered` → `medicationOrder`
- [ ] `MedicationItem` → `medicationItem` (small m)
- [ ] `eventId` → `eventID`
- [ ] `orderBy` → `orderedBy`

**Fix the content:**
- [ ] Typo: `Med_Order_Med_Order_Dispense_Type_(NEHR)` → `Med_Order_Dispense_Type_(NEHR)`
- [ ] `lowUnit` is plain text, not a group:
  ```xml
  <!-- now -->
  <lowUnit><codingSchemeName>Med_UnitofMeasure_(NEHR)</codingSchemeName><textDescription>Tab</textDescription></lowUnit>
  <!-- change to -->
  <lowUnit>Tab</lowUnit>
  ```
- [ ] `orderDateTime` must be the **same or later** than the latest `medicationItemOrderedDate`. Today it is earlier in all 3.
- [ ] Delete empty `<dosageInstruction/>` and `<reasonForMedication/>`.
- [ ] `durationUOM` text `Month()` looks like a bug. Check the brackets.
- [ ] `orderingLocation` — send the clinic name and address, not "Consultation Room 1".
- [ ] Delete `<authorizedBy>` when it is the same doctor as `orderedBy`. It is only for a junior doctor's order.
- [ ] `sourceGroupingID` — today `visit-<eventId>`. That is the **same** as in the dispensed file. It must be different. Example: `ORD^<orderID>` for orders and `DSP^<dispenseID>` for dispenses.
- [ ] Medicine codes are missing — see "Medicine codes" below.

**Order of tags in `<medicationOrder>`** (the medicine list goes **last**):
`recordIdentifier, eventID, orderID, sourceGroupingID, orderingInstitution, orderingLocation, orderType, orderDateTime, orderStatus, orderedBy, medicationItem`

**Order of tags in `<medicationItem>`:**
`itemId, sequenceNo, dosageInstruction, doseQuantity, medicationItemOrderedDate, medicationItemStatus, reasonForMedication, duration, durationUOM, medicationName, frequency, routeOfAdministration, doseForm`

---

### dispensedMedication.xml (putDispensedMedications)

**Biggest problem — duplicate messages:**
- [ ] 11 messages, only 3 different. Same JOIN problem as the ordered file. Send **one message per dispense**.

**Fix the tag names:**
- [ ] `medicationDispensed` → `medicationDispense`
- [ ] `MedicationItem` → `medicationItem`
- [ ] `eventId` → `eventID`

**Fix the content:**
- [ ] Add `<dispensingInstitution>9403258</dispensingInstitution>` (Mandatory, missing).
- [ ] `<dispensedBy/>` is empty. It must be the doctor, even if a clinic assistant gave the medicine:
  ```xml
  <dispensedBy>
    <id>M03009J</id>
    <name>PHILIP KOH</name>
  </dispensedBy>
  ```
- [ ] `dispenseDateTime` must be the **same or later** than the latest `medicationItemDispensedDate`. Today it is earlier in all 3.
- [ ] `lowUnit` → plain text (same as ordered).
- [ ] Delete empty `<dosageInstruction/>` and `<reasonForMedication/>`.
- [ ] `sourceGroupingID` must be different from the ordered file (see ordered).
- [ ] `dispensingLocation` — send the clinic address. Use a normal hyphen `-`, not the long dash `—`.
- [ ] Medicine codes are missing — see "Medicine codes" below.

**Order of tags in `<medicationDispense>`** (the medicine list goes **last**):
`recordIdentifier, eventID, orderID, sourceGroupingID, orderingInstitution, orderingLocation, orderedBy, dispenseID, dispensingInstitution, dispensingLocation, dispenseType, dispenseDateTime, dispenseStatus, dispensedBy, authorizedBy, medicationItem`

**Order of tags in `<medicationItem>`:**
`itemId, sequenceNo, dosageInstruction, doseQuantity, medicationItemDispensedDate, medicationItemStatus, reasonForMedication, duration, durationUOM, medicationName, frequency, routeOfAdministration, doseForm`

---

### Medicine codes (ordered, dispensed, medication list)

Today only the name of the code list and the text are sent, with **no code**. The rule: only send `codingSchemeName` if you also send `code`.

- [ ] **Drug name** — add the SDD code. I will send you our GCMS-to-SDD mapping file (116 drugs already matched).
  If a drug is not in SDD:
  ```xml
  <medicationName>
    <code>NONHSAMANAGED</code>
    <codingSchemeName>Non_Singapore_Drug_Dictionary_(NEHR)</codingSchemeName>
    <textDescription>Drug name here</textDescription>
  </medicationName>
  ```
- [ ] **Frequency, route, dose form, duration unit** — add the SMST code. I will send you the SMST file.
  If there is no matching code:
  ```xml
  <code>NoSMST</code>
  <codingSchemeName>NON_SMST_(NEHR)</codingSchemeName>
  <textDescription>Every morning</textDescription>
  ```

---

### cardiology.xml (putCardiology)

Good progress — most fixes from last time are done.

- [ ] `race` is empty in all 9. Fill it (see Part 1, fix 6).
- [ ] Delete empty `<occupation/>`, `<language/>`, `<maritalStatus/>`.
- [ ] `cardiologyReport > type`: add the missing description:
  ```xml
  <type>
    <code>2</code>
    <codingSchemeName>Cardiology_Procedure_Category_(NEHR)</codingSchemeName>
    <textDescription>Non Invasive</textDescription>
  </type>
  ```
- [ ] Remove `+08:00` from `lastUpdatedTime`, `startDateTime`, `reportDateTime`.
- [ ] `language` `Bahasa` → NHDD code (see Part 1, fix 6).
- [ ] **This file has real patients** (real names and NRICs). Please use test patients only. See the question at the end.

---

### referral.xml (putComposition — Referral)

**Fix the tag names:**
- [ ] `composition > creator > practitoner` → `practitioner`
- [ ] `composition > author > practitoner` → `practitioner`
  (Only inside `referralRequest > requester` and `recipient` is the spelling `practitoner` correct. Leave those.)
- [ ] `recipient > organization` → `organisation` (still not fixed from last time)
- [ ] `langauge` → `language` (still not fixed from last time)

**Delete these:**
- [ ] Empty `<system/>` inside `creator`.
- [ ] The `<type>` block inside `requester > organisation` (still there from last time). Keep the `<type>` inside `recipient > organisation`.
- [ ] `<period>` inside `referralRequest > encounter`. Only `identifier` is allowed there.
- [ ] Empty `<reason/>`. Or fill it: `<reason><textDescription>reason for referral, from GCMS</textDescription></reason>`.
- [ ] In "Patient Information", 2 `<other>` blocks per message have no `<extension>`. Delete them, or fill them.
- [ ] The "Referral Information" section has an empty `<content/>`. Delete the section, or add items.

**Fix the attachment:**
- [ ] `<other><code>` for the attachment: `1` → `2` (`2` = File Attachment, `1` = Notes Item).
- [ ] Add `<textDescription>PDF</textDescription>` in `contentType`.
- [ ] `<file/>` is empty in 2 of 3 messages. If there is no file, do not send the attachment section at all.

**Other:**
- [ ] `composition > identifier`: today it is the visit ID. Use the referral's own ID. If not, two referrals in one visit will overwrite each other.
- [ ] Keep the "NRIC" line in Patient Information. The Excel allows it here.

---

## Part 3 — Please answer these questions

1. **Medicines per patient.** For the same visit, the ordered file, the dispensed file and the medication list each show **different** medicines. The item IDs also go up in order across all patients. Are you picking the patient's own medicines, or the first rows of the table?
2. **msgSequenceID.** Every file starts at 1. Where does this number come from? It must keep going up across runs (day 1: 1–20, day 2: 21–40…). If it starts from 1 every day, NEHR may keep the old version of a record.
3. **cardiology.xml.** Where does this file come from? It has real patient data. Is the GitHub repo `xml-export` private?
4. **Batch file.** Each file holds many messages in one wrapper (`<PutEvent>…`). Please confirm the Adapter sends **one SOAP call per message** to NEHR.
5. **Time zone.** Which computer made this export? It is set to Myanmar time.

---

## Do NOT change these — they are correct

- `srcInstitution` = `9403258`, `licenseeNumber` = `L/9403258/MDS/007/242`, `srcApplication` = `MDX-GCMS`
- `MRNNumber` format `MDX-GCMS^13023442`
- `gender` `C` / `D`
- `serviceSpecialty` `NEHR_SS_0002` Cardiology
- The event ID is the same in all files for each visit — good.
- Allergy codes (other than information source): reporting flag `A`, indicator `Y`, profession `2`, severity `N`, outcome `1`, probability `1`, route `038`.
- Status codes: `New`, `Completed`, `Final`, `Active`, `Outpatient`.
- Cardiology attachment codes and PDFs.
- Referral codes: `ReferralNote`, section `4`, status `1`, type `4`, organisation type `2`.
