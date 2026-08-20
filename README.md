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

<img width="1543" height="398" alt="Screenshot 2026-08-20 142113" src="https://github.com/user-attachments/assets/547a2738-25b7-49de-af08-4d4f9d38382a" />

# Key Insights

-> Admission approval sits at 86.57%, but only 48.87% of all loans ever reach Settlement — the real leakage happens after admission approval, concentrated most sharply between Discharge Raised (63.56% of the original volume) and everything before it: nearly 4,000 approved loans never even raise a discharge request.

-> CF BRE risk tier is close to a hard gate: Green loans settle at a much higher rate than Red, and the Amber tier — 32.35% of all decisions — is where the real underwriting judgment call concentrates.

-> Discharge Approved (99.95% of Discharge Raised) is nearly a rubber stamp — once a discharge request is raised, it's almost always approved. The actual bottleneck is upstream, in whether a discharge ever gets raised at all.

-> Repeat borrowers settle at a notably higher rate (57.06%) than first-time borrowers (42.94%), despite being just 11.08% of the borrower base — a small, proven segment that converts meaningfully better.

-> Weekly cohort retention decays fast and stays low — roughly 3–6% of any given week's new borrowers return within the first week or two, tapering further after that. Expected for a hospital-admission product (most people don't need repeat hospitalization within weeks), not a red flag on its own.

-> 5 of 27 hospitals still operate on an unmapped GUID instead of a standard code — a small but concrete operational/data-onboarding gap, now visually flagged red on the leaderboard instead of invisible in the raw ID text.

-> Loan Breach Rate for admission decisions is just 0.05% — the credit decision engine is almost always fast (median well under a minute); the real turnaround problem sits in LBA Sign TAT (avg 19.74 hrs), where a small number of stuck cases pull the average far above what most loans actually experience.

# Data Model

Star schema, built entirely from a single source export (Loan_Data.xlsx, 11,643 rows).

<img width="994" height="579" alt="Screenshot 2026-08-20 125943" src="https://github.com/user-attachments/assets/7f15c40d-e954-4af7-82e1-ff64e1050e91" />

-> Fact_Loan (grain: one row per loan) — admission/discharge/LBA decisions and dates, parsed TAT fields, Bureau Score fields, borrower and geography attributes

-> Dim_Hospital — Hospital ID, Hospital_ID_Format (27 hospitals)

-> Dim_Geography — Borrower City, Borrower State, Geography_Key (composite key, since relationships can only join on one column)

-> Dim_Borrower — Borrower unique identifier, Borrower_Segment, Cohort_Week_Start, First_Loan_Date (9,856 borrowers)

-> Dim_Date — full daily calendar, marked as the official Date Table for time intelligence

Relationships: all four dimensions relate 1-to-many into Fact_Loan. Dim_Date connects on admission date (active).

Data Cleaning (Power Query) : 

Issue	     :              Fix

TAT fields stored as text ("2 hours and 59 minutes") : Parsed to numeric minutes via Text.BeforeDelimiter / Text.BetweenDelimiters

Bureau Score mixes real scores with reason/no-hit codes (-1, 11, 14–18) :	Split into Bureau_Score_Clean (nulls out codes, safe to average) and Bureau_Score_Category (visible in every risk breakdown)

Hospital ID format inconsistent — 22 standard 8-char codes vs. 5 full GUIDs :	Flagged via Hospital_ID_Format, surfaced with conditional formatting on the leaderboard rather than hidden

Negative Discharge Amount (449 rows — settlement adjustments) :	Flagged via Discharge_Amount_Flag, kept in the model, never deleted or zeroed

High null % on Partner BRE Decision, Discharge*, Insurance* fields : Not treated as dirty data — these are structural nulls from the funnel (a rejected admission never reaches a discharge decision). Every rate measure divides against the correct denominator, never a blind row count

Cohort Retention — built live, not as a static import :

The Weekly Acquisition Cohort matrix runs entirely on DAX against the live model, via a calculated bridge table (Cohort_Bridge) that cross-joins every borrower with 11 week-offsets, flags whether that borrower had a loan in each resulting target week, and only counts weeks that have actually occurred yet (a 2-week-old cohort has no "Week 8" data — not a zero). Retention updates automatically on refresh, no manual re-export needed.

# Tools & Techniques

-> Power BI Desktop — star-schema data modeling, Power Query cleaning, DAX measures

-> Power Query (M) — text-to-minutes TAT parsing, Bureau Score reason-code isolation, structural-null-aware cleaning (nulls preserved where they represent "stage not reached," never blanket-filled)

-> DAX — funnel/rate measures, USERELATIONSHIP-aware time intelligence, a calculated bridge table (GENERATESERIES + CROSSJOIN) powering a fully live weekly cohort retention matrix

-> Visuals used — KPI cards, funnel chart (via a disconnected stage-order table), line trend, donut charts, filled map, clustered bar comparisons, matrix with conditional-formatting heatmaps, scatter plot, conditionally formatted detail table

# Repository Contents

├── README.md

├── screenshots/

│   ├── 00_data_model.png

│   ├── 01_portfolio_overview.png

│   ├── 02_risk_underwriting.png

│   └── 03_operations_cohorts.png

├── Healthcare_Loan_Portfolio_Analysis.pbix

└── data/                                   (source dataset, if shared)

# How to Use

1. Clone the repo and open Healthcare_Loan_Portfolio_Analysis.pbix in Power BI Desktop.
   
2. Use the Month, Borrower State, and Insurance Type slicers at the top of any page to filter the full report.

3. Navigate between pages using the icon rail on the left (Overview, Risk & Underwriting, Operations & Cohorts).

4. On Page 3, read the cohort matrix by row — each row is a group of borrowers who took their first loan that week; columns show what % of them returned N weeks later. Week 0 is always 100% by definition.

# About This Project

Built as an end-to-end BI project: a raw operational export cleaned with an explicit, documented null-handling policy (structural nulls preserved, genuine data gaps patched), modeled into a star schema, layered with a full DAX measure library, and extended with a fully live cohort-retention engine built from scratch via a DAX bridge table — no manual re-exports required as the underlying data changes.
