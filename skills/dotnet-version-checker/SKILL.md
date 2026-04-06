---
name: dotnet-version-checker
description: >
  Checks the official .NET platform support policy to determine whether a new stable .NET release
  is available for production use. Use this skill when the user asks about the latest supported
  .NET versions, release status, or whether an update is recommended based on Microsoft's
  support policy.
applyTo: ["**/*"]
triggersOn:
  - "check .NET stable release"
  - "latest .NET support policy"
  - "new stable .NET release available"
compatibility: [".NET 8+", "web lookup"]
---

# .NET Support Policy Checker

This skill verifies the current .NET support policy from the official Microsoft page:
`https://dotnet.microsoft.com/en-us/platform/support/policy`.
It determines whether a newer stable .NET release is available, whether it is production-ready,
and whether the user's project should consider upgrading based on support lifecycle and stability.

## When to Use

- The user asks if a newer stable .NET release is available now
- The user wants to know whether `.NET X` is still supported or in standard support
- The user asks whether they should upgrade to the latest stable .NET version
- The user wants the current support policy status for .NET releases

## When Not to Use

- For preview or nightly release status questions
- For questions about unsupported, deprecated, or end-of-life frameworks unrelated to .NET
- For internal release planning that requires Microsoft roadmap details beyond the public support policy

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| currentVersion | No | The user's current .NET version, if available, to compare against the latest supported release |
| focus | No | Optional focus such as `LTS`, `Current`, or `stability` |

## Workflow

### Step 1: Fetch official policy page

Visit `https://dotnet.microsoft.com/en-us/platform/support/policy` and identify the current stable releases.
Look for the latest supported `.NET` versions, their support status, and whether they are classified as `Current` or `LTS`.

### Step 2: Determine latest stable release availability

- Confirm whether a newer stable release exists beyond the user's current version.
- Verify if that release is in standard support or just in a maintenance phase.
- Note whether the release is publicly recommended for production use.

### Step 3: Compare with the user's version

If the user provides `currentVersion`, compare it to the latest stable supported release:
- If the user is already on the latest supported stable version, state that no newer stable release is currently recommended.
- If a newer supported stable release exists, explain the difference and whether the upgrade is advisable.

### Step 4: Produce a concise recommendation

Summarize the findings with:
- Latest supported stable .NET release(s)
- Support status and end-of-support timeframe
- Whether the release is appropriate for production use today
- Recommended action based on the user's current version and the policy

## Validation

- [ ] The skill references `https://dotnet.microsoft.com/en-us/platform/support/policy`
- [ ] The response clearly states whether a new stable .NET release is currently available
- [ ] The response includes whether the release is recommended for production and its support status
- [ ] If a current version is provided, the response compares it to the latest stable supported release

## Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Using outdated .NET release information | Always refresh from the official support policy page before answering |
| Confusing preview releases with stable releases | Only treat `Current` or `LTS` stable releases as production-ready, not preview/beta builds |
| Recommending an unsupported release | Verify the release is in standard support or extended support before suggesting an upgrade |
| Ignoring the policy lifecycle | Mention support phase and end-of-support date in the recommendation |
