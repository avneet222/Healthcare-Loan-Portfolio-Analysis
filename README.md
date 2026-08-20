# Healthcare Loan Portfolio Analysis
A 3-page Power BI dashboard analyzing 11,643 hospital-financing loans — an embedded "cashless credit for medical bills" product spanning admission through settlement. Built to answer one core question: the pipeline processes thousands of loans, but where exactly does it leak, and who's carrying the risk?

The Product, in One Sentence

A patient is admitted → the hospital raises an Admission request → a risk engine (CF BRE = internal decision engine, Partner BRE = insurer routing) approves or rejects → on discharge, the hospital raises a Discharge request for the final bill → an LBA (Loan/Bill Agreement) gets signed → the case moves to Settlement. Every KPI on this dashboard traces back to a specific point in that pipeline.

# Dashboard Structure

Page 1 — Portfolio Overview

Executive view of volume, funnel health, and geography.

-> KPI cards: Total Loans (11,643), Total CF Approval Amount (₹118M), Admission Approval Rate (86.57%), Settlement Rate (48.87%), Avg Bureau Score (641.95)

-> Conversion Stages funnel — Admission Raised → Admission Approved → LBA Signed → Discharge Raised → Discharge Approved → Settlement Done, with stage-over-stage conversion %

-> Weekly Loan Volume Trend (W09–W19)

-> Loan Status Breakdown donut (Settlement Done / Awaiting Stages / Rejected / Dropped)

-> Loan Volume & Settlement Rate — India state map

-> Slicers: Month, Borrower State, Insurance Type

<img width="1199" height="668" alt="Screenshot 2026-08-20 135053" src="https://github.com/user-attachments/assets/97d00284-5ae2-49d1-a9fc-470d2b4c83a2" />

Page 2 — Risk & Underwriting

Deep-dive on the credit decision engine and supplier (hospital) risk.

-> KPI cards: No Bureau Hit Rate (7.80%), Prime Borrower Share (54%), Subprime Borrower Rate (6.06%), CF BRE Green/Amber/Red split (53.83% / 32.35% / 13.60%)

-> Admission Approval Rate vs. Settlement Rate by Bureau Score Category

-> CF BRE Decision × Partner BRE Decision matrix — approval/settlement rate by combination

-> Avg CF Approval Amount vs. Loan Volume by risk band

-> Hospital leaderboard — volume, approval rate, settlement rate per hospital, with Hospital ID Format conditionally flagged (5 hospitals still on an unmapped GUID instead of a standard code)

<img width="1197" height="665" alt="Screenshot 2026-08-20 130032" src="https://github.com/user-attachments/assets/fc909d6f-12e3-4c25-844c-ce08b4cf5474" />

Page 3 — Operations & Cohorts

Turnaround-time diagnostics and borrower retention behavior.

-> KPI cards: Avg Admission Decision TAT (0.69 min), Avg Discharge Decision TAT (1.33 min), Avg LBA Sign TAT (19.74 hrs), Loan Breach Rate >2hrs (0.05%), Drop-off Rate (7.15%)

-> Loan Performance Matrix — Discharge Amount vs. Final Bill Amount scatter, colored by Discharge_Amount_Flag to isolate negative-adjustment cases

-> Repeat vs. First-Time Borrower split (88.92% first-time / 11.08% repeat) and Settlement Rate by Segment (57.06% repeat vs. 42.94% first-time)

-> Weekly Acquisition Cohort matrix — live retention heatmap, borrowers grouped by the week of their first loan, tracked for repeat-loan activity across the following 10 weeks

<img width="1201" height="669" alt="Screenshot 2026-08-20 130051" src="https://github.com/user-attachments/assets/7c623953-e41a-4634-b677-f4f1aef88b50" />

# KPI Glossary




