# 如何导入另一个 Git库到现有的Git库并保留提交记录

推荐使用  `git subtree` 

> `git subtree` 可以让一个 repository 嵌入到另一个项目的子目录中。

```bash
git subtree add --prefix=install-pkg https://github.com/antfu/install-pkg.git main
```



> 参考文章：[git subtree 简单使用记录](https://einverne.github.io/post/2020/04/git-subtree-usage.html)

## git subtree 相关命令

常用的 `git subtree` 命令：

```
git subtree add   --prefix=<prefix> <commit>
git subtree add   --prefix=<prefix> <repository> <ref>
git subtree pull  --prefix=<prefix> <repository> <ref>
git subtree push  --prefix=<prefix> <repository> <ref>
git subtree merge --prefix=<prefix> <commit>
git subtree split --prefix=<prefix> [OPTIONS] [<commit>]
```

### 在父仓库中添加子项目

和 git submodule 一样，在使用 subtree 的时候也需要显式的指定需要添加的子项目

```
git subtree add --prefix=foo https://github.com/einverne/foo.git master --squash
```

解释：

- `--squash` 是将 subtree 的改动合并到一个 commit，不用拉取子项目完整的历史纪录
- 这里 `--prefix` 后面的 `=` 也可以使用空格，注意这里的 `foo` 就是项目克隆后在本地的目录名
- 命令中的 `master` 指的是 subtree 项目的分支名
- 可以使用 `git status` 和 `git log` 查看提交

使用 `git subtree` 添加项目后，subtree 就将原来的项目作为这个主项目的一个普通文件夹来看待了，对于父级项目而言完全无缝透明。上面的命令就是将 foo 这个项目添加到主项目中 foo 文件夹下。

日常更新的时候，正常的提交代码，如果更改了 foo 目录中的内容也正常的提交即可。

### 更新子项目仓库

如果依赖的子项目更新了，可以通过如下命令更新：

```
git subtree pull --prefix=foo https://github.com/einverne/foo.git master --squash
```

上面命令执行后，就可以将 foo 仓库中 master 上的更新更新到本地，`--squash` 表示只会在父项目生成一个 commit 提交。

### 将更改推送到子项目仓库

假如在修改代码时修改了依赖的 foo 中的代码，那么可以将这部分代码推送到远端仓库

```
git subtree push --prefix=foo https://github.com/einverne/foo.git master
```

看到这里可能发现，每一次操作 subtree 添加的项目时都需要敲一大段 URL 地址，这里可以使用 remote 来简化命令：

```
git remote add -f foo https://github.com/einverne/foo.git
```

然后

```
git subtree add --prefix=foo foo master --squash
git subtree pull --prefix=foo foo master --squash
git subtree push --prefix=foo foo master
```

到这里其实我们就可以发现，利用 subtree 或者 submodule 也好，都可以很方便的同步子项目，尤其是很多项目都依赖的那个项目。比如 A，B 都依赖与 Z，项目，那么使用这种方式可以很方便的在不同 A，B 项目间共享对 Z 的修改。比如 A，B 都依赖与 Z，项目，那么使用这种方式可以很方便的在不同 A，B 项目间共享对 Z 的修改。

#### 将推送的修改合并成一次提交

`git subtree push` 会将父项目中的提交每一次都进行提交，这会导致对于子项目来说无意义的提交信息，但是 `git subtree` 并没有提供类似 squash 的方式可以将多次提交合并成一次提交，但是 `git subtree` 提供了分支的特性，可以在父项目中将修改推送到某一个分支，然后在子项目中使用 squash merge 将修改合并到主干分支。

```
git subtree push --prefix=foo foo feature
```

这会在 foo 的仓库中创建一个叫做 feature 的分支。然后可以从 feature 分支合并回 master 分支。

一旦最新的提交都合并到 master 分支，可以通过 `pull` 来更新

```
git subtree pull --prefix=foo foo master --squash
```

这会在主项目中创建另外一个提交，包括了子项目中所有的修改。

这样的方式唯一的缺点就是会在父项目中产生一些多余的提交信息。

## 常见的使用场景

典型的使用场景就是当 A，B 项目同时依赖 Z 项目，需要对 Z 项目进行管理的时候。

### 使用 git subtree 来管理 vim plugin

虽然可以用 subtree 来管理 vim plugins，但我个人跟倾向于使用 [vim-plug](https://github.com/junegunn/vim-plug) 来管理 Vim 的插件。

这里不过是拿 vim plugin 管理来作为一个例子。[1](https://einverne.github.io/post/2020/04/git-subtree-usage.html#fn:vim)

```
# add
git subtree add --prefix .vim/bundle/vim-fugitive https://github.com/tpope/vim-fugitive.git master --squash
# update
git subtree pull --prefix .vim/bundle/vim-fugitive https://github.com/tpope/vim-fugitive.git master --squash
```

### 使用 git subtree 来管理博客的主题

无论是 WordPress 还是静态内容生成器 Hexo，Hugo 之类，他们的主题都是主项目下的一个文件夹，如果主题自身是一个仓库的话，那么可以直接使用 subtree 将主题放到主项目中来管理，修改也可以同步提交到主项目中，如果想要提交回主题本身也可以很方便的实现。

```
git subtree push --prefix=foo foo master --squash
```

`push` 的时候添加 `--squash` 会将多个提交合并成一个。

## 缺点

固然 subtree 有很多的优点，解决了 submodule 存在的一些问题，但是 subtree 也有其自己的问题。

### 子项目将修改提交到 upstream 的过程变得复杂

在 submodule 时，子项目是一个单独的项目，和所有的 git 管理的项目一样，可以在子项目中提交，提交再 `push` 到 upstream ，并且每一个 submodule 都有自己完整的提交历史。

而在 `subtree` 中因为对子项目的所有修改已经和主项目混合到了一起，所以需要单独对子项目提交，命令：

```
git subtree push --prefix=foo foo master
```

在 subtree 执行 push 命令时，git subtree 会为子项目生成新的提交，这个时候就会有一个问题。假如主项目提交过多，那么在 push 到子项目时会花费大量的时间来重新计算子项目的提交。并且因为每次 push 都是重新计算的，所以本地仓库和远端仓库的提交总是不一样的，这会导致 git 无法解决可能的冲突。

基于这些原因，git subtree 提供了 `split` 命令。

> Extract a new, synthetic project history from the history of the subtree. The new history includes only the commits (including merges) that affected , and each of those commits now has the contents of at the root of the project instead of in a subdirectory. Thus, the newly created history is suitable for export as a separate git repository. After splitting successfully, a single commit id is printed to stdout. This corresponds to the HEAD of the newly created tree, which you can manipulate however you want. Repeated splits of exactly the same history are guaranteed to be identical (ie. to produce the same commit ids). Because of this, if you add new commits and then re-split, the new commits will be attached as commits on top of the history you generated last time, so ‘git merge’ and friends will work as expected. Note that if you use ‘-squash’ when you merge, you should usually not just ‘-rejoin’ when you split.

当使用 `split` 命令后，使用 `git subtree push`，git 只会计算 split 后的新提交。[2](https://einverne.github.io/post/2020/04/git-subtree-usage.html#fn:split)[3](https://einverne.github.io/post/2020/04/git-subtree-usage.html#fn:s2)

这里需要注意：如果 push 使用了 `--squash` 合并提交，那么 `split` 时不能使用 `--rejoin` 参数。运行上面的命令后主项目中就多了一个 foo-branch 的分支，这个分支作为子项目的起点，下次 push 时就只会从这个点开始重新计算。

### 需要学习 subtree 的 merge 方法

使用 subtree 的时候，如果能够 `pull` 下来当然 OK，但是一旦出现 Conflict，过程就比较复杂了，甚至需要专门的 git subtree merge strategy 来解决。

假设把 Bproject 作为子项目

```
# -f flag tell git to immediately fetch the commits after adding
git remote add -f Bproject /path/to/B
```

准备 merge:

```
git merge -s ours --no-commit --allow-unrelated-histories Bproject/master
```

这告诉 git 准备一次 merge commit，使用 `ours` merge strategy 来告诉 git 我们需要 merge 的结果是 HEAD，然后将 project b 添加到 repository:

```
git read-tree --prefix=dir-B/ -u Bproject/master
```

`read-tree` 命令读取 B 项目的 master-tree 到 index 中，然后将其保存到给定的目录中 `dir-B/`，注意这里的目录结尾必须有 `/`，`-u` 选项让 read-tree 同样更新 working 目录中的内容。

最后提交：

```
git commit -m "Merge Project B into dir-B"
```

以后的更新，只需要 pull 即可

```
git pull -s subtree Bproject master
```

上面的流程可以看到，虽然使用 subtree 将项目中包含另一个项目的细节对项目的成员隐藏了，但实际上在产生冲突时需要管理者特别注意。[4](https://einverne.github.io/post/2020/04/git-subtree-usage.html#fn:merge)[5](https://einverne.github.io/post/2020/04/git-subtree-usage.html#fn:m2)[6](https://einverne.github.io/post/2020/04/git-subtree-usage.html#fn:m3)

## subtree 如何切换分支

使用 `git subtree` 加入到父项目的仓库，如果要切换分支，可以直接将 subtree 删掉，然后新加入子项目的分支即可。

```
git rm <subtree>
git commit
git subtree add --prefix=<subtree> <repository_url> <subtree_branch>
```

## 移除 subtree

如果有一天引入的 subtree 不再有用，那么整个 subtree 可以从项目中删除。

```
git rm -r subtree_folder
git commit -m "Remove subtree"
```

这种方式是删除整个 subtree，然后提交，但是问题在于 git log 中还会留有 subtree 的提交历史。

还可以使用 filter-branch 来重写提交历史：

```
git filter-branch -f --index-filter "git rm -r -f -q --cached --ignore-unmatch subtree_folder" --prune-empty HEAD
```

这种方式可以将 subtree 的提交历史也删除。

## 使用建议

就和上文所说那样，因为对 subtree 目录的修改和主项目是混合在一起的。所以为了让 commit messages 清晰，可以对主项目和子项目的修改分开进行。当然如果不在意子项目的 commit messages，那么一起提交，然后在对 subtree push 的时候再统一对 commit message 进行修改也可以。

## reference

- https://github.com/apenwarr/git-subtree/
- https://www.atlassian.com/blog/git/alternatives-to-git-submodule-git-subtree
- https://lostechies.com/johnteague/2014/04/04/using-git-subtrees-to-split-a-repository/

1. http://endot.org/2011/05/18/git-submodules-vs-subtrees-for-vim-plugins/ [↩](https://einverne.github.io/post/2020/04/git-subtree-usage.html#fnref:vim)

2. https://blog.walterlv.com/post/performance-of-git-subtree.html [↩](https://einverne.github.io/post/2020/04/git-subtree-usage.html#fnref:split)

3. https://github.com/apenwarr/git-subtree/blob/master/git-subtree.txt

   git subtree split [–rejoin] –prefix=foo –branch foo-branch [↩](https://einverne.github.io/post/2020/04/git-subtree-usage.html#fnref:s2)

4. https://mirrors.edge.kernel.org/pub/software/scm/git/docs/howto/using-merge-subtree.html [↩](https://einverne.github.io/post/2020/04/git-subtree-usage.html#fnref:merge)

5. https://help.github.com/en/github/using-git/about-git-subtree-merges [↩](https://einverne.github.io/post/2020/04/git-subtree-usage.html#fnref:m2)

6. https://stackoverflow.com/a/25311871/1820217 [↩](https://einverne.github.io/post/2020/04/git-subtree-usage.html#fnref:m3)
