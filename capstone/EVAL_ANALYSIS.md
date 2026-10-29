# Evaluation Results and Error Analysis

## Evaluation setup
The evaluation set contains 16 tasks. Configurations A and B
were evaluated on the same tasks using automatic graders.

Five LLM-based judge graders were removed from the evaluation copy
(tasks-auto.jsonl) to reduce model requests. The original tasks.jsonl
remains unchanged. These results therefore represent the automatic
graders retained in tasks-auto.jsonl, not the full original grader suite.

## Final results

| Metric | Configuration A | Configuration B |
|---|---:|---:|
| Tasks | 16 | 16 |
| Passed | 12 | 14 |
| Failed | 4 | 2 |
| Pass rate | 75% | 87.5% |

Configuration B improved the observed pass rate by 12.5 percentage
points compared with A.

## Error analysis: A

- rules-structural-wall: the answer did not include an expected
  engineer-related term.
- rules-event-too-late: the answer did not include the expected value 20.
- rules-missed-inspection: the answer did not include the expected value 150.
- status-verified: the expected get_permit tool was not called, and
  the answer did not include the expected review-related wording.

## Error analysis: B

- rules-office-hours: the answer did not include the expected wording
  indicating closure, leave, or a holiday.
- rules-failed-inspection: the answer did not include the expected
  no-fee wording.

## Configuration change

Configuration A uses the base system specification.
Configuration B adds explicit instructions to use the relevant tool
for fee and date questions and to preserve the fee returned by the
tool in the final answer.

In the five-task smoke test, the initial B configuration passed 4/5.
After strengthening the fee instructions, B passed 5/5. The final
16-task evaluation also showed a higher pass rate for B than A.

## Limitations

The evaluation uses automatic graders only; five original judge
graders were omitted from tasks-auto.jsonl. Results are specific
to this 16-task dataset and do not establish production readiness
or prove general performance on other tasks.

A separate adversarial prompt-injection and data-exfiltration test
remains outstanding. The tested write-approval path is documented
in SECURITY.md.
