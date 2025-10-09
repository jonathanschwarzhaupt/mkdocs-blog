---
date:
  created: 2025-10-08
tags:
  - Linux
  - Bash
  - Behind the scenes
categories:
  - Linux
---

# File Expansion in Bash

Something that we all are familiar with and use quite often are so-called "wildcard" characters such as `*`. When I found that I could make use of wildcards in bash, I did not think much of it. "Of course, it simply recognizes that I mean all elements". But what is the mechanism behind that? <!-- more --> Is there more to it than I thought and is it an important mechanic to how the shell functions under to hood?

## What is expansion and why is it important?

Bash actually performs several substitutions before executing the command. An example of this is the wildcard character `*`. In simple terms, we enter something, that something is expanded into something else which is then executed.

An example:

Let's take the `echo` command which prints its input to stdout. When we run `echo this is a test` it prints "this is a test". But when we run `echo  *` then it prints the contents of the current directory, much like `ls` does.

This illustrates that the shell does something with the `*` character, expanding it to list every file, before sending the command off to `echo`.

There are different types of expansions we can encounter in Linux, and its uses and applications are wide spread, making the shell feel magical if not properly understood.

Let's take a short look at each of the types of expansions.

## Pathname expansion

_Pathname expansion_ is the mechanism by which the wildcard character works.

We can use `echo D*`, `echo *s`, or even a pattern like `echo [[:upper:]]*`. What do you think these would print to stdout, assuming a "default" home directory?

* `echo D*` -> Desktop Documents; All elements starting with an uppercase d
* `echo *s`-> Documents, Pictures, Templates, Videos; All elements ending with s
* `echo [[:upper:]]` -> Desktop Documents Music Pictures Public Templates Videos; All elements beginning with an uppercase character

## Tilde expansion

The tilde (`˜`) character has a special meaning. It _expands_ into the home directory of the specified user or the current user if none specified.

Thus, if we do `echo ˜` it prints for example `/home/jonathan`

## Arithmetic expansion

Did you know that you can use the shell like a calculator? Well, _expansion_ allows us to do so.

Arithmetic expansion uses this format: `$((expression))` where expression can consist of arithmetic operators (`+ - * / % **`).

Thus, we can do cool things like `echo $((2 + 2))` and you will see 4 as the output. More complex calculations are also possible: `echo $(((5**2) * 3))` giving you 75.

## Brace expansion

We can leverage a pattern within braces to create multiple text strings from it.

Let's say that we want to create folders of the format YYYY-MM over two years. Instead of creating 24 folders manually, we can use _expansion_:

`mkdir {2023..2025}-{01..12}`

This will neatly create folders like `2023-01, 2023-02, 2023-03`and so forth. Notice also how we have leading zeros. This is a feature introduced in bash version 4. If we want more than one leading zero we can simply use something like `{001..12}` which would give us 2 leading zeroes.

## Parameter expansion

In essence, this is about creating and using variables on the fly. A scenario most often useful in scripting situations. We can use _expansion_ to obtain the value of a parameter (i.e. variable) like so: `echo $USER`. If the parameter is set, we will see its value output on the terminal.

But, as opposed to other commands that might indicate that something was not found, here we simply see nothing returned. Try for example: `echo $SUER`.
