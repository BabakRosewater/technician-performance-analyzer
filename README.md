# technician-performance-analyzer
1. What this app is for

A config-driven Tech Pay Calculator that:

Pulls actual production hours from your technician-performance-analyzer repo

Pulls rates, tiers, workdays, and days-off from your tech-pay-config repo

Calculates base pay, bonus, and total pay for each technician for any period

Lets you override anything manually (rate, hours, days off, bonus) without breaking the auto logic

Exports a clean CSV you can send to payroll or keep as backup.

2. Data sources + file naming

Production repo (technician-performance-analyzer):

Mid file (for H1 & H2 math):
technicians_MM_15_YYYY.csv

End file (for H2 & MTD):
technicians_MM_DD_YYYY.csv where DD is probed 31 → 28.

Config repo (tech-pay-config):

plans.csv – plan IDs (e.g., flat_v1) tied to techs

tech_rates.csv – technician base rates per plan

tech_aliases.csv – maps “Calen W.” → “Calen Weed” etc.

efficiency_bonus_tiers.csv – min efficiency % and $/prod hr

workdays_by_period.csv – e.g. MT014, 2025-11, H1 = 11 workdays

Monthly days-off file (per month):
tech_days_off_by_period_YYYY_MM_full.csv
(You’ll create a new one each month and delete the generic file.)

3. Context and period logic

At the top of the app we choose:

Store – MT014 or “Clark Hyundai” (both normalize to MT014)

Plan – currently flat_v1

Year / Month

Period – H1, H2, or MTD

Hours per day – default 8 (for efficiency math)

Key behaviors:

Changing Year or Month automatically invalidates production + days-off (so you’re forced to reload for that context).

Workdays come from workdays_by_period.csv for that store + year + month + period.

4. How the pay math works

For each tech (after aliases):

Production hours (Prod Hrs)

If Auto:

H1 → mid file total

H2 → (end − mid)

MTD → end total

If Manual: uses the number you type.

Rate

Currently comes from tech_rates.csv (flat hourly)

Auto vs Manual still uses the same base value, but you can override.

Days Off & Workdays

Workdays = from workdays_by_period.csv

Days Off (Auto) = from tech_days_off_by_period_YYYY_MM_full.csv filtered by:

store, year, month, period, technician

Days Worked = Workdays − Days Off (never below 0)

Efficiency

Hours available = Days Worked × Hours per day

Efficiency = Prod Hrs ÷ Hours available

Efficiency % determines the tier from efficiency_bonus_tiers.csv.

Bonus

If Bonus = Auto:
Bonus = Prod Hrs × bonus_per_prod_hour (from tier)

If Bonus = Manual: uses your number.

Total Pay

Base Pay = Rate × (Prod Hrs + Foreman Hrs)

Total Pay = Base Pay + Bonus

Vacation field is informational only right now (not included in math).

5. UI / UX decisions

Top “pill” buttons

Load Everything (Latest)

Finds latest technicians_* files on GitHub and sets Year/Month to match.

Load Selected

Uses the Year/Month/Period currently chosen and loads:

workdays

monthly days-off file

mid + end production files

Apply Defaults

Resets toggles (Rate/Prod/Days Off/Bonus) to Auto without wiping your manual values.

Export CSV

Print (print-friendly layout).

Tech grid layout

Stacked but more compact to reduce empty space.

Each row has toggles (Auto / Manual) for:

Rate, Prod, Days Off, Bonus

Once rendered, inputs don’t get rewritten on every keystroke → fixes the “only one digit at a time” bug.

We update only that row’s outputs (days worked, efficiency, tier, bonus, total) on input.

KPI panel

Base Pay Total

Bonus Total

Grand Total

Displays Period total hours across all techs (based on effective Prod).

Search + sort

Search techs by name.

Sort by:

Name (A→Z)

Total Pay (High→Low)

Efficiency (High→Low)

6. Current status

Right now the app:

Correctly calculates Days Worked = Workdays − Days Off using:

workdays_by_period.csv

tech_days_off_by_period_YYYY_MM_full.csv

Uses actual production files for mid & end.

Applies tiered bonuses correctly based on efficiency.

Supports fast manual overrides in a way that’s smooth to edit.

Exports a clean CSV with all of the above plus metadata (store, context, file names).
