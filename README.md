# 1. One-Page Test Plan

## 1.1 Scope

### In Scope
The testing scope covers the functional verification of bank transfers based strictly on the provided requirements:
* Validation of transfer amounts (100 KZT to 500,000 KZT, whole tenge requirement)
* Daily transfer limit validation (1,000,000 KZT threshold and priority logic)
* SMS OTP code logic (threshold of 100,000 KZT, 6-digit verification, 120-second timeout, 3-attempt cancellation limit)
* Error message prioritization when multiple rules are violated simultaneously

### Out of Scope
The following areas are explicitly excluded from this test cycle due to specification limits and test case constraints:
* **Insufficient balance handling** (behavior is undefined in the provided brief)
* **Midnight limit reset verification** (time-based execution is not covered within the 12 planned test cases)
* **Non-functional testing** (Performance, Security, Usability, Load testing)
* **UI/UX design and field-level localization**
* **Integration with core banking backends and SMS gateway infrastructure**
  
## 1.2 Test Approach
Black-box testing techniques from Week 3 will be used:
* Equivalence Partitioning (EP)
* Boundary Value Analysis (BVA)
* Decision Tables
* State Transition Testing

The test cases will focus on valid and invalid transfer amounts, boundary values, daily-limit decisions, SMS verification, and SMS state transitions.

## 1.3 Entry Criteria
Testing can start when:
1. The transfer functionality is available for testing;
2. The required test data is available;
3. Acceptance criteria for the defined transfer requirements (REQ-3.1, REQ-3.2, REQ-3.3) are baselined *(pending analyst confirmation on decimal input handling and insufficient balance)*;
4. The test environment is accessible.

## 1.4 Exit Criteria
Testing can be completed when:
1. All 12 planned test cases have been executed;
2. Critical and high-severity defects identified during testing are addressed or formally accepted;
3. The main requirements have test coverage documented in the traceability matrix;
4. Regression testing has been completed for affected functionality;
5. Remaining risks and requirement gaps are documented.

## 1.5 Top Product Risks
| Risk | Impact |
| :--- | :--- |
| **Incorrect handling of amount boundaries** | Invalid or prohibited transfers may be accepted |
| **Incorrect calculation of the daily limit** | Users may exceed the allowed daily transfer amount |
| **Incorrect SMS state handling** | A cancelled or expired transfer may be completed |

---

# 2. Test Cases (Exactly 12 Test Cases)

### TR-AMT-001 — Minimum valid transfer amount
* **ID:** `TR-AMT-001`
* **Requirement:** REQ-3.1
* **Preconditions:** User is logged in and can make a transfer.
* **Test data:** Amount = `100 KZT`
* **Steps:**
  1. Enter 100 KZT as the transfer amount.
  2. Continue the transfer.
* **Expected result:** The amount is accepted as valid and the transfer can proceed without requesting an SMS code.

---

### TR-AMT-002 — Amount below minimum
* **ID:** `TR-AMT-002`
* **Requirement:** REQ-3.1
* **Preconditions:** User is logged in and can make a transfer.
* **Test data:** Amount = `99 KZT`
* **Steps:**
  1. Enter 99 KZT.
  2. Continue the transfer.
* **Expected result:** The amount is rejected and an amount validation error is displayed.

---

### TR-AMT-003 — Maximum valid transfer amount
* **ID:** `TR-AMT-003`
* **Requirement:** REQ-3.1
* **Preconditions:** User is logged in and can make a transfer.
* **Test data:** Amount = `500,000 KZT`
* **Steps:**
  1. Enter 500,000 KZT.
  2. Continue the transfer.
* **Expected result:** The amount is accepted as valid. Because the amount is above 100,000 KZT, SMS verification is required.

---

### TR-AMT-004 — Amount above maximum
* **ID:** `TR-AMT-004`
* **Requirement:** REQ-3.1
* **Preconditions:** User is logged in and can make a transfer.
* **Test data:** Amount = `500,001 KZT`
* **Steps:**
  1. Enter 500,001 KZT.
  2. Continue the transfer.
* **Expected result:** The amount is rejected and an amount validation error is displayed.

---

### TR-AMT-005 — Decimal amount input (Pending Specification)
* **ID:** `TR-AMT-005`
* **Requirement:** REQ-3.1
* **Preconditions:** User is logged in and can make a transfer.
* **Test data:** Amount = `100.50 KZT`
* **Steps:**
  1. Enter 100.50 KZT.
  2. Continue the transfer.
* **Expected result:** **Pending Analyst Clarification.** The brief specifies "whole tenge only" but does not define the system behavior for decimal entries (e.g., input field masking, rounding, truncation, or explicit error validation). Test outcome will be evaluated once system behavior is specified.

---

### TR-APP-001 — Transfer at the SMS threshold
* **ID:** `TR-APP-001`
* **Requirement:** REQ-3.3
* **Preconditions:** User is logged in; daily transferred amount is within the limit.
* **Test data:** Amount = `100,000 KZT`
* **Steps:**
  1. Enter 100,000 KZT.
  2. Continue the transfer.
* **Expected result:** The amount is accepted and SMS verification is not required because the amount is not above 100,000 KZT.

---

### TR-APP-002 — Amount above SMS threshold (Happy Path)
* **ID:** `TR-APP-002`
* **Requirement:** REQ-3.3
* **Preconditions:** User is logged in; daily transferred amount is within the limit.
* **Test data:** Amount = `100,001 KZT`
* **Steps:**
  1. Enter 100,001 KZT.
  2. Continue the transfer.
  3. Observe the verification step.
  4. Enter a valid 6-digit SMS code before 120 seconds expire.
* **Expected result:** SMS verification is required. The valid code is accepted, and the transfer is confirmed and executed.

---

### TR-APP-003 — Daily limit exactly reached
* **ID:** `TR-APP-003`
* **Requirement:** REQ-3.2
* **Preconditions:** User is logged in; 900,000 KZT has already been transferred today.
* **Test data:** New transfer amount = `100,000 KZT`
* **Steps:**
  1. Enter 100,000 KZT.
  2. Continue the transfer.
* **Expected result:** The resulting daily total is exactly 1,000,000 KZT. The transfer is allowed because the daily limit is 1,000,000 KZT.

---

### TR-APP-004 — Daily limit exceeded
* **ID:** `TR-APP-004`
* **Requirement:** REQ-3.2
* **Preconditions:** User is logged in; 900,000 KZT has already been transferred today.
* **Test data:** New transfer amount = `150,000 KZT`
* **Steps:**
  1. Enter 150,000 KZT.
  2. Continue the transfer.
* **Expected result:** The transfer is rejected because the resulting daily total would exceed 1,000,000 KZT. A daily-limit validation error is displayed.

---

### TR-APP-005 — Amount error has priority
* **ID:** `TR-APP-005`
* **Requirement:** REQ-3.1, REQ-3.2
* **Preconditions:** User is logged in; daily transferred amount is already 1,000,000 KZT.
* **Test data:** Amount = `500,001 KZT`
* **Steps:**
  1. Enter 500,001 KZT.
  2. Continue the transfer.
* **Expected result:** The transfer is rejected with an amount validation error. The amount error is shown because the requirements specify that the amount error has priority when both conditions are violated.

---

### TR-OTP-001 — Three incorrect SMS codes
* **ID:** `TR-OTP-001`
* **Requirement:** REQ-3.3
* **Preconditions:** A transfer above 100,000 KZT is waiting for SMS verification.
* **Test data:** Three incorrect 6-digit codes
* **Steps:**
  1. Enter an incorrect SMS code.
  2. Repeat with a second incorrect code.
  3. Enter a third incorrect code.
* **Expected result:** After the third incorrect code, the transfer is cancelled. The transfer cannot be completed using that cancelled verification flow.

---

### TR-OTP-002 — Expired SMS code
* **ID:** `TR-OTP-002`
* **Requirement:** REQ-3.3
* **Preconditions:** A transfer above 100,000 KZT is waiting for SMS verification.
* **Test data:** Valid 6-digit code entered after more than 120 seconds
* **Steps:**
  1. Start a transfer requiring SMS verification.
  2. Wait until more than 120 seconds have passed.
  3. Enter the valid SMS code.
* **Expected result:** The expired code is rejected and the transfer is not completed.

---

# 3. Traceability Matrix

| Requirement | Requirement description | Covered by | Status |
| :--- | :--- | :--- | :--- |
| **REQ-3.1** | Transfer amount must be 100–500,000 KZT and use whole tenge | TR-AMT-001, TR-AMT-002, TR-AMT-003, TR-AMT-004, TR-APP-005 | **Partially Covered** -decimal handling behavior is undefined |
| **REQ-3.2** | Daily transfer limit is 1,000,000 KZT and resets at midnight Almaty time | TR-APP-003, TR-APP-004, TR-APP-005 | **Partially Covered** - midnight reset not tested |
| **REQ-3.3** | Transfers above 100,000 KZT require a 6-digit SMS code; the code is valid for 120 seconds; three wrong attempts cancel the transfer | TR-AMT-003, TR-APP-001, TR-APP-002, TR-OTP-001, TR-OTP-002 | **Covered** |
| **REQ-3.4** | Insufficient balance handling | None | **UNCOVERED** - behavior is not specified in the provided brief |

### Coverage gaps & open questions
* **REQ-3.1 - Decimal input behavior:** The brief states "whole tenge only", but does not specify how non-integer values (e.g., `100.50 KZT` in `TR-AMT-005`) should be handled (input mask rejection, rounding, truncation, or validation message).
* **REQ-3.2 - Midnight reset:** The daily transfer limit is specified to reset at midnight Almaty time, but this time-based behavior is not directly covered by the 12 planned test cases.
* **REQ-3.4 - Insufficient balance:** The provided brief does not define insufficient balance as a test condition and does not specify the expected system behavior when the account balance is insufficient. Additional analyst clarification is required.

---

# 4. Release Checklist

1. [ ] Transfer amounts from 100 to 500,000 KZT are accepted.
2. [ ] Amounts below 100 KZT are rejected.
3. [ ] Amounts above 500,000 KZT are rejected.
4. [ ] Decimal entry behavior is verified once business rules/specifications are clarified (whole tenge requirement).
5. [ ] Transfers of 100,000 KZT or less do not require SMS verification.
6. [ ] Transfers above 100,000 KZT require a 6-digit SMS code.
7. [ ] A correct SMS code entered within 120 seconds allows the transfer to be confirmed.
8. [ ] An SMS code is rejected after 120 seconds.
9. [ ] The third incorrect SMS code cancels the transfer.
10. [ ] The daily transfer total cannot exceed 1,000,000 KZT.
11. [ ] The daily limit is reset at midnight Almaty time.
12. [ ] When both the amount and daily-limit conditions are violated, the amount validation error has priority.

---

# 5. Defect Reports


### BUG-001 - Transfer amount above maximum is accepted
* **Title:** `[Transfers]: Amount above 500,000 KZT is accepted`
* **Traceability:** REQ-3.1
* **Environment:** Test environment
* **Preconditions:**
  * User is logged in.
  * User can initiate a transfer.
* **Steps:**
  1. Enter 500,001 KZT.
  2. Continue the transfer.
* **Expected result:** The amount is rejected and an amount validation error is displayed.
* **Actual result - mocked:** The application accepts the amount and allows the transfer to proceed.
* **Reproducibility:** 5/5 simulated attempts
* **Severity:** High
* **Priority:** High
* **Evidence:** Mocked application log showing acceptance of 500,001 KZT.

---

### BUG-002 -Third incorrect SMS code does not cancel transfer
* **Title:** `[Transfers]: Transfer remains active after three incorrect SMS codes`
* **Traceability:** REQ-3.3
* **Environment:** Test environment
* **Preconditions:**
  * A transfer above 100,000 KZT is waiting for SMS verification.
* **Steps:**
  1. Enter an incorrect 6-digit SMS code.
  2. Enter another incorrect 6-digit SMS code.
  3. Enter a third incorrect 6-digit SMS code.
* **Expected result:** The transfer is cancelled after the third incorrect code.
* **Actual result - mocked:** The transfer remains in the SMS verification state after the third incorrect code.
* **Reproducibility:** 5/5 simulated attempts
* **Severity:** High
* **Priority:** High
* **Evidence:** Mocked application state log showing the transfer remaining active after three incorrect attempts.

---

### BUG-003 - Expired SMS code is accepted
* **Title:** `[Transfers]: Expired SMS code is accepted and transfer is completed`
* **Traceability:** REQ-3.3
* **Environment:** Test environment
* **Preconditions:**
  * A transfer above 100,000 KZT is waiting for SMS verification.
  * A valid SMS code has been issued.
* **Steps:**
  1. Wait for more than 120 seconds.
  2. Enter the valid SMS code.
  3. Confirm the transfer.
* **Expected result:** The expired SMS code is rejected and the transfer is not completed.
* **Actual result - mocked:** The application accepts the expired code and completes the transfer.
* **Reproducibility:** 5/5 simulated attempts
* **Severity:** Critical
* **Priority:** High
* **Evidence:** Mocked application log showing a successful transfer after SMS-code expiration.

# 6. AI Appendix

## 6.1 Summary of AI Usage
AI was used strictly as a formatting and grammar assistant. All QA analysis, test design techniques (EP, BVA, Decision Tables, State Transitions), and coverage decisions were performed manually by myself.

## 6.2 Key Manual Corrections (Fixing AI Hallucinations)
I critically reviewed all AI drafts and corrected the following unstated assumptions:

1. **Decimal Amount Handling (`TR-AMT-005`):** Removed the AI's invented UI error message. Re-classified this scenario as an open requirement gap (*Pending Analyst Clarification*) since the brief does not specify UI behavior for non-integer inputs.
2. **Traceability Matrix Statuses:** Changed `REQ-3.2` to **Partially Covered** (midnight reset is not verified within the 12 planned test cases) and explicitly marked `REQ-3.4` (Insufficient Balance) as **UNCOVERED**.
3. **Environment Data Clean-up:** Stripped out fabricated parameters (`TE-1, App v4.12.0`), keeping the field clean as `Test environment`.
4. **Terminology Alignment:** Removed added timezone labels (`UTC+5`) to strictly match the exact wording from the brief (*midnight Almaty time*).

## 6.3 Conclusion
While AI helped speed up Markdown formatting, it showed a clear weakness in QA domain logic by inventing missing specifications. All test designs, boundaries, and coverage decisions were independently made and verified by me.
