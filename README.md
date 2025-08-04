# Neovim Treesitter Regression Testing: `git bisect` Process

## Overview

This document outlines the process I followed to identify the first bad commit causing a regression in Neovim's Treesitter highlighting functionality. The issue was traced using the `git bisect` tool, which helped pinpoint the commit that introduced the bug.

## Environment Details

- Operating System: Arch Linux 6.15.9-arch1-1
- Neovim Version: Built from source with CMAKE_BUILD_TYPE=RelWithDebInfo
- Tested repository is my own repo: https://github.com/liftctrl/nvim-highlight-error-repo
- Affected Feature: Treesitter highlighting (Lua parsing)
- Error Message: Invalid 'end_col': out of range (from nvim.treesitter.highlighter)

## Video

I recorded test results for both "bad" and "good" commit ranges:

- master_fixed.mp4
- 37fb09c162_fixed.mp4
- 0.11.3_error.mp4
- a80bdf0d9b_error.mp4

## Steps Taken

### 1. Start Bisecting to Find the First Bad Commit

The first step is to initiate the git bisect process by marking the known bad and good commits:

```bash
git bisect start
git bisect bad v0.11.3  # Known bad version
git bisect good master   # Known good version
```

Then, I tested intermediate commits, and based on the results, the bisect process continued with each step narrowing down the possible commits:


| Commit Hash | Result | Notes                  |
| ----------- | ------ | ---------------------- |
| a99c469e54  | ✅ Good | NVIM v0.11.0           |
| dc87a0d80a  | ✅ Good |                        |
| 36c6f488e4  | ✅ Good |                        |
| db3b856779  | ✅ Good |                        |
| 282f9fb816  | ❌ Bad  |                        |
| 2d3517012a  | ✅ Good |                        |
| 0f81af53b0  | ❌ Bad  |                        |
| 37fb09c162  | ✅ Good |                        |
| a80bdf0d9b  | ❌ Bad  | ➜ **First bad commit** |


The first bad commit is identified as:

```bash
a80bdf0d9bd62762938e8de6bf2cd601f0689763
fix(treesitter): ensure TSLuaTree is always immutable
```


### 2. Reversed Bisecting to Find the Last Known Good Commit

After identifying the first bad commit, I reversed the bisecting process to find the last known good commit:

```bash
git bisect start
git bisect good master       # Known good version
git bisect bad a80bdf0d9bd   # First bad commit
```

During this process, I tested the following commits:


| Commit Hash | Result | Notes                        |
| ----------- | ------ | ---------------------------- |
| master      | ✅ Good |                              |
| a99c469e54  | ✅ Good |                              |
| 2b2a3449f7  | ✅ Good |                              |
| 25a869a097  | ✅ Good |                              |
| ce292026ea  | ✅ Good |                              |
| 638bc951b2  | ✅ Good |                              |
| bfcf541a9e  | ✅ Good |                              |
| 41ceefe804  | ✅ Good |                              |
| e732cbe36c  | ✅ Good |                              |
| 37fb09c162  | ✅ Good | ➜ **Last known good commit** |


## 3. Conclusion

- First Bad Commit:
  - a80bdf0d9bd62762938e8de6bf2cd601f0689763
  - Commit message: fix(treesitter): ensure TSLuaTree is always immutable
- Last Known Good Commit:
  - 37fb09c162583fcff4350faad7e4a2bb60c60854
  - Commit message: test(treesitter): test tree:root() is idempotent

## Notes

- Build Setup: The testing was done on Neovim built from source using CMAKE_BUILD_TYPE=RelWithDebInfo.
- Test Process: Each commit was tested with consistent behavior checks to identify when the regression occurred.

## Command

### Find bad commit

```bash
[yang@arch neovim]$ git bisect start
status: waiting for both good and bad commits
[yang@arch neovim]$ git bisect bad v0.11.3
status: waiting for good commit(s), bad commit known
[yang@arch neovim]$ git bisect good master
Bisecting: a merge base must be tested
[a99c469e547fc59472d6d105c0fae323958297a1] NVIM v0.11.0
[yang@arch neovim]$ git bisect good
Bisecting: 132 revisions left to test after this (roughly 7 steps)
[dc87a0d80a2b2c37f0792d88de63036996016e24] fix(lua): vim.validate `message` param #33675
[yang@arch neovim]$ git bisect good
Bisecting: 66 revisions left to test after this (roughly 6 steps)
[36c6f488e436c385f55e4785436213c56a6abf6c] fix(terminal): fix OSC 8 parsing (#34424)
[yang@arch neovim]$ git bisect good
Bisecting: 33 revisions left to test after this (roughly 5 steps)
[db3b85677928e4157d459aebb21a673a8bc84f28] feat(defaults): map "grt" to LSP type_definition #34663
[yang@arch neovim]$ git bisect good
Bisecting: 16 revisions left to test after this (roughly 4 steps)
[282f9fb816d765cb5e6113e8035884d5ecfdb61d] fix(incsearch): include compsing characters with Ctrl-L
[yang@arch neovim]$ git bisect bad
Bisecting: 8 revisions left to test after this (roughly 3 steps)
[2d3517012a774b52bbc89670bd25e3bd767842ea] Merge #34722 from justinmk/release
[yang@arch neovim]$ git bisect good
Bisecting: 4 revisions left to test after this (roughly 2 steps)
[0f81af53b03793d11feb505f866bf162557c3ef5] build: support static build #34728
[yang@arch neovim]$ git bisect bad
Bisecting: 1 revision left to test after this (roughly 1 step)
[37fb09c162583fcff4350faad7e4a2bb60c60854] test(treesitter): test tree:root() is idempotent
[yang@arch neovim]$ git bisect good
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[a80bdf0d9bd62762938e8de6bf2cd601f0689763] fix(treesitter): ensure TSLuaTree is always immutable
[yang@arch neovim]$ git bisect bad
a80bdf0d9bd62762938e8de6bf2cd601f0689763 is the first bad commit
commit a80bdf0d9bd62762938e8de6bf2cd601f0689763 (HEAD)
Author: Rodrigodd <rodrigobatsmoraes@hotmail.com>
Date:   Thu Jun 26 19:45:28 2025 -0300

    fix(treesitter): ensure TSLuaTree is always immutable

    Problem:

    The previous fix in #34314 relies on copying the tree in `tree_root` to
    ensure the `TSNode`'s tree cannot be mutated. But that causes the
    problem where two calls to `tree_root` return nodes from different
    copies of a tree, which do not compare as equal. This has broken at
    least one plugin.

    Solution:

    Make all `TSTree`s on the Lua side always immutable, avoiding the need
    to copy the tree in `tree_root`, and make the only mutation point,
    `tree_edit`, copy the tree instead.

    (cherry picked from commit 168bf0024ece7f73b846d0b465ae7d8109bb97e0)

 runtime/lua/vim/treesitter/_meta/tstree.lua |  1 +
 runtime/lua/vim/treesitter/languagetree.lua |  4 +--
 src/nvim/lua/treesitter.c                   | 41 ++++++++++++++-----------
 3 files changed, 26 insertions(+), 20 deletions(-)
```

### Find good commit

```bash
[yang@arch neovim]$ git bisect start
status: waiting for both good and bad commits
[yang@arch neovim]$ git bisect good master
status: waiting for bad commit, 1 good commit known
[yang@arch neovim]$ git bisect bad a80bdf0d9b
Bisecting: a merge base must be tested
[a99c469e547fc59472d6d105c0fae323958297a1] NVIM v0.11.0
[yang@arch neovim]$ git bisect good
Bisecting: 121 revisions left to test after this (roughly 7 steps)
[2b2a3449f78b5ea984e012147709625e0fc78532] fix(runtime): conceal paths in help, man ToC loclist #33764
[yang@arch neovim]$ git bisect good
Bisecting: 60 revisions left to test after this (roughly 6 steps)
[25a869a0976c1925dd684e7304e8e5439b48d9e4] Merge pull request #34305
[yang@arch neovim]$ git bisect good
Bisecting: 30 revisions left to test after this (roughly 5 steps)
[ce292026ea93a277186aacbd5d69062a52824fd9] docs(meta): fix incorrect bar -> backtick replacement (#34520)
[yang@arch neovim]$ git bisect good
Bisecting: 15 revisions left to test after this (roughly 4 steps)
[638bc951b2ecac16b1c906963bb5e7370d158747] Merge pull request #34651 from zeertzjq/backport
[yang@arch neovim]$ git bisect good
Bisecting: 7 revisions left to test after this (roughly 3 steps)
[bfcf541a9edbcc22f3a545063470dbc7041569ae] fix(tutor): cannot find tutors in pack/*/start/* #34689
[yang@arch neovim]$ git bisect good
Bisecting: 3 revisions left to test after this (roughly 2 steps)
[41ceefe8047bec123f53575b28981c887e6f42e9] test(exrc): lua exrc knows its location #34713
[yang@arch neovim]$ git bisect good
Bisecting: 1 revision left to test after this (roughly 1 step)
[e732cbe36c803aaf6c9ba6d42a36b7341c6c26da] fix(vim.system): env=nil passes env=nil to uv.spawn
[yang@arch neovim]$ git bisect good
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[37fb09c162583fcff4350faad7e4a2bb60c60854] test(treesitter): test tree:root() is idempotent
[yang@arch neovim]$ git bisect good
a80bdf0d9bd62762938e8de6bf2cd601f0689763 is the first bad commit
commit a80bdf0d9bd62762938e8de6bf2cd601f0689763
Author: Rodrigodd <rodrigobatsmoraes@hotmail.com>
Date:   Thu Jun 26 19:45:28 2025 -0300

    fix(treesitter): ensure TSLuaTree is always immutable

    Problem:

    The previous fix in #34314 relies on copying the tree in `tree_root` to
    ensure the `TSNode`'s tree cannot be mutated. But that causes the
    problem where two calls to `tree_root` return nodes from different
    copies of a tree, which do not compare as equal. This has broken at
    least one plugin.

    Solution:

    Make all `TSTree`s on the Lua side always immutable, avoiding the need
    to copy the tree in `tree_root`, and make the only mutation point,
    `tree_edit`, copy the tree instead.

    (cherry picked from commit 168bf0024ece7f73b846d0b465ae7d8109bb97e0)

 runtime/lua/vim/treesitter/_meta/tstree.lua |  1 +
 runtime/lua/vim/treesitter/languagetree.lua |  4 +--
 src/nvim/lua/treesitter.c                   | 41 ++++++++++++++-----------
 3 files changed, 26 insertions(+), 20 deletions(-)
```
