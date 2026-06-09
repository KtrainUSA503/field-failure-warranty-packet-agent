# Power Automate flow — Warranty Claim Actions

One flow, triggered from the approval card's `Action.Submit`, switching on `data.action`.

## Trigger
Copilot Studio topic passes the card submission payload (action, actionSubmitId, claimId,
serial, verdict, completenessScore, approvedBy, missing).

## Topic-side pre-gate (defense in depth)
Before the flow is called, the topic's condition node re-checks:
`action = "approve"` AND `IsBlank(Topic.missingItemsSummary)`.
The card's disabled button is UI; this is enforcement. The else-path replies:
"Can't file — required items are still missing. Use Request Missing Items."

## Branch: request_items
1. Post a Teams message (Flow bot or service account) to the field tech.
2. Body: claim ID, serial, and the numbered missing-item list from `missing`.
3. Reply in topic: confirmation that the request was sent.

## Branch: approve
1. Create item in the **Warranty Claims** SharePoint list.
2. Read the item back by ID and compare key fields (serial, verdict, score).
3. On match → reply with claim record link + "Filed and verified."
4. On mismatch/failure → error branch: reply "Filing failed — nothing was recorded"
   (or "record created but verification failed — check manually"), with run link.
   Never report success that wasn't verified.

## Branch: reject
Update conversation state; no system writes. Optional: log rejection to the list
with status Rejected for the audit trail.

## SharePoint list schema — Warranty Claims
| Column | Type | Notes |
|---|---|---|
| Title | Single line | claimId ({serial}-{failureDate}) |
| Serial | Single line | |
| FailedPart | Single line | |
| Verdict | Choice | Eligible / Needs Review / Likely Excluded |
| CompletenessScore | Number | |
| FailureMode | **Multi-line text** | single-line truncates at 255 chars |
| ExclusionRisk | **Multi-line text** | |
| MissingItems | **Multi-line text** | |
| Sources | **Multi-line text** | |
| ApprovedBy | Single line | captured at decision time |
| ApprovedAt | Date/time | |
| Status | Choice | Filed / Rejected |

## Telemetry (no Azure)
Power Automate run history + Copilot Studio analytics + the list itself as audit record.
