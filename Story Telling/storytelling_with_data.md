# 📖 Storytelling with Data

## Business Problem
A Portuguese bank ran a large-scale **direct marketing campaign** — mostly outbound phone calls — to convince clients to open a **term deposit**. Every call has a cost: the agent's time, the client's patience, and a limited number of dials a call center can make in a day. The bank needs to know **who is actually worth calling, when, and how**, so marketing spend goes toward the clients most likely to say yes instead of being spread evenly across a client base that, as the data shows, mostly says no. This analysis turns 41,188 raw call records into a set of concrete answers about where the bank's next campaign should focus.

## Dataset Overview
The dataset (`bankmarketing(4).csv`) contains **41,188 records and 21 columns** covering three layers of information about each contact attempt:
- **Client profile** — age, job, marital status, education, and existing credit/loan/default status
- **Campaign details** — contact type, month, day of week, call duration, number of contacts made, and outcome of any previous campaign
- **Macroeconomic backdrop** — employment variation rate, consumer price/confidence index, Euribor 3-month rate, and number of employees in the economy at the time of contact
- **Target outcome (`y`)** — whether the client subscribed to a term deposit (`yes`/`no`)

## Key Questions Explored
- What does a "typical" client contacted by the bank look like?
- Does the client's job or age influence whether they subscribe?
- How imbalanced is the outcome, and what does that mean for interpreting results?
- Does call behavior (duration, number of contacts) relate to other campaign or economic factors?
- Which communication channel is the bank actually using, and what does that say about its outreach strategy?

## Data Cleaning Journey
The raw data looked complete at first glance — **zero explicit `NaN` values** — but a closer look told a different story:

- **Disguised missing data**: columns like `default`, `education`, `housing`, `loan`, `job`, and `marital` used the string `"unknown"` to hide missing information — `default` alone had **8,597 unknown entries** (about 1 in 5 clients). Dropping these rows would have thrown away a fifth of the dataset and skewed the results, so `"unknown"` was deliberately kept as its own valid category rather than deleted or guessed at.
- **Duplicates**: **12 exact duplicate rows** were found and removed, tightening the dataset to 41,176 unique records.
- **Outliers**: the IQR method flagged real extremes in `age`, `campaign`, `duration`, and `previous` — instead of deleting these rows (and losing genuine customers with long calls or repeat contacts), values were **capped at the IQR bounds**, keeping every record while preventing a handful of extreme values from distorting the averages.
- **A sentinel value left alone**: `pdays` uses `999` to mean "never contacted before." This is a *code*, not a true outlier, so it was correctly left untouched — a good example of knowing when *not* to apply a cleaning rule.

This cleaning approach — preserve records, categorize the unknowns, cap rather than cut — reflects a conservative, business-safe philosophy: **never let the cleaning process itself introduce bias** into who ends up represented in the analysis.

## Exploratory Data Analysis

### Insight 1 — The Bank's Core Audience Is Mid-Career Adults
- **Observation**: The age histogram is right-skewed, with the tallest bars clustered between roughly 28 and 40 years old, tapering off sharply after 50 and again after 60.
- **Business Interpretation**: The bank's outreach — whatever list or channel it draws from — is dominated by working-age adults in their 30s, not retirees or very young clients. This isn't necessarily who *should* be called most, just who currently *is* being called most.
- **Why it Matters**: If the most valuable segment turns out to lie outside this core 30–40 age band (see Insight 2), the bank is spending most of its dialing effort on a group that isn't its best converter — a mismatch between contact volume and contact value.

### Insight 2 — Students and Retirees Convert Far Better Than Anyone Else
- **Observation**: Subscription rate by job tells a very different story than call volume. **Students subscribe at ~31–32%** and **retired clients at ~25%** — both roughly double the third-highest group (unemployed, ~14%). At the bottom, **blue-collar workers convert at only ~7%**, despite being one of the largest job categories in the dataset.
- **Business Interpretation**: The clients least likely to be the "default" target of a campaign — students and retirees, who typically have smaller balances and lower income — are actually the most receptive to the offer. Meanwhile, blue-collar and services workers, who likely make up a large share of the calls made, convert the worst.
- **Why it Matters**: This is the single most actionable finding in the dataset. If the bank is currently calling blue-collar and services clients in the same proportion it calls students and retirees, it's burning agent time on the lowest-yield segments while under-investing in the highest-yield ones.

### Insight 3 — The Outcome Is Heavily Imbalanced, and That Changes How Every Other Chart Should Be Read
- **Observation**: Of 41,176 cleaned records, roughly **89% did not subscribe** and only **~11% did** (about 4,640 "yes" outcomes).
- **Business Interpretation**: Term deposit conversion is a low-probability event. Campaign success isn't about flipping the majority of clients — it's about finding the specific slice of clients where the odds shift meaningfully above that 11% baseline (like students and retirees at 25–32%).
- **Why it Matters**: This imbalance is also a modeling warning sign: any predictive model built on this data (e.g., to score call priority) needs to account for the skew, or it will simply learn to predict "no" for everyone and still look "accurate" while being useless for targeting.

### Insight 4 — The Client Base Skews Married, Which Sets the Context for Every Demographic Cut
- **Observation**: Married clients make up the largest group by a wide margin (~24,900), followed by single clients (~11,500), with divorced clients (~4,500) and an "unknown" sliver rounding out the total.
- **Business Interpretation**: Any targeting strategy the bank builds will, by sheer volume, mostly be reaching married clients — so campaign messaging and offers should be calibrated with that household context (e.g., joint savings goals) in mind by default, while still carving out distinct messaging for the sizable single-client segment.
- **Why it Matters**: Marital status alone isn't a strong predictor here, but it's useful as a segmentation and messaging variable once combined with stronger signals like job and age.

### Insight 5 — Subscribers Skew Slightly Older, With a Wider Age Range Than Non-Subscribers
- **Observation**: The box plot of age by subscription outcome shows subscribers have a noticeably wider interquartile range (roughly 31–50 years) than non-subscribers (roughly 32–47 years), with subscribers' median and upper quartile both shifted higher.
- **Business Interpretation**: This lines up directly with Insight 2 — the pull toward retirees among subscribers stretches that age box upward, while the age range of non-subscribers stays more tightly clustered around the bank's core working-age callers.
- **Why it Matters**: Age isn't a strong standalone predictor, but combined with job it reinforces that the bank should be less afraid to call older clients than the current age distribution (Insight 1) suggests it currently is.

### Insight 6 — Most Calls Are Short, But a Meaningful Minority Run Long — And That's a Trap for Interpretation
- **Observation**: Even after capping extreme values, call duration remains strongly right-skewed: the median sits around 180 seconds (3 minutes), with the box plot's upper whisker reaching all the way to the capped ceiling (~645 seconds), showing plenty of calls that ran far longer than typical.
- **Business Interpretation**: Longer calls are well known in this type of dataset to correlate with genuine interest (a client who stays on the phone is a client who's engaged) — but duration is only known *after* the call ends, so it can't be used to decide who to call in the first place. It's a great explanatory variable, a poor targeting variable.
- **Why it Matters**: Any temptation to build a "call longer to convert more" strategy is backwards — duration is a *symptom* of interest, not a lever the bank can pull before dialing.

### Insight 7 — The Macroeconomic Indicators Move Together, and Duration Marches to Its Own Beat
- **Observation**: The correlation heatmap shows `emp.var.rate`, `euribor3m`, and `nr.employed` are very strongly correlated with each other (0.91–0.97), while `duration` correlates weakly with essentially every numeric feature (all values near 0, e.g. -0.08 with `campaign`).
- **Business Interpretation**: The three economic indicators aren't three independent signals — they're one macroeconomic story told three times (interest rates, employment levels, and economic sentiment tend to rise and fall together). Call duration, by contrast, is driven by something the dataset doesn't capture directly — most likely individual client interest — rather than by campaign mechanics or the economic climate.
- **Why it Matters**: For any future modeling work, the bank should treat those three correlated economic fields as one signal (to avoid double-counting the same information), and should look outside this dataset — client conversation quality, offer relevance — to explain what actually drives longer, more productive calls.

### Insight 8 — The Bank Has Already Shifted to Mobile-First Outreach
- **Observation**: The contact-type pie chart shows **63.5% of contacts were made via cellular, versus 36.5% via landline telephone**.
- **Business Interpretation**: The bank's calling strategy already leans mobile, which matters because cellular contact in this kind of campaign data typically reaches clients more directly and personally than a shared home landline.
- **Why it Matters**: This channel shift is worth protecting and potentially deepening — but it should be validated against the actual subscription rate by channel (not shown in the current charts) before assuming "more cellular" automatically means "more conversions."

## Hidden Patterns
- **The `previous` column quietly lost all its variation during cleaning.** The IQR bounds calculated for `previous` came out as **0 to 0** — because the vast majority of clients had never been contacted before this campaign, the interquartile range itself is zero. Capping to those bounds forced every single value in the column to 0, which is why the heatmap shows `previous` as a blank row and column. This is a subtle but important cautionary tale: **applying a single outlier rule uniformly across very different columns can silently erase a real signal** (in this case, whether a client had prior campaign contact — potentially a meaningful predictor of a repeat "yes"). It's a reminder to visually or statistically sanity-check the *after* state of every capped column, not just the ones that "look" like they had obvious outliers.
- **The biggest converters are the smallest contact groups.** Students and retirees convert best, yet the age distribution (Insight 1) shows the campaign's calling volume is centered on the 30–40 age band where those groups are least represented. The bank's dialing effort and its highest-yield audience are currently pointed in different directions.

## Business Recommendations
1. **Re-prioritize the call list by job segment.** Shift a larger share of outbound calls toward students and retirees, and reduce volume to blue-collar and services segments unless paired with a stronger, more tailored offer.
2. **Build age into targeting alongside job, not instead of it.** Since subscribers skew slightly older with a wider spread, don't let a "young working professional" default assumption crowd out older clients — especially retirees.
3. **Use call duration as a live coaching signal, not a targeting filter.** Since long calls associate with interest but can't predict it in advance, use early call duration as a real-time cue for agents to escalate engagement (e.g., offer more detail, move to next steps) rather than as a pre-call scoring metric.
4. **Collapse the three macroeconomic indicators into one composite signal** before feeding them into any future scoring or forecasting model, to avoid overweighting what is effectively the same economic trend three times.
5. **Fix the `previous` column before using it in any further analysis.** Recompute outlier handling for that field separately (or exclude it from blanket IQR capping) so prior-contact history isn't lost — it's a plausible strong predictor of repeat conversions that the current cleaning step accidentally erased.
6. **Double-check that the cellular-contact shift is paying off.** Confirm mobile contacts convert better than landline contacts before investing further in a fully mobile-first calling strategy.

## Conclusion
The data tells a consistent story: this bank's marketing campaign is casting a wide net centered on the wrong part of the pond. Its heaviest calling volume goes to mid-career, working-age clients — yet its best results, by a wide margin, come from students and retirees. Overall conversion is low (about 11%), which is normal for this kind of campaign, but the variation *within* that 11% — by job, and to a lesser extent by age — is where the real opportunity sits. Add to that a data-cleaning artifact that silently zeroed out a potentially valuable "prior contact" signal, and the path forward becomes clear: tighten the targeting toward proven high-converting segments, fix the analytical blind spots, and let call duration inform coaching rather than dialing decisions. None of this requires new data collection — it requires the bank to act on the patterns already sitting inside the data it has.
