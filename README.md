# GIT-commands-experiences
Continuation from https://github.com/Aleksandar15/GIT-reminds-commands, except in here I will only keep my personal experiences and clarifications of some of my confusion using GIT commands.

---
#### Extra experiences:
1. New experience (***keep reading to the end as there was issues***): on my Server Cars Club I wanted to push changes into a new branch "feature-2", but instead I pushed them at "feature-1" then ran `git checkout -b feature-2` to create that one and `git push origin feature-2` (because commits were 'merged' into my new branch) and then I merged this PR for "feature-2" but **not** "feature-1", but, *logically indeed*, the "feature-1" showed the same amount of "*commits behind*" as the merged PR "feature-2" whereas my initial goal was to leave "feature-1" as-is as if a backup to go back in time, but it was already too late, so as a fix I had to run commands: `git switch feature-1` ~then `git reset HEAD~31`~ (31 was the amount of new commits ahead - prior to the "feature-2"'s PR merge (which, *again*, had exactly same commits as "feature-1")) then `git push origin feature-1 -f`.
    - And an issue: I wasn't able to switch back to "feature-2" because the "feature-1" had (31) '*local changes for commit*', but a *wrong attempt* `git pull origin feature-1` **didn't work** because it says '*Already up to date*' and that's *not logically true*: my local directory files doesn't match the ''origin feature-1'' but it's *technically true* as my local repository matches my "origin feature-1" because I had run the default `git reset` flagless meaning `--mixed` flag was used so my local changes didn't go back 31 commits behind and `git switch` is now asking me to re-`commit` or `git stash` those "*local changes*" before switching as to not overwrite them. The solution was running `git reset--hard` flag but I needed a way to go back/reverse those 31 commits first, as to **not** go back 62 ~commits~ HEADs (thinking back from now: I'd have went 61 commits aback because there was one HEAD 'switching' (caused from the previous `reset` command) in those 62 HEADs.):
      - (Thinking back again, `get reset HEAD~32` should have done it without the `git reflog` solution below.):
      - Yet another silly issue where ChatGPT tricked me: tried to go forward '*31 commits ahead*' by a wrong command `git reset HEAD~31^` (and `git reset HEAD^31` alike), well that's *logically **impossible*** -> once HEAD is RESET there's no HEADs above it; so I instead had ran `git reflog` and copied the hash where initial 'switch from feature-2 to feature-1' happened & ran `git reset --hard abc123` and then ran the **correct** command:
      - **Correct**: `git reset --hard HEAD~31`; now that my "feature-1" PR was already merged (*from the story above*), running `git push origin feature-1 -f` was not necessary because the message is 'Everything up-to-date' but otherwise in the future for such a case I **must** run that command afterwards, and once PR is merged in GitHub: running test command `git pull origin feature-1` shows 'Already up to date' & I can switch freely to `git switch feature-2`.
- Additionally, (good info👍) while I am inside the branch "**feature-2**" running a command like `git push origin feature-1` the command will still safely **push** the changes inside of the "**feature-1**" branch & its own local repository (AKA "_`git commit`-ed_") files without any problems nor negative interaction with the current "***feature-2***" branch's local repository.
    - EXPLANATIONS: By "its own local repository" I meant 'its own commits', because no *local branch* has "its own repository".
2.
- Potential issues with Forks is when I tried to Sync *`main`* branch with the original repo's *`main`* branch, in a case where I didn't use the "*magical*" **Sync** button offered by GitHub.com (the "magic" is revealed <a href="https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/syncing-a-fork#syncing-a-fork-branch-from-the-command-line">here</a>), *which was previously called <a href="https://i.stack.imgur.com/NKEzt.png">Fetch and merge</a> button*, is that when I actually merged those *X commit(s) behind* from the GitHub website at my fork https://github.com/Aleksandar15/<fork-name\> -> my *`main`* branch became *X commit(s) ahead* - which was the most confusing part because all the files match as the <original-repo\>, so **0 files changes**, but turns out, *under the hood* what happens is that *merge* came from the <original-repo\> so my <fork-repo\> got one (*could been more*) extra merge and changed its *HEAD* to be 1 HEAD(s) above the *upstream/main*.
  - *NOTE*: For the below solution to work I have to set a *upstream* (or any name) "*remote*", which is mentioned in the link above at the "*magic is revealed*" <a href="https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/configuring-a-remote-repository-for-a-fork">part</a>:
    - `git remote -v` to view all of my current *remote*s; `-v` flag for the links.
    - `git remote add upstream <link>` where <link\> is `https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git`.
      - *Note*: the `.git` file extension at the end is not necessary, but it's a good convention to follow.
    - `git remote remove <remote-name>` to remove a remote link.
  - To fix this issue, and return this *X commit(s) ahead* back to *Synced*, there's a great answer I found <a href="https://github.com/orgs/community/discussions/22440#discussioncomment-3236721">here</a>:
  ```
  git checkout main
  git fetch upstream
  git reset --hard upstream/main
  git push --force
    ```
    - Which essentially switches HEAD: *HEAD is now at #hash-here Commit message here (#PR-number)*
      - And now running `git status` shows:
        ```
        On branch main
        Your branch is behind 'origin/main' by 1 commit, and can be fast-forwarded.   
        (use "git pull" to update your local branch)
        nothing to commit, working tree clean
         ```
          - *NOTE*: **don't** run `git pull` because that's not the *goal* here.
       - Whereas running the normal `git push` **without** a `--force` flag returns an error:
         ```
         To https://github.com/Aleksandar15/react.dev.git
         ! [rejected]          main -> main (non-fast-forward)
         error: failed to push some refs to 'https://github.com/Aleksandar15/react.dev.git'
         hint: Updates were rejected because the tip of your current branch is behind
         hint: its remote counterpart. Integrate the remote changes (e.g.
         hint: 'git pull ...') before pushing again.
         hint: See the 'Note about fast-forwards' in 'git push --help' for details.
         ```
         - So, running the apropriate command (*for this case scenario*) `git push --force` returns a success along the lines of:
           ```
           Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
           To https://github.com/Aleksandar15/react.dev.git
            + d412a7fb...cdc99178 main -> main (forced update)
           ```
             - *Note*: The shorthand flag `-f` works as well.
              - And my Fork at GitHub website says: "*This branch is up to date with reactjs/react.dev:main.*".
  - An ***important notice*** here is that while the opened PR & its branch would stay as "*X commit(s) behind*" that is totally fine in a sense that it's not the main branch, so the only issue would be if that branch has clashing changes AKA conflicting merge files in those "*X commit(s) behind*", then it would be an issue, otherwise there's not much you can do about your *branched off* branch without modifying your local changes - back to original (by *git pull*) which would ruin the purpose of this "*feature branch*"'s goal to improve something; at that point I'd have to re-write all the changes I previously *pushed*  which is not the *goal*.
    - And as expected, even my testings are failing when I run `git fetch upstream ; git checkout feature-branch ; git merge upstream/feature-branch` I get "*merge: upstream/improve-challenge-explanations - not something we can merge*" (*again, these are commands of <a href="https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/syncing-a-fork#syncing-a-fork-branch-from-the-command-line">behind the magic of Sync</a>*). 
      - *Note*: that's a same error I got when I tried to run the `git merge upstream/main` but the issue was I hadn't set up an "*upstream*" remote link, this time I do have, but fast-forwarding is not possible to a branch* as answered in <a href="https://community.atlassian.com/t5/Bitbucket-questions/What-happens-if-I-merge-a-PR-that-is-out-of-sync-from-master/qaq-p/1291041#M51499">this</a> article.
        - *As to why *fast-rowarding* is not possible on a branched off branch in which I *have* local file changes (*not matching the Upstream's repo's "main" branch* even though they were *staged* and already *pushed* onto my "*improve-challenge-explanatons*" branch, but **not yet merged** ((but at that point I should at least get merging conflicts error, instead of the error above, so the following second reason seems more to be the actual issue)), and on top of it, *commits* which don't exist in the *upstream*'s **Git**'s *main* branch, a combination of both doesn't allow *Fast-forwarding*.
          - It seem very likely, like a **scenario** where I'd need to run `git pull` (a combo of `get fetch`;`git merge`) which has its limitations: I don't want my local changes to be overriden inside my "*improve-challenge-explanations*" if I still plan to update it further, or the PR has comments and other suggestions alike, at that point I **must not** `git pull`, but rather wait, and let this branch be as-is; and if I wanted to start working on a different feature, in a different file, I'd `git checkout main` and `git pull origin main` and then create a new branch from there `git checkout -b new-branch-name`, branch-off and keep working on another feature.
          - Which, funnily enough the **scenario** I was predicting/talking about (_or at least somewhat similar_) has happened to me and is written in _Experience # '3.'_ below.
            
3. All of it was done for my Cars Club Frontend's repository (IDs of the PRs are: ever since [pull/50](https://github.com/Aleksandar15/Cars-Club-frontend/pull/50) - until [pull/59](https://github.com/Aleksandar15/Cars-Club-frontend/pull/59)).
- **UPDATE3:** (*As written in the experience number #2, above, towards the very ending:*) Again, all of this could have been avoided if I had created a branch by following the instructions below - instead of branching off of "_feature_18_" branch and gathering its **unwanted commits history** for the back-then-new "_hotfix_2_" branch - to instead have switched to "**main**" branch and *#1* either running `git pull origin main` then `git checkout -b hotifx_2`; or *#2* `git checkout -b hotfix_2` then `git pull origin main` (*in 2nd case if: I don't want to update my local **"main"** branch* - as ***tested***, both work exactly the same for **"hotfix_2"**):
  - (As I had written in my own [comment](https://github.com/Aleksandar15/Cars-Club-frontend/pull/56#issuecomment-1998917385) in [/pull/56](https://github.com/Aleksandar15/Cars-Club-frontend/pull/56) at Cars-Club-frontend.)
  - Create a branch
  - 1. `git checkout main` (or `git switch main`) from any folder in my local `project-folder-name` repository.
    1. `git pull origin main` to ensure I have the **latest `main`** code.
    1. `git checkout -b the-name-of-my-branch` (replacing `the-name-of-my-branch` with a suitable name) to create a branch.
       - (*Source for these steps are the [react.dev doc's](https://github.com/reactjs/react.dev?tab=readme-ov-file#create-a-branch) at their GitHub README.*)
- **UPDATE2:** Looking back at it, once I was inside my _then-new_ "**hotfix2**" branch I could have "uncommit" the last 2 commits by running the command `git reset --soft HEAD~2`: and only to `git add` then `git commit` the 3rd Hotfix commit that I will `git push origin hotfix2`; and then switch back to my "**feature_18**" and continue my feature-work (with those _2 commit **intact**_). **EXTRA:** Also once that given "hotfix" is merged, when I switch to `feature_18` I will need to re-run `git pull main`.
  - (*Update2 is above ORIGINAL for easy readability.*)

- **ORIGINAL:** New experiences **explanations** -> the issue was **started** by the mere fact that I did **not** expect switching (by either _`git checkout -b` or `git switch -c`_, _doesn't matter_ which command still _same outcome_**)** from `feature_18` branch (_a branch that has a few `git commit`s but **0** `git push`es_) into a new fresh `hotfix_2` branch would still keep up the _old commits_ from my previous branch `feature_18` -> and hence, while on my then-new `hotfix_2` branch I 1. did my file-changes to fix the bug; 2. ran `git commit -m`; 3. then, running `git push` command turned into a messy `push` of both the _**hotfix bug fixes**_ as well as the _features from my `feature_18` branch_ which were **not** meant for this `hotfix_2` branch!

- Then as a solution I had created new branch **hotfix_2.0** and I did all of its mistakes that ***hotfix_2***'s branch & _same-named_ PR has had:
  - 2 unwanted commits from ***feature_18*** branch; and 1 correct commit from _hotfix_2_ branch.
  - I had tried using `git rebase -i` command that ChatGPT suggested to me, but after many merge conflicts - then, all due to my lack of knowledge with this command I can **not** tell exactly whether 1. the command itself can't do what I wanted it to do; or 2. whether I lack the knowledge (ex.: which _flags_ to use with `git rebase`, etc.) to achieve what I wanted => that is: to remove the 2 unwanted commits from ***feature_18*** branch & to keep the last and **only** 1 correct commit from _hotfix_2_ branch that I do wanted to `git push` & merge the PR quickly into my `main` branch!
  - For visualization this is how my commits looked like:
    1. commit1 (unwanted)
    2. commit2 (unwanted)
    3. commit3 (hotfix changes I wanted to ***push*** immediately; all the while I was working on features in my Cars Club app)
  - I came up with my own solution that is to reset the newly created branch ***hotfix2.0*** into the `origin/main`; and so I did ran these commands: `git reset --hard origin/main` followed by `git push origin hotfix_2.0 -f`.

- After running `git reset --hard origin/main` and then `git push origin hotfix_2.0 -f` the GitHub.com website has automatically "_merged and closed_" this very same _hotfix_2.0_ PR (which has had ***2 unwanted and 1 correct commit***); GitHub also automatically deleted the older commits (all 3 of them _dissapeared_) - expectedly so! ✔ However on my local machine switching to my older branches (say ***feature_18*** or ***hotfix2***) I can see these commits in there - which is all good. 👍

- As a reminder my main issue was with "_`hotfix_2`_" (without the _.0_, as branch _`hotfix_2.0`_ is a different one) branch - there I had the commits from the other branch (_feature_18_) so the _hotfix_2_ was about to commit unwanted-future-based _**feature**_ commits (committed in _feature_18_ but not yet _`git push`ed_), and as such, the situation, technically was impossible to remove certain commits only to keep the latest commit inside my _hotfix_2_ branch, so I had created this new branch _hotfix_2.0_ instead, only to, as a last resort, try my own logic of modifying both the _local files/working directory_ AND my commits to match my latest version of _main branch_ (`git reset --hard origin/main`) and then later to re-add **_hotfix bug fixes/changes_** and then run `git push origin hotfix_2.0` once I'm done with my changes, so that that way I'd avoid pushing the commits from _feature_18 branch_.

- Additionally, I've got answers by ChatGPT suggesting me to use `git rebase -i` command, but it's either my lack of knowledge about this command or the incapability of its functionality to do what I wanted it to do. For now I can't be 100% sure because my lack of deep knowledge about `git rebase` command.

- I've also tried running `git pull origin main` (that's `git fetch` + `git merge`) but of course that didn't do what I wanted to do because I had no such issue that'd be fixed by pulling the commits from _remote branches_ - I rather wanted to modify both my _local files/working directory_ AND my commits into my new "_hotfix_2.0_ branch" (_to match the "**main** branch_") in order to only commit the **_hotfix bug fixes_**, and so the above worked. :)

- Of course switching back to my _feature_18_ branch the old commits are kept, also my local files re-update to match my old files-updates ("old files-updates" = pre-_hotfix_2.0_ files-updates) so it's all fine on my _working directory/working tree_! 👍 

- As well of course I can't click the button _"reopen and comment"_ the _hotfix_2.0_ PR because, logically, GitHub says: "there are no new commits on the hotfix_2.0 branch", because, again, technically I've only "removed" the old mistaken commits (by running _`git reset --hard origin/main`_) and re-push to this very same _hotfix_2.0_ branch by using the `-f` flag short of `--force` flag.
  - But, once I do some edits on my _working directory_, then I run commands of 1. _git add ._ + 2. _git commit -m""_ + 3. _git push_ then GitHub.com will auto re-open the _`hotfix_2.0`_ PR.
    - I'm not sure what would have happened if there are multiple closed PRs of the same "_hotfix-branch to main branch" relationship_, but as of the moment I did my test there was only 1 closed PR "_`hotfix_2.0`_" so it auto-opened by GitHub.com.
      - Even if I have multiple closed PR's of the same "_`hotfix`-branch to `main` branch" relationship_, logically I can **only** open 1 of these closed PRs while the rest of the closed PR's "_reopen PR_" buttons will remain grayed out/unclickable and GitHub (_expectedly_) says: **_"There is already an open pull request from `hotfix_2.0` to `main`."_**.
4. Continuation from exeperience #3, above, while as I already knew that `git switch`ing between branches **A/B/C/D** updates the working directory accordingly (to their *latest commit(?)*), then while I was in the Branch "**B**" (*which by the way could be any number of commits ahead of the rest of the branches **A/C/D***) I had the need of running `git reset --soft ccba242`, then by checking the _repository history_ (by running `git reflog`) and finding the very same "`ccba242`" as its _latest hash_ matching across all of the rest of the branches (***which happened to be my case***) then wherever I `git checkout`/`git switch` from branch **B** to say branch **A/C/D** I'd notice something *unexpectedly* happened that was the fact that those branches didn't updated their previous *working directory* (*as was the case previously*) & **instead** they kept the ***working directory*** from whatever *local edits I made* in my branch `"B"`. The **reason** why was because now all of the branches "**A/B/C/D**" were having had the same hash "`ccba242`" as their latest hash.
   - Additional reminder as to why my branches "**A/C/D**" were all having had the same _latest hash commit_ was because I had branched off of branch "**A**" for both the branch "**C**" and "**D**", (as well as the branch "**B**" but in there I was doing the messing around as explained above in the experience #3; TLDR: the `HOTFIX2` issue I have had for my cars club frontend repository).
5. _(Date 3 June 2024: Here would be writing a new case scenario of undeletable corrupted files in my Cars Club Frontend directory, the 3 files that are unremovable/unrenamable/unmovable are: `index` in `.git` folder, then in the main projet's `src` folder's `utilities` the following 2 folders (and its files) were corrupted: `modalPost_FN` + `axiosInterceptor`. I will write about my workarounds by re-running command __`git remote add`__ (re-git-remote-adding) the project to a new directory (named __frontend3__) and copy pasting all but `index` file from .git folder (because `index` is unmovable))_
6. Just testing today to see what message is returned if I try to push (**git push**) into repository that I `git clone`d without me owning the project, here's the CLI error message that's returned by GitHub:
```
$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   yarn.lock

no changes added to commit (use "git add" and/or "git commit -a")

aleks@Aleksandar MINGW64 /A/folderName/project-name (main)
$ git push
remote: Permission to randomUsername/project-name.git denied to Aleksandar15.
fatal: unable to access 'https://github.com/randomUsername/project-name.git/': The requested URL returned error: 403
```
  - Mind you, I am able to `git add` or `git commit` as that's kept in my local repository.
