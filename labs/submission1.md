# Lab 1 — Submission

## Task 1 — SSH commit signing

### Benefits of signing commits
- Authenticity: proves who created the commit.
- Integrity: helps detect tampering with commit history.
- Trust in collaboration: reviewers can rely on verified authorship.

### Why is commit signing important in DevOps workflows?
Commit signing increases supply-chain security and accountability: teams can verify that code changes come from trusted identities, which reduces risk of malicious or spoofed commits.

### Evidence: signed + verification
### Evidence

SSH authentication to GitHub was successfully verified using `ssh -T git@github.com`.

A signed commit was created using SSH signing. GitHub shows the commit as **Verified**, and local verification with `git log --show-signature -1` confirms a valid ED25519 signature.
![alt text](evidence_ssh.png)

Verification evidence:
![alt text](evidence.png)
