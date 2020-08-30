---
layout: post
title:  "How to use multiple Github accounts on a single machine?"
date:   2020-08-30 06:35:03 +0300
---
This problem comes up almost for every developer during his career. The reasons might be different: for example, you might have a personal account and corporate account, or you have a separate account for freelance, or maybe you wanna work from the shadow and make commits from someone's account.

![](/assets/images/multiple-github-accounts.jpg)
In this article, I will use authentication with GitHub via ssh and the solution is pretty straightforward - multiple ssh keys settings for different GitHub accounts. This case is taken from my own experience, so I'll show all the steps on Mac machine, but it shouldn't be complicated to repeat on Windows/Linux machine. For more detailed instructions, please refer to [Github's documentation](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh).

Let's imagine that we have 2 GitHub accounts. The first one is - *kovalc0mrade-personal* and the second one is - *kovalc0mrade-company*.

#### Step 1: Generate ssh keys
First of all, I'm gonna generate separate ssh keys with names *github_company* and *github_personal*:
```
$ ssh-keygen -t rsa -C "kovalc0mrade-personal@email.com"
$ ssh-keygen -t rsa -C "kovalc0mrade-company@email.com"
```
Each command will create two files, for example, *github_personal* and *github_personal.pub* for *kovalc0mrade-personal@email.com*.

#### Step 2: Add keys to ssh-agent
The *ssh-agent* is a program that works in the background and keeps your keys loaded into memory, so you don't need to enter your passphrase every time you need to use them.
```
$ ssh-add -K ~/.ssh/github_personal
$ ssh-add -K ~/.ssh/github_company
```

#### Step 3: Modify ssh config file:
Config file is located in *~/.ssh* folder, if it doesn't exist you can create it with `touch config` command.
Open file with `vim config` and paste:
```
Host github.com-kovalc0mrade-personal
  HostName github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/github_personal

Host github.com-kovalc0mrade-company
  HostName github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/github_company
```

#### Step 4: Add public ssh keys in Github
Open your GitHub *Settings*, navigate to *SSH and GPG keys* page and press button *New SSH key*, here you should paste content from file with `.pub` extension that was generated in *Step 1*.
```
$ cat ~/.ssh/github_personal.pub
$ cat ~/.ssh/github_company.pub
```

#### Step 5: Clone repositories to your local machine
Now it's time to clone repositories and configure them.
As was mentioned in the beginning, I'll use ssh links for this purpose.
```
$ clone git@github.com:kovalc0mrade-personal/your-repository-1.git
$ clone git@github.com:kovalc0mrade-company/your-repository-2.git
```
Since I have different emails, I have to update a local git config with commands.
```
// In your-repository-1 folder
$ git config user.email "kovalc0mrade-personal@email.com"

// In your-repository-2 folder
$ git config user.email "kovalc0mrade-company@email.com"
```
Remote origin should also be updated to use proper ssh key.
Get current remote origin command:
```
// In repository folder
// Output example: git@github.com:kovalc0mrade-personal/your-repository.git
git remote get-url origin
```
And set new origin with updated *github.com* to *github.com-kovalc0mrade-personal* or *github.com-kovalc0mrade-company*.
```
git remote set-url origin
```

And that's it, now you can push and pull without any worries about accounts.

This post isn't an exhaustive guide and doesn't contain all the answers to what/why/where questions, but I hope all the gaps can be easily covered by just googling.
