---
title: github使用
date: 2019-10-27 11:01:53
tags: github
categories: 
---

## github删除子模块
参考
[这个](https://stackoverflow.com/a/29850245)


```bash
git submodule deinit <asubmodule>    
git rm <asubmodule>
# Note: asubmodule (no trailing slash)
# or, if you want to leave it in your working tree
git rm --cached <asubmodule>
rm -rf .git/modules/<asubmodule>
```

## 插入自定义代码块
[snippet generator](https://snippet-generator.app/?description=website&tabtrigger=wbs&snippet=%5B%241%5D%28%242%29%240&mode=vscode)

直接在上述网址中输入即可

## 删除remote上的commit
- 思路就是先从本地的log中找出所需的commit，然后本地先回滚，然后push -f 强行改写remote上的记录即可
- 具体操作如下：

    - 通过revert来（commit还是保留了下来）

        1. 首先通过 `git log` 获得所有commit 挑选出需要的

        ```bash
        git revert <commit>       
        git push -f <remote>
        ```
    - 通过reset来（之后的commit都丢弃）
        ```bash
        git reset --hard <commit>       
        git push -f <remote>
        ```

## 设置或者取消代理

### 设置全局代理

```bash
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
```

### 取消代理


```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```



# 添加remote

[Adding a remote](https://docs.github.com/en/free-pro-team@latest/github/using-git/adding-a-remote)

```bash
$ git remote add origin https://github.com/user/repo.git
# Set a new remote

$ git remote -v
# Verify new remote
> origin  https://github.com/user/repo.git (fetch)
> origin  https://github.com/user/repo.git (push)
```



# `git add`所有文件，但排除一部分

参考 [Add all files to a commit except a single file?](https://stackoverflow.com/questions/4475457/add-all-files-to-a-commit-except-a-single-file)

> Now `git` supports `exclude certain paths and files` by pathspec magic `:(exclude)` and its short form `:!`. So you can easily achieve it as the following command.
>
> ```bash
> git add --all -- :!main/dontcheckmein.txt
> git add -- . :!main/dontcheckmein.txt
> ```
>
> Actually you can specify more:
>
> ```bash
> git add --all -- :!path/to/file1 :!path/to/file2 :!path/to/folder1/*
> git add -- . :!path/to/file1 :!path/to/file2 :!path/to/folder1/*
> ```
>
> For Mac and Linux, surround each file/folder path with quotes
>
> ```bash
> git add --all -- ':!path/to/file1' ':!path/to/file2' ':!path/to/folder1/*'
> ```



使用：

1. 尝试在dry mode下看看add的文件：eg:`git add -n -- . :!pytorch_new_templates\configs\image_single_frame\*`
2. 确认无误后继续添加



# 使用github template

## 基本步骤

1. 在其他人的template repository中点击use this template，创建一个属于自己的且基于该template的repo
2. clone到本地，`git remote add`，注意要添加两个remote：自己的repo和template的repo（尤其后者，这样能保证fetch updates）

<img src="https://raw.githubusercontent.com/khalitt/IMG_REPO/master/20210113213442.png" align=center/>



3. 照常fetch pull
4. 设置branch，注意要设置到本地的branch



## template和当前仓库之间的关系

参考 [how to merge or update a template repository?](https://softwareengineering.stackexchange.com/a/378052)

理论上二者是没关系的，本质上上面这种方法类似于[fork](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/fork-a-repo)，因此fetch updates时参考[Syncing a fork](https://docs.github.com/en/free-pro-team@latest/github/collaborating-with-issues-and-pull-requests/syncing-a-fork)

# 删除remote

```bash
usage: git remote remove <name>
```



# 解决Git中fatal: refusing to merge unrelated histories

在你操作命令后面加`--allow-unrelated-histories`

例如：
`git merge master --allow-unrelated-histories`

```
~/SpringSpace/newframe on  druid ⌚ 11:36:49
$ git merge master --allow-unrelated-histories
Auto-merging .gitignore
CONFLICT (add/add): Merge conflict in .gitignore
Automatic merge failed; fix conflicts and then commit the result.12345
```

如果你是`git pull`或者`git push`报`fatal: refusing to merge unrelated histories`
同理：
`git pull origin master --allow-unrelated-histories`
等等，就是这样完美的解决咯！



# 停止track原来被track 的文件（修改gitignore后马上生效）

参考 [How to make Git “forget” about a file that was tracked but is now in .gitignore?](https://stackoverflow.com/a/1274447)

`.gitignore` will **prevent untracked files from being added** (without an `add -f`) to the set of files tracked by git, **however git will continue to track any files that are already being tracked**.

To stop tracking a file you need to remove it from the index. This can be achieved with this command.

```
git rm --cached <file>
```

If you want to remove a whole folder, you need to remove all files in it recursively.

```
git rm -r --cached <folder>
```

The removal of the file from the head revision will happen on the next commit.

**WARNING: While this will not remove the physical file from your local, it will remove the files from other developers machines on next `git pull`.**



# uncommitt：reset

参考 [How to uncommit my last commit in Git [duplicate]](https://stackoverflow.com/a/2846154)

If you aren't totally sure what you mean by "uncommit" and don't know if you want to use `git reset`, please see "[Revert to a previous Git commit](https://stackoverflow.com/q/4114095/119963)".

If you're trying to understand `git reset` better, please see "[Can you explain what "git reset" does in plain English?](https://stackoverflow.com/questions/2530060/can-you-explain-what-git-reset-does-in-plain-english)".

------

* 仅仅uncommit：If you know you want to use `git reset`, it still depends what you mean by "uncommit". If all you want to do is **undo the act of committing, leaving everything else intact**, use:

```
git reset --soft HEAD^
```

* uncommit + 从staged files中删去：If you want to **undo the act of committing and everything you'd staged, but leave the work tree (your files intact):

```
git reset HEAD^
```

* uncommit + 从staged files中删去 + 去除修改：And if you actually want to *completely* undo it, **throwing away all uncommitted changes, resetting everything to the previous commit** (as the original question asked):

```
git reset --hard HEAD^
```

------

The original question also asked it's `HEAD^` not `HEAD`. `HEAD` refers to the current commit - generally, the tip of the currently checked-out branch. The `^` is a notation which can be attached to *any* commit specifier, and means "the commit before". So, `HEAD^` is the commit before the current one, just as `master^` is the commit before the tip of the master branch.

Here's the portion of the [git-rev-parse documentation](http://linux.die.net/man/1/git-rev-parse) describing all of the ways to specify commits (`^` is just a basic one among many).



另一个方法：

## Undo a commit & redo

```sh
$ git commit -m "Something terribly misguided" # (0: Your Accident)
$ git reset HEAD~                              # (1)
[ edit files as necessary ]                    # (2)
$ git add .                                    # (3)
$ git commit -c ORIG_HEAD                      # (4)
```

1. This command is responsible for the **undo**. It will undo your last commit while **leaving your working tree (the state of your files on disk) untouched.** You'll need to add them again before you can commit them again).
2. Make corrections to working tree files.
3. `git add` anything that you want to include in your new commit.
4. Commit the changes, reusing the old commit message. `reset` copied the old head to `.git/ORIG_HEAD`; `commit` with `-c ORIG_HEAD` will open an editor, which initially contains the log message from the old commit and allows you to edit it. If you do not need to edit the message, you could use the `-C` option.

**Alternatively, to edit the previous commit (or just its commit message)**, `commit --amend` will add changes within the current index to the previous commit.

**To remove (not revert) a commit that has been pushed to the server**, rewriting history with `git push origin master --force` is necessary.

------

### Further Reading

[How can I move HEAD back to a previous location? (Detached head) & Undo commits](https://stackoverflow.com/questions/34519665/how-to-move-head-back-to-a-previous-location-detached-head/34519716#34519716)

The above answer will show you `git reflog`, which you can use to determine the SHA-1 for the commit to which you wish to revert. Once you have this value, use the sequence of commands as explained above.

------

`HEAD~` is the same as `HEAD~1`. The article [What is the HEAD in git?](https://stackoverflow.com/a/46350644/5175709) is helpful if you want to uncommit multiple commits.



# 添加文件到last commit/any commit

## 添加到last commit

[How to add a file to the last commit in git? [duplicate]](https://stackoverflow.com/a/40503483)



```
git add the_left_out_file
git commit --amend --no-edit
```



## 添加到任意commit

关键：找到sha-1



Use [`git rebase`](http://git-scm.com/docs/git-rebase). Specifically:

1. Use `git stash` to store the changes you want to add.
2. Use `git rebase -i HEAD~10` (or however many commits back you want to see).
3. Mark the commit in question (`a0865...`) for edit by changing the word `pick` at the start of the line into `edit`. Don't delete the other lines as that would delete the commits.
4. Save the rebase file, and git will drop back to the shell and wait for you to fix that commit.
5. Pop the stash by using `git stash pop`
6. Add your file with `git add <file>`.
7. Amend the commit with `git commit --amend --no-edit`.
8. Do a `git rebase --continue` which will rewrite the rest of your commits against the new one.
9. Repeat from step 2 onwards if you have marked more than one commit for edit.

If you are using `vim` then you will have to hit the Insert key to edit, then Esc and type in `:wq` to save the file, quit the editor, and apply the changes. Alternatively, you can [configure a user-friendly git commit editor](https://stackoverflow.com/questions/2596805/how-do-i-make-git-use-the-editor-of-my-choice-for-commits) with `git config --global core.editor "nano"`.



## 发生You must edit all merge conflicts and then mark them as resolved using git add

<img src="https://raw.githubusercontent.com/khalitt/IMG_REPO/master/20210113213538.png" align=center/>

再把unstaged的文件add且commit后，再`git rebase --continue`即可

<img src="https://raw.githubusercontent.com/khalitt/IMG_REPO/master/20210113213500.png" align=center/>