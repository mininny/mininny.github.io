---
title:  "(한글) Highlights from Git 2.26?"
category: translation
date: 2020-03-24
---

> This is translated work from The GitHub Blog [post](https://github.blog/2020-03-22-highlights-from-git-2-26/) by Taylor Blau.  

깃 프로젝트가 얼마전에 Git 2.26을 배포했습니다. 이제 Git 2.25버전에서부터 바뀐 항목들을 살펴보도록 하겠습니다.

## 이젠 프로토콜 버전 2가 기본 설정입니다.

2018년도에 Git이 network fetch protocol을 [새로 소개했던것을](https://github.blog/2018-09-10-highlights-from-git-2-19/) 기억하실수 있는데요, 2.26버전에서부터는 이 프로토콜이 기본으로 사용됩니다. 그러면 뭐가 바뀌는지 알아보도록 할게요.

구버전 프로토콜의 가장 큰 문제는 클라이언트가 무슨 데이터를 보내기전에 서버가 바로 리포의 모든 브랜치, 태그, 등등 다른 항목을 바로 출력한다는것이었죠. 몇몇 리포에서는 클라이언트가 마스터 브랜치에만 관심이 있는데 쓸데없는 데이터를 몇메가씩 보내게 된다는 것이었습니다.

새로운 프로토콜은 클라이언트의 리퀘스트로 시작이 되고, 클라이언트가 서버에게 무슨 항목에 관심이 있는지 알려줄수 있습니다. 한개의 브랜치를 fetch한다면 그 브랜치에 대해서만 물어볼 것이고, 클론을 한다면 브랜치와 태그에 관해서만 테이터를 받을수 있죠. 브랜치랑 태그 말고 뭐가 더 있어? 할수도 있겠지만, 서버 리포들은 다른 항목들(예를들면 모든 PR의 HEAD)을 저장할수 있었죠.

이젠 큰 리포에서 fetch하는것도 매우 빨라질거에요, 특히 fetch가 더 작다면요.

그리고 가장 좋은 점은, 아무것도 구현을 안해도 된다는것입니다! 좀 똑똑한 설계 덕분에, 새로운 프로토콜을 사용하는 모든 클라이언트는 문제없이 예전 프로토콜이나 새로운 프로토콜을 사용하는 서버들과 통신할수 있죠. 만약에 서버가 새로운 프로토콜을 지원을 안한다면, 기존 프로토콜을 채택 하게 됩니다. 2018년 출시 이후로 이렇게 늦게 배포하는 이유는 에러를 줄이기 위해...

만약에 업데이트 준비가 되지않았다면, Git 2.19에서 `git config --global protocol.version 2` 를 사용해서 작동여부를 확인해볼수 있습니다.

더 테크니컬한게 궁금하시다면 [Introducing Git protocol version 2](https://opensource.googleblog.com/2018/05/introducing-git-protocol-version-2.html)를 확인해보세요. 

[source](https://github.com/git/git/compare/a14aebeac330e6d58f9628a02521ea780daf0a5b...684ceae32dae726c6a5c693b257b156926aba8b7)

## 새로운 Config 설정

깃은 설정을 여러군데에서 읽어올수 있죠: 리포에서 (.git/info/config), 또는 유저에서 (~/.gitconfig), 또는 시스템에서요 (/etc/gitconfig). 또 커맨드 라인이나 개발 환경에서 (예. `git -c foo=bar ...`)에서도 설정이 가능하죠. 그래서 가끔식 헷갈릴때가 있죠. 분명히 설정을 바꿨는데, 누가 바꿨는지 모르거나, 아니면 다른 설정과 겹치는지 모를수 있죠.

기준에는 `git config` 명령어에 `--show-origin` 옵션이 있었는데, 출력은 더 복잡했죠. 설정이 어디서 바뀌는지 실제 주소를 알려줬는데, 파일내에서 수정한다면 쓸모있겠지만, git config을 사용해서(예. `--system`, `--global`, `--local`) 값을 바꿀 거였다면 별로 도움이 안됬었죠.

깃 2.26에서 `--show-scope`이 생겼는데, 이 친구는 좀더 쓸모있는 정보를 알려주죠. 마치:
```
$ git config --show-scope --get-regexp 'diff.*'
global  diff.statgraphwidth 35
local   diff.colormoved plain

$ git config --global --unset diff.statgraphwidth
```

이 새로운 옵션하고 `--show-origin`을 같이 사용하면, 더 많은 정보를 다양하게 받을수 있습니다.
```
$ git --list --show-scope --show-origin
global  file:/home/user/.gitconfig      diff.interhunkcontext=1
global  file:/home/user/.gitconfig      push.default=current
[...]
local   file:.git/config      branch.master.remote=origin
local   file:.git/config      branch.master.merge=refs/heads/master
```

또 다른 개선사항은 이제 credential URL을 사용할때 와일드 카드를 쓸 수 있다는 것입니다. Git의 모든 HTTP설정은 모든 연결(`http.extraHeader`)이나 특정 연결(`http.https://example.com.extraHeader`)으로 변경할수 있었죠. Credential config도 똑같이 범용적으로나(`credential.helper`), 혹은 특정 URL(`credential.https://example.com.helper`)에 설정할수 있었죠. 

이젠 `http` config matcher 에서 `*.example.com`같은 와일드 카드를 사용할수 있습니다. 이젠 서브 도메인을 관리한다거나 할때 매우 유용하게 설정을 변경할수 있죠:
`[credential "https://*.example.com]`
위 설정은 `foo.example.com`, `bar.example.com`, 등등 에 적용됩니다.

[[source](https://github.com/git/git/compare/9f3f38769d49255d3fbf97f35a0dec591de17db3...145d59f48233c64cb8a9262c9f1451cc7d66b530), [source](https://github.com/git/git/compare/25063e2530e2fbdccf5d372a8ad80309421c9df1...46fd7b390034eb5dde6a3f955e677c5260e4d10c)]

## Updates to git sparse-checkout

From our last blog post and technical deep-dive, you may recall our discussion of the new sub-command git sparse-checkout. We recommend checking out both of those posts, but in case you did and forgot, let’s take some time to cover what sparse-checkouts are, and how they’re used.

Sparse-checkouts are a way to have only part of your repository checked out at a time. For example, let’s consider that you’re working in a monorepo, and only need the client/macos directory, and everything in it. You probably don’t want to spend time downloading old blobs from outside of that directory if you’re never going to use them. So, how do you do it?

Git does this in two parts:

First, it asks the server to only send tree and commit objects instead of all blobs.
Then, it tells the client to expect that some objects from the repository may be missing, and to ask the server for any of those objects if they are needed, say, for a checkout.
To tell Git to do both of these things, simply add --filter=blob:none and --sparse as command-line options to git clone, and your repository will be cloned in such a way as to only fetch blob objects as they’re needed.

You’ll notice the first time that you checkout your repository that your Git client will issue a subsequent fetch to the server. This is done automatically in order to fetch the blobs in the top-level of your working copy so that it can be checked out for you to browse around it.

In historical versions of Git, the only way to add new sub-directories to Git’s list of directories that need their blobs populated has been to run git sparse-checkout set, which sets the list of sparse-checkout directories, forcing you to re-specify them every time.

Git 2.26 now has a new git sparse-checkout add mode, which allows you to add new directory entries one at a time. Here’s an example:

$ git clone --filter=blob:none --sparse git@github.com:git/git.git
Cloning into 'git'...
remote: Enumerating objects: 175470, done.
remote: Total 175470 (delta 0), reused 0 (delta 0), pack-reused 175470
Receiving objects: 100% (175470/175470), 59.07 MiB | 10.48 MiB/s, done.
Resolving deltas: 100% (111328/111328), done.
remote: Enumerating objects: 379, done.
remote: Counting objects: 100% (379/379), done.
remote: Compressing objects: 100% (379/379), done.
remote: Total 431 (delta 0), reused 0 (delta 0), pack-reused 52
Receiving objects: 100% (431/431), 1.73 MiB | 4.06 MiB/s, done.
Updating files: 100% (432/432), done.

$ cd git

$ git sparse-checkout init --cone
$ git sparse-checkout add t
remote: Enumerating objects: 797, done.
# ...
Updating files: 100% (1946/1946), done.

$ git sparse-checkout add Documentation
remote: Enumerating objects: 334, done.
# ...
Updating files: 100% (723/723), done.

$ git sparse-checkout list
Documentation
t
In the example, we told Git to clone git/git excluding all blob objects and to avoid checking anything that wasn’t in the top-level directory out into our working copy. From the example, you can observe a few things:

There are two “enumerating objects” lines in the output of git clone. These come from the initial clone request, and the subsequent fetch to load in all blobs in the top-level directory.
Even after multiple git sparse-checkout adds, we don’t need to re-specify the directories that we want checked out, as we would have had to with git sparse-checkout set.
Sometimes Git asks for objects using more fetch requests than it needs to (like when we added directory t, three fetches took place when there only needed to be one). Some rough edges like this exist on the client side, and are being improved with each release.
Likewise, some rough edges have been cleaned up in the “cone” mode of sparse checkout. Learn more from Git’s documentation or on our blog.

[source, source]

Tidbits

If you’ve never used git grep to search through your Git repository, this release is a good time to try it out, since git grep is now faster than before. If you haven’t used git grep, here’s a quick run-down: git grep behaves like grep, but works in your Git repository. You can grep through the checked-out contents of your repository, but you can also grep through historical revisions, too.

git grep uses multiple threads to enhance its performance when scanning through the contents of your working tree. However, in previous releases of Git, due to some details of Git’s object storage mechanism, git grep avoided using multiple threads when looking at historical revisions.

In Git 2.26, this limitation is no more, thanks to work by Matheus Tavares, a Google Summer of Code student to make reading from the object storage layer support concurrent access. Now, you can enjoy all of the benefits of git grep --threads regardless of where you search. And since --threads defaults to the number of cores on your workstation, you don’t even have to type --threads at all.

[source]

Another lesser-known feature of Git are “worktrees”. In these posts, we’ve often discussed “the working copy”—in fact, there’s a reference in the tidbit just above this one! This has always intended to mean: “the copy of your repository that you have on your hard drive”. But, did you know that you can have multiple working copies per repository?

Though this may remind you of Git’s submodules, worktrees are entirely different. For example, I use a worktree to mount a special meta branch in my fork of Git, which contains scripts and Makefile tweaks. meta is a separate branch in ttaylorr/git without a history, but I can mount it in a top-level Meta directory in my checkout of ttaylorr/git to check scripts into my repository without having to send them upstream, or use a submodule.

In Git 2.26, the completion engine that powers the results you receive when you type git <TAB> learned about git worktree, and can now complete subcommands, paths, refs, and more.

[source]

Way back in our post about Git 2.19, we had a number of color-related tidbits. We talked about git config --color, colorization by age in git blame, and reminisced about git diff --color-moved. Since it’s been a few versions, it felt right to talk about Git’s use of color again.

In many Git commands, you can use the --format option to specify the appearance of the output to suit your liking. For example, you can write:

$ git log --format="%aN - %s"

This shows the author and message for each commit in the output of git log. These --format specifiers also support shorthands for the ANSI color escape sequences, so you can type %C(blue) instead of \x1b.

Now, Git supports the “bright” variant of the colors that have ANSI escape sequences, so you can now write %C(brightblue) to obtain, well, bright blue!

[source]

If you use Scalar or are otherwise in-the-know, you might know about Git’s capability to interact with fsmonitor-like tools (such as Facebook’s Watchman) in order to skip filesystem operations that are expensive over large trees. For example, instead of having Git query the filesystem for updates (which gets slower the larger your repository is on disk), Watchman can tell Git which files change, skipping those queries from Git altogether, making operations such as git status (which typically involves many filesystem interactions) much faster.

Watchman supports a number of different ways to tag the time at which the filesystem was last updated: the UNIX epoch, a vector-like clock identifier, and opaque tokens [source]. Because Watchman prefers the clock identifier style, Git has been updated to understand that this may be sent instead of a UNIX epoch.

When you upgrade to Git 2.26, all you need to do is replace the hook your repository uses to communicate with Watchman, and Git will work as expected.

[source]

When we talked about partial clones earlier, we highlighted that the “partial” in “partial clones” roughly means taking the set of objects Git would have sent you and filtering it down to just the ones that you want. Ordinarily, this filtration process requires a full object traversal and thus gets slower as the number of unfiltered objects increases. However, some of these checks can be made faster by using Git’s bitmap machinery, and in Git 2.26, they have.

Because we don’t need to perform an object traversal to check whether an object is a blob, or what size it is, partial clone filters --filter=blob:none and --filter=blob:limit=<n> can run much more quickly using just bitmaps.

We’re running these patches at GitHub, so you can experiment with them and try out partial clones today by cloning any repository on GitHub.com.

[source]

When you’re performing a rebase, previous versions of Git have used a different mechanism for merging based on whether or not you’re using rebase interactively (this is the difference between git rebase and git rebase -i).

Previously, git rebase -i used the “merge” backend, and git rebase used the “apply” backend. In 2.26, both git rebase and git rebase -i now both use the “merge” backend. The two backends behave slightly differently, and so it’s worth knowing about the differences.

For example: suppose you’re rebasing and a commit pauses with conflicts. After you resolve the conflicts and stage your files, you tell Git to move to the next step using git rebase --continue. This used to immediately move ahead, taking the old commit message verbatim. With the new backend, you’re now prompted to edit the commit message so you can make note of the resolved conflicts.

For more, the author of this change contributed excellent documentation on some caveats and differences between the “merge” backend, and its former counterpart the “apply” backend, which you can learn more about from Git’s documentation.

[source]

