+++
title = "Use 'git subtree' to create new repository from a sub-directory"    
date = 2017-10-15T14:45:49+05:30
tags = ["git"]
+++

As a part of my ongoing [Ansible blog
series](https://dharmitshah.com/series/ansible/), I plan to walk-through the
provisioning code of [CentOS Containr
Pipeline](http://github.com/CentOS/container-pipeline-service/) service which
lives under `provisions` directory.

My plan:

- Walk-through the existing code base
- Make changes to the code without touching my fork of the repo
- Push and test the new code

For the second part, i.e., modifying the code without touching my fork, I
needed to create a new git repository from the said
[`provisions`
directory](https://github.com/CentOS/container-pipeline-service/tree/master/provisions).

A bit of Google-fu and [this](https://stackoverflow.com/a/17864475/395670)
stackoverflow answer came to rescue! All I did was, use this `git subtree`
command to create a new branch that holds the code I'm interested in:

```bash
$ git subtree split -P provisions -b provisions-only
```

Here, `provisions` is the directory I want to create a new git repo out of and
`provisions-only` is the branch that will hold the code that's inside
`provisions` directory

Create a new directory outside the directory holding my fork of the repo and do
`git init`:

```bash
$ mkdir ~/repos/cccp-provisions
$ cd ~/repos/cccp-provisions
$ git init
```

And `git pull` from the fork to new directory:

```bash
$ git pull ~/repos/container-pipeline-service provisions-only
```

Here, `~/repos/container-pipeline-service` is the path to the forked repo that
I don't want to touch and `provisions-only` is the branch in that repo that
holds the code I'm interested in.

Whenever there's a change in the big repo (`container-pipeline-service`) and I
want to copy the changes to `cccp-provisions` repo, all I need to do is execute
the same `git subtree` command I used above in the big repo, and do `git pull
--rebase` in `cccp-provisoins` :

```bash
$ cd ~/repos/container-pipeline-service

$ git diff --unified=0
diff --git a/README.md b/README.md
index fd32557..fb14f4f 100755
--- a/README.md
+++ b/README.md
@@ -0,0 +1,2 @@
+Test
+
diff --git a/provisions/README.md b/provisions/README.md
index f956524..67f70dc 100644
--- a/provisions/README.md
+++ b/provisions/README.md
@@ -0,0 +1,2 @@
+Test
+

$ git add -A

$ git commit -m "Test"

$ git subtree split -P provisions -b provisions-only
Created branch 'provisions-only'
3dc47520e2d7a0bb603b6f0ac9dd60143f99f331

$ cd ~/repos/cccp-provisions

$ $ git pull --rebase ~/repos/container-pipeline-service provisions-only
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 3 (delta 2), reused 0 (delta 0)
Unpacking objects: 100% (3/3), done.
From /home/dshah/repos/container-pipeline-service
 * branch            provisions-only -> FETCH_HEAD
Updating d203d40..3dc4752
Fast-forward
 README.md | 2 ++
 1 file changed, 2 insertions(+)
```

What I did above was: modify two files; one inside the `provisions` directory
and one outside it; committed the changes it to my `master` branch and using
the `git subtree` command, created a `provisions-only` branch like we did
earlier. When I go to the `cccp-provisions` directory and do `git pull
--rebase`, git knows that only one of the two files modified in that commit is
of our interest and so it changes only one file:

```bash
Fast-forward
 README.md | 2 ++
 1 file changed, 2 insertions(+)
```

That was pretty cool for me! Now I can go ahead and walk-through the
provisioning bits and also make changes to it without affecting my fork of the
repo.
