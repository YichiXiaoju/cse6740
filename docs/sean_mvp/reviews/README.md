# Initial human review records

These files preserve Sean's 200 ID and 60 OOD judgments. They are initial review data, not independently adjudicated ground truth. The original sample files remain blank templates.

- `ID_review_completed.csv` / `OOD_review_completed.csv`: original judgment values, normalized to UTF-8 and LF for git; uploaded-byte hashes in `review_sources.json`.
- `initial_review_summary.json`: counts and initial match-flag rates. The legacy `false_negative_rate` key is computed from these judgments; interpret N2 as preliminary flag rates pending adjudication.
- `ID_N3_retained_initial_review.csv` / `OOD_N3_retained_initial_review.csv`: 100 / 19 N3 negatives retained under the initial review, with original splits. These files contain negatives only.
- `n2_adjudication_queue.csv`: two specifically observed disagreements needing a second reviewer decision. Labels are not silently changed; this is not an exhaustive dispute list.
- `upstream_n2_diagnostics.json`: deterministic structural issues in the upstream ID N2 table, separate from subjective review disagreements.

N3 acceptance does not establish absence of text-only shortcuts: the MVP substitutes only four simple target colors and undercovers complex color names. No text-only control has been run. Other N3 candidates remain unreviewed. Near/far OOD category choices remain provisional; no OOD threshold tuning or training is permitted.
