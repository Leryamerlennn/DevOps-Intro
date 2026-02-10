# Lab 2 — Version Control & Advanced Git (Submission)

## Task 1 — Git Object Model Exploration

### 1.1 Create sample commit
**Goal:** create a commit so we can inspect how Git stores data (commit → tree → blob).

```bash
echo "Test content" > test.txt
git add test.txt
git commit -m "Add test file"
```

Output:

```bash
[feature/lab2 69123ad] Add test file
1 file changed, 1 insertion(+)
create mode 100644 test.txt
```

### 1.2 Inspect objects (blob/tree/commit)

Commit object (HEAD): shows metadata + pointer to the project snapshot (tree) and previous commit (parent).

```bash
leryamerlen@MacBook-Pro DevOps-Intro % git log --oneline -1
69123ad (HEAD -> feature/lab2) Add test file

leryamerlen@MacBook-Pro DevOps-Intro % git cat-file -p HEAD
tree 9a587ac0f6b142a6e4d0707ee32c88eda5d8452b
parent fbb72ec7831b033708c27978147c458d570e11a5
author Valeria Neganova <v.neganova@innopolis.university> 1770715025 +0300
committer Valeria Neganova <v.neganova@innopolis.university> 1770715025 +0300
gpgsig -----BEGIN SSH SIGNATURE-----
 U1NIU0lHAAAAAQAAADMAAAALc3NoLWVkMjU1MTkAAAAgBaC9QahhF2UY9jtzPYBogv8whA
 YX35+QLfpJ2Ci8RnEAAAADZ2l0AAAAAAAAAAZzaGE1MTIAAABTAAAAC3NzaC1lZDI1NTE5
 AAAAQM5NTpY03aIe3oTGQrfzC+DSyiLMG6PnDZYs6sO+ZQqzIVHYcc40O+y1g6jriEg9Yj
 l537fv4j8+8xqABjahhgY=
 -----END SSH SIGNATURE-----

Add test file
```

Tree object: lists files/directories and points to blobs (file contents) and subtrees.

```bash
leryamerlen@MacBook-Pro DevOps-Intro % git cat-file -p 9a587ac0f6b142a6e4d0707ee32c88eda5d8452b
040000 tree 8077f2c1b2d2aebcc5970a3bcfa35cf209a95a65    .github
100644 blob 6e60bebec0724892a7c82c52183d0a7b467cb6bb    README.md
040000 tree a1061247fd38ef2a568735939f86af7b1000f83c    app
040000 tree 721498e805517d65cac5986b38361742399b7d9b    labs
040000 tree d3fb3722b7a867a83efde73c57c49b5ab3e62c63    lectures
100644 blob 2eec599a1130d2ff231309bb776d1989b97c6ab2    test.txt
```

Blob object: stores the actual content of a file
```bash
leryamerlen@MacBook-Pro DevOps-Intro % git cat-file -p 2eec599a1130d2ff231309bb776d1989b97c6ab2
Test content
```

#### Short explanation:

commit: metadata + points to tree (snapshot) + parent (previous commit)
tree: directory listing (filenames + modes) and links to blob/sub-tree
blob: raw file content

## Task 2 — Reset and Reflog Recovery

### 2.1 Practice branch commits
```bash
leryamerlen@MacBook-Pro DevOps-Intro % git switch -c git-reset-practice
echo "First commit" > file.txt
git add file.txt
git commit -m "First commit"

echo "Second commit" >> file.txt
git add file.txt
git commit -m "Second commit"

echo "Third commit" >> file.txt
git add file.txt
git commit -m "Third commit"

Switched to a new branch 'git-reset-practice'
[git-reset-practice 5ac6e34] First commit
 1 file changed, 1 insertion(+)
 create mode 100644 file.txt
[git-reset-practice 84fade2] Second commit
 1 file changed, 1 insertion(+)
[git-reset-practice e5108d7] Third commit
 1 file changed, 1 insertion(+)
leryamerlen@MacBook-Pro DevOps-Intro % git log --oneline --decorate -5

e5108d7 (HEAD -> git-reset-practice) Third commit
84fade2 Second commit
5ac6e34 First commit
69123ad (feature/lab2) Add test file
fbb72ec (origin/main, origin/HEAD, main) chore: add pull request template
```


### 2.2 Reset modes + reflog recovery

`git reset --soft`

```bash
leryamerlen@MacBook-Pro DevOps-Intro % git reset --soft HEAD~1
git status
git log --oneline -5

On branch git-reset-practice
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   file.txt


84fade2 (HEAD -> git-reset-practice) Second commit
5ac6e34 First commit
69123ad (feature/lab2) Add test file
fbb72ec (origin/main, origin/HEAD, main) chore: add pull request template
0104dcc docs: add commit signing summary
```
Result:
1) `HEAD` moved back by 1 commit (Third → Second)
2) Changes from the removed commit stayed **staged**
3) Working tree unchanged; index still contains changes from the removed commit


`git reset --hard`

```bash
leryamerlen@MacBook-Pro DevOps-Intro % git reset --hard HEAD~1
git status
git log --oneline -5

HEAD is now at 5ac6e34 First commit
On branch git-reset-practice

5ac6e34 (HEAD -> git-reset-practice) First commit
69123ad (feature/lab2) Add test file
fbb72ec (origin/main, origin/HEAD, main) chore: add pull request template
0104dcc docs: add commit signing summary
d6b6a03 Update lab2
```

Result:
1) `--hard` moved `HEAD` back and reset **index + working tree** to match the target commit
2) Changes from the removed commit were discarded from staging area and working directory (can be recovered via reflog)

`git reflog`

```bash
leryamerlen@MacBook-Pro DevOps-Intro % git reflog -10

5ac6e34 (HEAD -> git-reset-practice) HEAD@{0}: reset: moving to HEAD~1
84fade2 HEAD@{1}: reset: moving to HEAD~1
e5108d7 HEAD@{2}: commit: Third commit
84fade2 HEAD@{3}: commit: Second commit
5ac6e34 (HEAD -> git-reset-practice) HEAD@{4}: commit: First commit
69123ad (feature/lab2) HEAD@{5}: checkout: moving from feature/lab2 to git-reset-practice
69123ad (feature/lab2) HEAD@{6}: commit: Add test file
fbb72ec (origin/main, origin/HEAD, main) HEAD@{7}: checkout: moving from main to feature/lab2
fbb72ec (origin/main, origin/HEAD, main) HEAD@{8}: clone: from https://github.com/Leryamerlennn/DevOps-Intro.git
leryamerlen@MacBook-Pro DevOps-Intro % git reset --hard e5108d7
git log --oneline --decorate -5

HEAD is now at e5108d7 Third commit
e5108d7 (HEAD -> git-reset-practice) Third commit
84fade2 Second commit
5ac6e34 First commit
69123ad (feature/lab2) Add test file
fbb72ec (origin/main, origin/HEAD, main) chore: add pull request template
```

Result:
1) `git reflog` showed the previous `HEAD` states and the “lost” commit hash (`e5108d7`)
2) `git reset --hard e5108d7` restored the branch to the Third commit

## Task 3 — Visualize Commit History

Commands:
```bash
git switch -c side-branch
echo "Branch commit" >> history.txt
git add history.txt
git commit -m "Side branch commit"
git switch -
git log --oneline --graph --all --decorate
```

Result:
```bash
* a90bddc (side-branch) Side branch commit
* e5108d7 (HEAD -> git-reset-practice) Third commit
* 84fade2 Second commit
* 5ac6e34 First commit
* 69123ad (feature/lab2) Add test file
* fbb72ec (origin/main, origin/HEAD, main) chore: add pull request template
...
```

Result:
1) Created a new branch `side-branch` and made a commit `a90bddc`.
2) `git log --graph --all --decorate` visualized the commit history across all branches and showed branch pointers (e.g., `side-branch`, `feature/lab2`, `main`) and current `HEAD`.

## Task 4 — Tagging Commits

Commands:
```bash
leryamerlen@MacBook-Pro DevOps-Intro % git tag v1.0.0
git show-ref --tags
git push origin v1.0.0

69123ad8fe802de0a31b28c659aea4987ad783ce refs/tags/v1.0.0
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 12 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 520 bytes | 520.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/Leryamerlennn/DevOps-Intro.git
 * [new tag]         v1.0.0 -> v1.0.0
```

Result:

1) Created a lightweight tag v1.0.0 pointing to commit 69123ad8fe802de0a31b28c659aea4987ad783ce.
2) Pushed the tag to the remote repository (origin).

Why tags are useful:
Tags mark specific commits as releases/versions (e.g., v1.0.0) and make them easy to reference in CI/CD and GitHub Releases

## Task 5 — switch vs checkout vs restore

`git switch`

```bash
leryamerlen@MacBook-Pro DevOps-Intro % git switch -c cmd-compare
git switch -
git branch

Switched to a new branch 'cmd-compare'
Switched to branch 'feature/lab2'
  cmd-compare
* feature/lab2
  git-reset-practice
  main
  side-branch
```

`git checkout`

```bash
leryamerlen@MacBook-Pro DevOps-Intro % git checkout -b cmd-compare-2
git checkout -
git branch

Switched to a new branch 'cmd-compare-2'
Switched to branch 'feature/lab2'
  cmd-compare
  cmd-compare-2
* feature/lab2
  git-reset-practice
  main
  side-branch
```

`git restore`

Commands:
```bash
# (prepare tracked file for restore demo)
echo "base" > demo.txt
git add demo.txt
git commit -m "chore: add demo.txt for restore demo"

echo "scratch" >> demo.txt
git status

git restore demo.txt
git status

echo "scratch" >> demo.txt
git add demo.txt
git status

git restore --staged demo.txt
git status
```
Output:

```bash
# after editing demo.txt (not staged)
Changes not staged for commit:
        modified:   demo.txt

# after `git restore demo.txt`
nothing added to commit but untracked files present

# after `git add demo.txt`
Changes to be committed:
        modified:   demo.txt

# after `git restore --staged demo.txt`
Changes not staged for commit:
        modified:   demo.txt

```

Result:

1) git restore <file> discards local changes in working tree (restores file from HEAD).
2) git restore --staged <file> removes the file from staging area (keeps the content in working tree).
3) `git status` confirms the file state transition between working tree and staging area.


## Task 6 — GitHub Community

Actions:
- Starred the course repository and `simple-container-com/api`.
- Followed users: `@Cre-eD`, `@marat-biriushev`, `@pierrepicaud`.
- Followed at `iJeannne`, `Ily17as`, `KosmonikOS`

Why it matters:
- Stars are used to bookmark projects and signal interest/quality to maintainers.
- Following helps track people’s activity and makes collaboration easier.
