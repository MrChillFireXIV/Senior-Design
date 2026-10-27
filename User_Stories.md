# User Stories and Use Cases: Smart Barbell Load Cuffs

**Assignment 4, Part 1** · Tag: `week04`  
**Team:** Tyler Beck, Colin Hill

**Project in one line:** Two thin rings clamp onto a standard Olympic barbell. Each ring measures the weight loaded on its side (shear strain gauges) and tracks reps and bar speed (IMU). A lifter taps their phone to the ring (NFC) to sign in, and every set logs to their account automatically.

---

## 1. Stakeholder map

| Category | Stakeholder | What they need from the system | How we reach them |
|---|---|---|---|
| **Primary** | Recreational lifter at the campus rec center | Accurate weight and rep log with no typing between sets | Interviews and observation on the gym floor |
| **Primary** | Strength coach running team sessions | Per-rep bar speed for each athlete, to adjust load the same day | Interview with a varsity or club coach |
| **Secondary** | Rec center equipment manager | Bars that stay charged and working; easy to see which ones need attention | Interview with facility staff |
| **Secondary** | Front-desk / floor staff | Answers for members when a bar won't connect | Short survey of staff |
| **Hidden** | University data privacy officer (compliance) | One member's workouts never recorded under another member's account on a shared bar | Interview or email with the privacy / IT security office |
| **Hidden** | Downstream fitness apps (Google Health Connect, Apple Health) | Workout records in a standard format they can import | Competitive analysis of their data formats |
| **Hidden** | Custodial staff who disinfect equipment | Rings that survive daily wipe-downs with gym disinfectant | Observation of the cleaning routine |

---

## 2. User stories

**US-01 (Primary: recreational lifter)**  
As a recreational lifter at the campus rec center,  
I want the weight and rep count of each set recorded without entering them myself,  
so that my training log is complete and I spend my rest time resting instead of typing.

**US-02 (Primary: strength coach)**  
As a strength coach running a team lifting session,  
I want each athlete's bar speed for every rep of their working sets,  
so that I can raise or lower their load that same session based on measured velocity instead of guesswork.

**US-03 (Secondary: equipment manager)**  
As the rec center's equipment manager,  
I want to know which bars have low batteries or have stopped reporting,  
so that I can charge or fix them before opening and no bar is out of service when members arrive.

**US-04 (Hidden: compliance, university data privacy officer)**  
As the university's data privacy officer,  
I want a lifter's session on a shared bar to end once they stop using it or someone else signs in,  
so that no member's sets are ever stored in another member's account.

---

## 3. Use cases

### UC-01: Log a set automatically

**Expands:** US-01  
**Primary actor:** Recreational lifter  
**Secondary actors:** Load cuffs (two rings on the bar), lifter's phone (web app over Bluetooth), account server

**Preconditions** (each one can be checked by a tester before starting)

1. The lifter has an account and is signed in on an Android phone with Bluetooth and NFC turned on.
2. Both rings are clamped on the bar, one inside each collar, and each reports a battery level of at least 20%.
3. Both rings were calibrated within the last 30 days (calibration date is shown in the app).
4. The bar is resting in the J-hooks with plates and collars on.

**Main success flow**

1. **Lifter:** taps their phone to either ring.
2. **System:** wakes the bar, opens the logging page, connects, and shows the bar's ID within 5 s.
3. **Lifter:** lifts the bar out of the J-hooks and holds it still.
4. **System:** after 1 s of stillness, reads both sides, adds the 20 kg bar, and shows the total weight.
5. **Lifter:** performs the set.
6. **System:** counts each rep and records its peak and mean concentric velocity.
7. **Lifter:** returns the bar to the J-hooks.
8. **System:** after 10 s with no reps, closes the set and shows a summary: weight, reps, and velocity per rep.
9. **Lifter:** accepts the summary (or does nothing for 30 s, which counts as accepting).
10. **System:** saves the set to the lifter's account with the bar ID and a timestamp.

**Alternate flow A1: Next set without tapping again**

- A1.1 (after step 10) **Lifter:** changes plates and lifts the bar again within 10 minutes.
- A1.2 **System:** keeps the same session and continues from step 4, measuring the new weight.

**Alternate flow A2: Lifter corrects the rep count**

- A2.1 (at step 9) **Lifter:** changes the rep count, for example to drop a partial rep.
- A2.2 **System:** saves the corrected count and marks the set as edited, keeping the measured count for later accuracy analysis.

**Exception flow E1: Bluetooth link drops during the set**

- E1.1 (during steps 5 to 8) The phone loses its connection to the bar.
- E1.2 **System (rings):** keeps recording the set in the ring's memory (up to 20 sets).
- E1.3 **System (phone):** reconnects automatically when back in range and uploads the stored sets with no duplicates.
- E1.4 If the phone does not reconnect within 10 minutes, the stored sets wait on the bar and upload the next time the same lifter taps in.

**Exception flow E2: Sides are loaded unevenly**

- E2.1 (at step 4) The two sides differ by more than 2.5 kg.
- E2.2 **System:** warns the lifter before the set, naming the lighter side and the difference, through both the screen and a vibration pattern on the phone.
- E2.3 **Lifter:** racks the bar and fixes the plates; the flow resumes at step 3.

**Postcondition**  
The set is stored in the lifter's account with total weight, rep count, per-rep velocity, bar ID, and time, and appears in their history. If the bar sees no motion for 10 minutes, it returns to sleep.

---

### UC-02: End a session on a shared bar

**Expands:** US-04  
**Primary actor:** Lifter A (the lifter who is leaving)  
**Secondary actors:** Lifter B (the next lifter), load cuffs, account server, university data privacy officer (audits the outcome)

**Preconditions**

1. Lifter A has an active session on the bar (UC-01, step 2 completed).
2. Lifter B has a separate account and is signed in on a different phone.

**Main success flow**

1. **Lifter A:** finishes their last set and walks away from the bar.
2. **System:** detects 10 minutes with no bar motion.
3. **System:** ends Lifter A's session, disconnects their phone, and tells Lifter A that the session ended and how many sets were saved.
4. **System:** puts the bar to sleep.
5. **Lifter B:** later taps their phone to the ring.
6. **System:** starts a new session for Lifter B only.

**Alternate flow A1: Lifter A ends the session on purpose**

- A1.1 (at step 1) **Lifter A:** ends the session from their phone before walking away.
- A1.2 **System:** ends the session immediately and continues from step 3.

**Exception flow E1: Someone taps in while Lifter A's session is still open**

- E1.1 (before step 2) **Lifter A or Lifter B:** taps a phone to the ring while Lifter A's session is still open.
- E1.2 **System:** checks the account signed in on the phone that tapped and compares it with the account that owns the open session.
- E1.3 **Same lifter (Lifter A taps again):** the system keeps Lifter A's session open, reconnects Lifter A's phone if it had dropped, and continues from UC-01 step 3. No wait is needed and no new session starts.
- E1.4 **New lifter (Lifter B), bar idle for less than 12 s:** Lifter A's last set is not confirmed yet, so the system does not accept Lifter B's tap. It tells Lifter B the bar is still in use and to tap again in a few seconds.
- E1.5 **New lifter (Lifter B), bar idle for at least 12 s:** the system confirms Lifter A's last set and saves it to Lifter A, ends Lifter A's session, and notifies Lifter A's phone.
- E1.6 **System:** starts Lifter B's session. Every set from this point forward goes only to Lifter B.

**Postcondition**  
At most one lifter is signed in to a bar at any time. Every set is stored in exactly one account, the account that was signed in when the set began.

---

## 4. Acceptance criteria (Given / When / Then)

### UC-01: Log a set automatically

**AC-01.1 (main success flow: weight and reps)**

- **Given** the UC-01 preconditions are met and the bar is loaded with 20 kg plates on each side (60.0 kg total, measured on a reference scale),
- **When** the lifter taps in, unracks, performs 5 full reps, and racks the bar,
- **Then** within 13 s of racking (10 s to close the set plus 3 s) the summary shows 60 kg ± 1.0 kg and 5 reps, and the set appears in the lifter's account history within 10 s.

**AC-01.2 (main success flow: connection time)**

- **Given** the bar has been asleep for at least 10 minutes,
- **When** the lifter taps their phone to a ring,
- **Then** the logging page shows the correct bar ID within 5 s, in at least 19 of 20 attempts.

**AC-01.3 (alternate flow A1: next set)**

- **Given** a set was saved less than 10 minutes ago,
- **When** the lifter loads 10 kg more and lifts again without tapping,
- **Then** the new set is saved to the same session with a weight 10 kg ± 1.0 kg higher than the previous set.

**AC-01.4 (exception flow E1: link drops)**

- **Given** a 5-rep set is in progress,
- **When** the phone's Bluetooth is turned off after rep 2 and turned back on within 2 minutes,
- **Then** within 15 s of reconnecting the set is saved once (no duplicate) with all 5 reps.

**AC-01.5 (exception flow E2: uneven load)**

- **Given** 20 kg is on the left side and 15 kg is on the right,
- **When** the lifter unracks the bar and holds it still for 1 s,
- **Then** within 2 s the phone shows a warning naming the right side as lighter by 5 kg ± 1.0 kg and plays a vibration pattern, and no set is recorded until the sides are within 2.5 kg.

### UC-02: End a session on a shared bar

**AC-02.1 (main success flow: idle timeout)**

- **Given** Lifter A has an active session and has finished a set,
- **When** the bar is left untouched,
- **Then** Lifter A's session ends between 10:00 and 10:30 minutes after the last motion, Lifter A's phone shows the number of sets saved, and the bar's current drops to sleep level (under 100 µA).

**AC-02.2 (exception flow E1: new lifter taps before the set is confirmed)**

- **Given** Lifter A has an open session and racked the bar 5 s ago,
- **When** Lifter B taps their phone to the ring,
- **Then** within 2 s Lifter B's phone shows that the bar is still in use, no session starts for Lifter B, and Lifter A's set is still pending in Lifter A's session.

**AC-02.3 (exception flow E1: new lifter taps after the set is confirmed)**

- **Given** Lifter A has an open session and the bar has been idle for at least 12 s,
- **When** Lifter B taps their phone to the ring and completes a 3-rep set,
- **Then** within 5 s of the tap Lifter A's phone shows that the session ended, Lifter A's history contains their last set, Lifter B's history contains the 3-rep set, and Lifter A's history contains 0 sets timestamped after Lifter B's tap.

**AC-02.4 (exception flow E1: same lifter taps again)**

- **Given** Lifter A has an open session and racked the bar 5 s ago,
- **When** Lifter A taps their own phone to the ring,
- **Then** within 2 s Lifter A's phone shows the same session (no new session starts), and Lifter A's next set is saved to that session.
