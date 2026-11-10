# Lab 02: Governance: Management Groups, Policy, Tags, Locks and Costs

**Status:** In progress (Tasks 1–3 done; Task 4a done; 4b in progress) · **Date(s):** Oct 26, Nov 1 and Nov 9, 2026– · **Time spent:** - · **Cost (Cost analysis):** -

Lab guide: [labs/02-governance-policy-locks-costs.md](../labs/02-governance-policy-locks-costs.md) · Screenshots: `screenshots/lab-02/`

## What I built
<!-- 3-5 bullets in my own words, written after finishing the lab. -->
-

## Evidence
<!-- When a screenshot is committed, replace its path with an embed, e.g.
     ![Checkpoint 02-01: what it proves](../screenshots/lab-02/02-01-name.png)
     and set Status to Done. Redact first: docs/screenshot-guide.md -->
| Checkpoint | What it proves | Screenshot | Status |
|---|---|---|---|
| 02-01 | The expanded MG tree with your subscription under `mg-hlg-landingzones` | ![Checkpoint 02-01: mg-hlg with landingzones, platform and sandbox; subscription under landingzones](../screenshots/lab-02/02-01-mg-hierarchy.png) | Done |
| 02-02 | Resource providers filtered to Microsoft.App, plus Usage + quotas | ![Checkpoint 02-02a: subscription renamed to Halsted Logistics - V1, parent MG mg-hlg-landingzones](../screenshots/lab-02/02-02a-subscription-renamed.png) ![Checkpoint 02-02: Microsoft.App registered](../screenshots/lab-02/02-02-subscription-settings.png) ![Checkpoint 02-02b: Usage + quotas, Compute: per-region vCPU limits by VM family](../screenshots/lab-02/02-02b-usage-quotas.png) | Done |
| 02-03 | Resource groups list showing the tag columns | ![Checkpoint 02-03a: 7 resource groups across 4 regions created by one Bash loop](../screenshots/lab-02/02-03a-rgs-created-cli.png) ![Checkpoint 02-03c: a wrapped line split the storage command](../screenshots/lab-02/02-03c-wrapped-line-error.png) ![Checkpoint 02-03b: sandbox RG and storage account created after the fixes](../screenshots/lab-02/02-03b-sandbox-storage-cli.png) ![Checkpoint 02-03: all 8 RGs carry the 5 standard tags](../screenshots/lab-02/02-03-rg-tags.png) ![Checkpoint 02-03d: move sthlgsbx0808 from rg-hlg-sandbox to rg-hlg-mgmt-eus](../screenshots/lab-02/02-03d-move-target.png) ![Checkpoint 02-03e: move validation Succeeded](../screenshots/lab-02/02-03e-move-validation.png) ![Checkpoint 02-03f: storage account during the move](../screenshots/lab-02/02-03f-moved-to-mgmt.png) ![Checkpoint 02-03g: moved back to rg-hlg-sandbox, Activity log shows both moves](../screenshots/lab-02/02-03g-moved-back.png) ![Checkpoint 02-03h: DataClassification=Internal on the storage account (portal, one resource)](../screenshots/lab-02/02-03h-storage-tag.png) ![Checkpoint 02-03i: Region-Owner=IT-Chicago on all 8 rg-hlg-* groups (portal, bulk Assign tags)](../screenshots/lab-02/02-03i-region-owner-bulk.png) ![Checkpoint 02-03j: az tag update --operation Merge adds Purpose without touching the other tags](../screenshots/lab-02/02-03j-tag-merge-cli.png) ![Checkpoint 02-03k: one Bash loop removed Region-Owner from all 8 RGs; sandbox back to the 5 standard tags](../screenshots/lab-02/02-03k-tag-cleanup-cli.png) | Done |
| 02-04 | (a) the RequestDisallowedByPolicy error, (b) the initiative assignment's compliance page, (c) `rg-hlg-untagged` auto-tagged by Modify | ![Checkpoint 02-04a: storage account in Brazil South denied by Allowed locations, but the resource group got through](../screenshots/lab-02/02-04a-rg-allowed-storage-denied.png) ![Checkpoint 02-04a: both location policies assigned at mg-hlg (Halsted Logistics)](../screenshots/lab-02/02-04a-policy-assignments.png) ![Checkpoint 02-04a: the RG policy's enforcement mode was Enroll, fixed to Default from the CLI](../screenshots/lab-02/02-04a-enforcement-enroll-fix.png) ![Checkpoint 02-04b: TEMP Require a tag on resource groups (CostCenter) at mg-hlg, enforcement Default](../screenshots/lab-02/02-04b-temp-require-tag-assign.png) ![Checkpoint 02-04b: rg-hlg-untagged denied, RequestDisallowedByPolicy with tags[CostCenter] Exists = false](../screenshots/lab-02/02-04b-untagged-rg-denied.png) ![Checkpoint 02-04a: after the fix, rg-hlg-test-brazil3 denied by HLG - Allowed locations for RGs with the custom non-compliance message](../screenshots/lab-02/02-04a-rg-brazil3-denied.png) ![Checkpoint 02-04b: each policy's tag parameters mapped to the initiative parameters tagName and tagValue](../screenshots/lab-02/02-04b-initiative-param-mapping.png) ![Checkpoint 02-04b: HLG - Tagging standard initiative: Require (Deny), Add (Modify), Inherit (Modify)](../screenshots/lab-02/02-04b-initiative-created.png) ![Checkpoint 02-04b: initiative assigned at mg-hlg, enforcement Default, system-assigned managed identity in East US](../screenshots/lab-02/02-04b-initiative-assign-review.png) | Partial: 4a done; 4b initiative assigned, Modify test + remediation task next; 4c to do |
| 02-05 | The *Access keys* error caused by the ReadOnly lock | `screenshots/lab-02/02-05-readonly-lock.png` | Pending |
| 02-06 | Both budgets, plus the Advisor configuration page | `screenshots/lab-02/02-06-budgets-advisor.png` | Pending |

## Commands I want to remember
```bash
# Create a tagged resource group (region is required)
az group create -n rg-hlg-sandbox -l eastus --tags Project=az104-hybrid "Company=Halsted Logistics" CostCenter=CC-1001 Environment=lab Owner=ryan-holt -o table

# List only this project's groups
az group list --query "[?tags.Project=='az104-hybrid'].{Name:name, Region:location}" -o table

# Storage account with a random, unique name
SBX=sthlgsbx$RANDOM; az storage account create -g rg-hlg-sandbox -n $SBX -l eastus --sku Standard_LRS -o none; echo $SBX

# Tags: Merge adds, Delete removes one, Replace wipes everything else (don't use it here)
ID=$(az group show -n rg-hlg-sandbox --query id -o tsv)
az tag update --resource-id $ID --operation Merge --tags Purpose=experiments
az tag list --resource-id $ID -o jsonc
az tag update --resource-id $ID --operation Delete --tags Purpose=experiments

# Find every RG with a tag, then strip it from all of them in one loop
az group list --query '[?tags."Region-Owner"].name' -o tsv
for g in $(az group list --query '[?tags."Region-Owner"].name' -o tsv); do az tag update --resource-id $(az group show -n $g --query id -o tsv) --operation Delete --tags Region-Owner=IT-Chicago -o none; echo "cleared $g"; done

# Inspect policy assignments at the MG where they were made (searching from the subscription returned [])
MG=/providers/Microsoft.Management/managementGroups/mg-hlg
az policy assignment list --scope $MG --query "[].{name:displayName, enforcement:enforcementMode, definition:policyDefinitionId, locations:parameters.listOfAllowedLocations.value}" -o jsonc

# Which MG is the subscription under?
az account management-group entities list --query "[?type=='/subscriptions'].{sub:displayName, parent:parent.id}" -o table

# Turn enforcement on (the CLI needs the assignment's internal name, not its display name)
N=$(az policy assignment list --scope $MG --query "[?displayName=='HLG - Allowed locations for RGs'].name" -o tsv)
az policy assignment update --name $N --scope $MG --enforcement-mode Default
```

## What broke and how I fixed it
- **Symptom:** `argument --sku: expected one argument`, then `bash: Standard_LRS: command not found`.
  **Cause:** The command wrapped onto two lines when I copied it from the lab guide. In Bash, a line break ends a command unless the line ends with `\`, so `Standard_LRS` ran as its own command.
  **Fix:** Pasted the command as one line.
- **Symptom:** `ResourceGroupNotFound` for `rg-hlg-sandbox`.
  **Cause:** The sandbox group is a separate step from the loop, and I hadn't created it yet. My first `az group create` also failed because I left out the region (`-l`).
  **Fix:** `az group create -n rg-hlg-sandbox -l eastus --tags ...`
- **Symptom:** The storage account name came out as `sthlgsbx/bin/bash808`.
  **Cause:** I typed `$0808`. `$` means "insert a variable," and `$0` is the shell's own name (`/bin/bash`).
  **Fix:** Used plain digits (`sthlgsbx0808`) or `$RANDOM`. Storage account names are 3–24 characters, lowercase letters and numbers only, and globally unique.
- **Symptom:** With *Allowed locations* assigned, the storage account in Brazil South was denied, but `rg-hlg-test-brazil` was created in Brazil South anyway.
  **Cause:** The built-in *Allowed locations* policy deliberately skips resource groups. I had only assigned that one.
  **Fix:** Also assigned *Allowed locations for resource groups* at `mg-hlg`. Policies go on a scope (the top management group), not per region or per resource group. The 4 regions are the allowed list.
- **Symptom:** With both policies assigned at `mg-hlg` for a day, `rg-hlg-test-brazil2` (Brazil South) was still created.
  **Cause:** The RG policy's enforcement mode was `Enroll` (a preview option on the Basics tab), not `Default`. The definition, scope and regions were all correct. Listing assignments from the subscription with `--disable-scope-strict-match` returned `[]`. Querying the MG with `--scope` showed the setting.
  **Fix:** `az policy assignment update --enforcement-mode Default`. About 20 minutes later, `rg-hlg-test-brazil3` was denied with my non-compliance message. Lesson: when a policy is assigned but doesn't block, check enforcement mode first. Disabled (DoNotEnforce) still evaluates and reports, but denies nothing.

- **Symptom:** `Invalid JSON` on an initiative parameter, then "CostCenter is unused" with no Delete option.
  **Cause:** I typed `[CC-1001]` in Allowed Values (JSON strings need quotes: `["CC-1001"]`) and named the parameter after its value. The Name is the variable (`tagName`); the Default Value is what it holds (`CostCenter`).
  **Fix:** Left Allowed Values empty and created `tagName` and `tagValue`. The portal dropped the unused parameter when the initiative was saved.

- **Symptom:** After building the initiative, `rg-hlg-untagged` was created with `"tags": null`. My fix command then failed with `argument -n/--name: expected one argument`.
  **Cause:** I had created the initiative *definition* but never *assigned* it, so neither Modify nor Deny ran. `$N` was empty because no assignment had that name. I also ran the fix before the diagnosis.
  **Fix:** Assigned the initiative at `mg-hlg` with a system-assigned managed identity. Lessons: a definition does nothing until it's assigned; `echo "N=[$N]"` before using a variable; diagnose first, then fix. The Remediation tab said remediation wasn't supported during assignment, so the remediation task gets created afterward from Policy → Remediation.

## Exam tips in my own words
-

## Knowledge check score
- __ / 5 first try. Misses:

## Questions to follow up
-