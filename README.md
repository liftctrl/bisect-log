# bisect-log

hwo to find bad commit

git bisect start
git bisect bad v0.11.3
git bisect good master

- v0.11.3: 0.11.3_error.mp4  state: bad

- master: master_fixed.mp4   state: good

1. - a99c469e54: a99c469e54_fixed.mp4 state: good

2. - dc87a0d80a: dc87a0d80a_fixed.mp4 state: good

3. - 36c6f488e4: 36c6f488e4_fixed.mp4 state: good

4. - db3b856779: db3b856779_fixed.mp4 state: good

5. - 282f9fb816: 282f9fb816_error.mp4 state: bad

6. - 2d3517012a: 2d3517012a_fixed.mp4 state: good

7. - 0f81af53b0: 0f81af53b0_error.mp4 state: bad

8. - 37fb09c162: 37fb09c162_fixed.mp4 state: good

9. - a80bdf0d9b: a80bdf0d9b_error.mp4 state: bad

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

how to find good commit

git bisect start
git bisect good master
git bisect bad a80bdf0d9b

- master: master_fixed.mp4 state: good

1. - a99c469e54: a99c469e54_fixed.mp4 state: good

2. - 2b2a3449f7: 2b2a3449f7_fixed.mp4 state: good

3. - 25a869a097: 25a869a097_fixed.mp4 state: good

4. - ce292026ea: ce292026ea_fixed.mp4 state: good

5. - 638bc951b2: 638bc951b2_fixed.mp4 state: good

6. - bfcf541a9e: bfcf541a9e_fixed.mp4 state: good

7. - 41ceefe804: 41ceefe804_fixed.mp4 state: good

8. - e732cbe36c: e732cbe36c_fixed.mp4 state: good

9. - 37fb09c162: 37fb09c162_fixed.mp4 state: good

- a80bdf0d9b: a80bdf0d9b_error.mp4 state: bad

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
