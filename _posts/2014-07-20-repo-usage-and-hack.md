---
layout: post
title: Repo usage and hack
category: learn
tags: [git]

---

My repo hack source [git-repo](https://github.com/xianyo/git-repo)


Install
========

* Google version(in github mirror)

	```bash
	$ curl -L -k https://raw.githubusercontent.com/xianyo/git-repo/stable-github/repo > /tmp/repo
	$ sudo mv /tmp/repo /usr/local/bin/repo
	$ sudo chmod a+x /usr/local/bin/repo
	```

<!--break-->

* Hack version

	```bash  
	$ curl -L -k https://raw.githubusercontent.com/xianyo/git-repo/stable-hack/repo > /tmp/repo
	$ sudo mv /tmp/repo /usr/local/bin/repo
	$ sudo chmod a+x /usr/local/bin/repo
	```


HACK
========

### repo push

```
Upload changes for no code review

Usage: repo push [-u <branch>] [-d <branch/tag>] [-t <tag>] [-x <cmd>] [-r <remote>] [--tags] {[<project>]... }

Options:
  -r REMOTE   Choice the remote group project.
  -u BRANCH   Create new feature branch on git server.
  -d NAME     Delete branch or tag on git server.
  -t TAG      Push tag.
  --tags      Push all tags.
  -x CMD      Execute the cmd.
```

### repo tag

```
Create tag

Usage: repo tag <tagname> [-m <msg>] [-r <remote>] [-x <cmd>] [-d <tag>] [<project>...]

Options:
  -m MSG      Tag commit.
  -x CMD      Execute the cmd.
  -d TAG      Delete the tag.
  -r REMOTE   Choice the remote group project.
```

