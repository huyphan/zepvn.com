---
title: '[git] Change your previous commit'
author: zepvn
layout: post
categories:
  - linux
tags:
  - git
excerpt: This post is a note for myself when dealing with git. It shows you the way to go back and change your previous commit without losing any of the new work you’ve done since you commited.
---
This post is a note for myself when dealing with git. It shows you the way to go back and change your previous commit without losing any of the new work you’ve done since you commited. Source of this post can be found at this [link][1]

  1. Save and stash your work so far.
  2. Look at ***`git log`*** and copy the first 5 or so characters from the ID of the commit you want to edit onto your clipboard.
  3. Start the interactive rebase process, pasting in the characters from the ID: ***`git rebase --interactive ID`***
  4. Your editor will come up with several lines like*** `pick d3adb33 Commit message`***, one line for each commit since the older one.
  5. Change the word “pick” to “edit” in front of the commit you want to change.
  6. Save and quit.
  7. Edit your project files to make the correction, then run `git<em><strong> commit --all --amend</strong></em>`.
  8. After you’ve committed the fixed version, do ***`git rebase --continue`***.
  9. Git will do its magic, recreating all the commits since then.
 10. You might need to resolve some conflicts, if the change you made affected later commits. Follow Git’s instructions to resolve those.
 11. Once the rebase is done, re-apply the stash and continue happily with your life.

 [1]: http://blog.jacius.info/articles/2008/6/22/git-tip-fix-a-mistake-in-a-previous-commit
