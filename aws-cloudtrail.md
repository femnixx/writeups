# AWS CloudTrail Analysis — Cloud Compromise

**Category:** Cloud Forensics / Log Auditing
**Tools used:** Splunk (SPL queries against ingested CloudTrail logs)

**Note on process:** First time working with Splunk/SPL, so I asked for query syntax as I went. The reasoning behind each search (what event type and field to look for) was my own — I knew conceptually what I was looking for at each step and just needed help translating that into correct SPL, similar to needing Wireshark filter syntax early on with pcaps.

## Scenario
Investigate AWS CloudTrail logs (ingested into Splunk) for an external credential compromise: identify the initial compromised account, what the attacker accessed, how they tried to expose data, and how they established persistence.

## Investigation
- Worked through this by reasoning about what CloudTrail event name would correspond to each stage of a typical AWS compromise, then asked for the correct SPL syntax to search for it.
- Reasoned that the initial compromise would show up as a `ConsoleLogin` event, and that a successful one would have a specific response field — searched for `eventName=ConsoleLogin` with a successful result, which surfaced the compromised account: `helpdesk.luke`.
- Reasoned that once inside, the attacker would start reading objects — searched for `GetObject` events to find the first point of data access and which S3 bucket was targeted.
- Reasoned that if they were trying to expose data publicly, there'd be a specific API call for that — searched for `PutBucketPublicAccessBlock` (an event that disables a bucket's public-access restrictions), which identified the exposed bucket.
- Reasoned that persistence in AWS usually means creating a new account and escalating its permissions — searched for `CreateUser` followed by `AddUserToGroup`, which surfaced the rogue account (`marketing.mark`) and its addition to the Admins group.

## Key Findings
- Compromised account: helpdesk.luke (console login succeeded 2023-11-02 09:54)
- First S3 access: 2023-11-02 09:55 (GetObject)
- Targeted bucket: product-designs-repository31183937 (`.dwg` design files)
- Publicly exposed bucket: backup-and-restore98825501 (via PutBucketPublicAccessBlock)
- Rogue persistence account: marketing.mark, added to the Admins group

## What I'd do as an analyst
- Disable/rotate the helpdesk.luke credentials immediately.
- Delete the rogue marketing.mark account and audit for any other unfamiliar IAM users.
- Re-apply the public access block on backup-and-restore98825501 and audit its access logs for actual data exposure/download during the window it was public.
- Review CloudTrail for any other CreateUser/AddUserToGroup pairs across the account, since that combination is a strong persistence signal.
- Consider Service Control Policies (SCPs) to prevent any user, even a compromised admin-level one, from disabling public access blocks on critical buckets.

## Notes / things I learned
- This was my first time with Splunk/SPL — the actual investigative logic (which event name maps to which stage of a compromise) came fairly naturally once I thought about what an attacker would need to do at each step, but I needed help with correct query syntax throughout.
- CreateUser immediately followed by AddUserToGroup (especially into an admin-level group) is a strong, specific persistence signal in AWS environments — worth remembering as a pattern to alert on.
- Skill to improve: get comfortable enough with SPL syntax that I'm not needing to ask for the query structure each time — the reasoning is there, the tool fluency isn't yet.
