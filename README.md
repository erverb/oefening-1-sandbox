# Submission workflow sandbox

This is a disposable target repository for the submission processor in the
sibling `oefening-1` controller repository. Its eight identifiers and all names,
PDFs, and commits are synthetic; it contains no copied student submissions.
There is no processing workflow in this repository: the controller operates on
this target from its own repository.

`main` contains the untouched `studenten.txt` roster and `teamA` through `teamK`.
Seven independent `fixtures/*` branches start from that same initial commit.
Each sample commit is authored by `Exercise sandbox <sandbox@example.invalid>`.
The machine-readable branch references and expected results are in
[`fixtures/scenarios.json`](fixtures/scenarios.json).

## Run the complete local demonstration

Install the controller dependencies as described in its README, then run from
the controller repository:

```bash
python scripts/local_smoke.py --sandbox ../oefening-1-sandbox
```

The command runs the actual batch processor in `report` and `process` modes
against a temporary local bare origin. A fake GitHub API supplies open fork-PR
metadata and successful checks. It verifies that reporting makes no remote
changes, that processing merges Alice and Bob consecutively without losing
either adjacent roster edit, and that invalid submissions stay untouched.
Reports and a summary are saved under `oefening-1/artifacts/local-smoke/`.
The sandbox's branches, files, index, and working tree stay unchanged. No token,
network access, or GitHub repository is required.

This validates local processor behavior. It simulates the GitHub API; testing
the actual Actions runner, token permissions, branch policies, and GitHub PR
merge recognition requires the separate GitHub trial below.

## Fixture branches and expected batch results

Open all seven scenarios together to reproduce this table. Alice and Bob edit
adjacent roster rows independently, exercising the processor's merge logic.

| Branch (`fixtures/` prefix) | Synthetic identifier | Change | Report | Process |
| --- | --- | --- | --- | --- |
| `valid-alice` | `r0000001` | Alice + valid PDF in teamA | `geschikt` | `verwerkt` |
| `valid-bob` | `r0000002` | Bob + valid PDF in teamB | `geschikt` | `verwerkt` |
| `missing-pdf` | `r0000003` | Carol, PDF absent | `nakijken` | `nakijken` |
| `extra-file` | `r0000004` | Dan + PDF + unexpected notes.txt | `nakijken` | `nakijken` |
| `duplicate-eve` | `r0000005` | Eve + valid PDF, duplicate ID claim | `nakijken` | `nakijken` |
| `duplicate-fay` | `r0000005` | Fay + valid PDF, duplicate ID claim | `nakijken` | `nakijken` |
| `invalid-pdf` | `r0000006` | Gail + text disguised as PDF | `nakijken` | `nakijken` |

Eve and Fay are individually valid; opening both together is what exercises
duplicate-claim detection. The last two roster identifiers remain unused.
After processing the full batch, `main` must contain Alice and Bob followed by
the other six original identifiers, and only their two new PDFs.

## Later: try the workflow on GitHub

Only local repositories have been prepared. When ready, create an empty GitHub
sandbox repository and push **only `main`** to it:

```bash
git remote add upstream https://github.com/SANDBOX_OWNER/oefening-1-sandbox.git
git push -u upstream main
```

Create a GitHub fork under a separate test account (or another eligible owner)
and add that fork as a second remote. Push the sample branches to the fork:

```bash
git remote add fork https://github.com/FORK_OWNER/oefening-1-sandbox.git
git push fork 'refs/heads/fixtures/*:refs/heads/fixtures/*'
```

Open a PR for each sample branch **from the fork into the upstream sandbox's
`main`**. The validator explicitly requires fork submissions; PRs from branches
inside the upstream repository are rejected. Keep `duplicate-eve` and
`duplicate-fay` open at the same time.

Publish and configure the controller as described in `../oefening-1/README.md`,
select this sandbox as its target, and run `report` first. Check the table above,
then run `process` and inspect the resulting roster, PDFs, merge commits, and
remaining open PRs. Use a fresh disposable sandbox for another identical trial
after successful processing. Target the real `test` repository only after this
trial succeeds.
