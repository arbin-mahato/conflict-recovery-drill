# Pull Request: Repository Recovery and Feature Integration

## What Was Broken

The repository suffered from several critical failures that made it undeployable:

- **Corrupted Integration**: Commit `0565dcf` on the integration branch contained raw Git conflict markers (`<<<<<<<`, `=======`), which would cause the application to crash.
- **Unfinished Work Merged**: Commit `d050dd7` was a "WIP" commit with unresolved conflicts, indicating a breakdown in branch discipline.
- **Divergent Feature Logic**: Both `feature/discounts` and `feature/tax-v2` modified the same block in `checkout.js`. Without coordination, merging either would overwrite the other's logic.
- **Drifting Branches**: Feature branches were developed in isolation for too long, leading to a complex multi-way conflict during integration.

## Recovery Actions Taken

I followed a "Clean Reconstruction" strategy to restore the repository to a stable state:

1. **Branch Isolation**: Created the `recovery/main-fix` branch starting from commit `0fbf8d4` (the last known stable point before the integration mess).
2. **Sequential Merging**: Merged `feature/discounts` and then `feature/tax-v2` to identify the exact point of logical collision.
3. **Manual Conflict Resolution**: Instead of choosing one feature over the other, I manually combined the logic in `checkout.js` to support both business requirements.
4. **Selective Integration**: Used `git cherry-pick` to extract the critical rounding fix from the `bugfix/rounding` branch. I chose `cherry-pick` over `merge` here to avoid pulling in any other potentially unstable changes from that branch, as we are close to the release deadline.

## Conflict Resolution Decisions

- **File**: `checkout.js`
- **Function**: `checkout(items)`
- **Conflict**: The `discounts` feature introduced a `total * 0.9` operation, while the `tax-v2` feature introduced a `total * 1.05` operation.
- **Decision**: I integrated both. The final code now correctly applies the 10% discount first, then the 5% tax, and finally rounds the result to two decimal places. This ensures that customers get their discounts while the company remains tax-compliant.

## Repository State After Recovery

- **Deployability**: The `main` branch (via this PR) is now 100% deployable. All conflict markers have been removed.
- **Verification**: A new `test.js` script has been added which validates the calculation ($10.00 base price results in a final total of $9.45).
- **History**: The commit history is now readable, with each step of the recovery documented with descriptive messages.
- **Open Branches**: Branches like `integration` remain open but are flagged as "unstable" and should be deleted once this recovery is merged to `main`.
