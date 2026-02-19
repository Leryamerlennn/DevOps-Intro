# Lab 3 — CI/CD with GitHub Actions (Submission)

Platform: **GitHub Actions**

## Task 1 — First GitHub Actions Workflow (push trigger)

### What I did
- Created workflow file: `.github/workflows/lab3.yml`
- Configured trigger: `push` on branch `feature/lab3`
- Workflow contains one job running on `ubuntu-latest` with steps:
  - print event/ref/sha
  - checkout repo
  - list files

### Evidence
- Workflow run link : https://github.com/Leryamerlennn/DevOps-Intro/actions/runs/22189505222

### Key concepts / observations
- Workflow is defined as YAML under `.github/workflows/`.
- A workflow run consists of **jobs**, each job consists of **steps**.
- Runner used: `ubuntu-latest`.
- The run was triggered by a `push` event to `feature/lab3`.

### What caused the run to trigger
The workflow run was triggered automatically by a **push** event to the `feature/lab3` branch (a commit with the workflow file was pushed to GitHub).

### Log snippets (short)
```text
Event: push
Ref: refs/heads/feature/lab3
SHA: a375305...

Run actions/checkout@v4
Syncing repository: Leryamerlennn/DevOps-Intro

Run pwd
/home/runner/work/DevOps-Intro/DevOps-Intro
drwxr-xr-x ... .github
-rw-r--r-- ... README.md
```

### Analysis of workflow execution process
GitHub Actions detected the push event for `feature/lab3`, allocated an `ubuntu-latest` runner, and executed the single job. The job ran steps sequentially: printed context variables, checked out the repository with `actions/checkout@v4`, and listed files to confirm the workspace contents.


## Task 2 — Manual trigger + System information

### What I changed
- Added `workflow_dispatch` to allow manual runs.
- Added a step that prints runner system information (OS/CPU/RAM/disk).

### Evidence
- Workflow run link : https://github.com/Leryamerlennn/DevOps-Intro/actions/runs/22191096949

### What caused the run to trigger
This workflow run was started manually from the GitHub Actions UI using the `workflow_dispatch` trigger on the `main` branch.

### Collected runner information 

```text
== OS / Kernel ==
Linux runnervmieams 6.14.0-1017-azure #17~24.04.1-Ubuntu SMP Mon Dec  1 20:10:50 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux

== CPU ==
4
Architecture:                            x86_64
CPU op-mode(s):                          32-bit, 64-bit
Address sizes:                           48 bits physical, 48 bits virtual
Byte Order:                              Little Endian
CPU(s):                                  4
On-line CPU(s) list:                     0-3
Vendor ID:                               AuthenticAMD
Model name:                              AMD EPYC 7763 64-Core Processor
Thread(s) per core:                      2
Core(s) per socket:                      2
Socket(s):                               1
Virtualization:                          AMD-V
Hypervisor vendor:                       Microsoft
Virtualization type:                     full
L1d cache:                               64 KiB (2 instances)
L2 cache:                                1 MiB (2 instances)
L3 cache:                                32 MiB (1 instance)

== Memory ==
               total        used        free      shared  buff/cache   available
Mem:            15Gi       892Mi        13Gi        39Mi       1.8Gi        14Gi
Swap:          3.0Gi          0B       3.0Gi

== Disk ==
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       145G   53G   92G  37% /
tmpfs           7.9G   84K  7.9G   1% /dev/shm
tmpfs           3.2G 1008K  3.2G   1% /run
/dev/sda16      881M   63M  757M   8% /boot
/dev/sda15      105M  6.2M   99M   6% /boot/efi

```

### Comparison: push vs manual trigger
- Push trigger: runs automatically on each commit pushed to the target branch.
- Manual trigger: can be started from GitHub UI (“Run workflow”), selecting the branch to run against.

### Analysis of workflow execution process (manual run)
The manual trigger runs the same job as the push trigger, but it is initiated explicitly by the user via the UI. The system information step confirms the runner environment (kernel/CPU/RAM/disk) used during execution.
