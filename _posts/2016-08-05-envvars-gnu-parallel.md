---
layout: post
title: Exporting Environment Variables per-process in GNU Parallel
published: true
---

## The Problem

I want to have different environment variables for each process/job in GNU Parallel. When run locally, each process in GNU
parallel will pick up the current environment variables, which means getting different env vars per-process was more complicated.
Attempts 1-3 all were using environment variables as the command input (stored per-line in a file)

Space delineated with `parallel 'export {}'` fails, created processes with `a=1\ b=2` in the environment variables. 

Contents of `test_envs`:

```bash
a=1 b=2
a=3 b=4
```

Example:

```bash
[astraw@helios parallel]# cat test_envs | parallel --verbose --tagstring="Job #{#}:" 'export {}; echo a=$a - b=$b'
export a=1\ b=2; echo a=$a - b=$b
export a=3\ b=4; echo a=$a - b=$b
export a=5\ b=6; echo a=$a - b=$b
Job #1: a=1 b=2 - b=
Job #2: a=3 b=4 - b=
Job #3: a=5 b=6 - b=
```

Note how `b` is not set in any of the processes. Semi-colon delineated environment variables also fails for the same reason. 

## The solution

Create properties files! 

If you want to have X subprocesses, create X env files, with the desired environment variables space-separated on the first line. 
You can use the job # to access files-per-process. 

`env_1`:

```bash
a=1 b=1
```

`env_2`:

```bash
a=2 b=2
```

And so on. Then, when you form your parallel command, include the following at the start:

`'export $(cat env_{#})'`

Final example:

```bash
seq 1 10 | parallel --tagstring="Job #{#}: " 'export $(cat env_{#});echo a=$a - b=$b'
```

Will output:

```bash
Job #1:         a=1 - b=1
Job #2:         a=2 - b=2
Job #3:         a=3 - b=3
Job #4:         a=4 - b=4
Job #5:         a=5 - b=5
Job #6:         a=6 - b=6
Job #7:         a=7 - b=7
Job #8:         a=8 - b=8
Job #9:         a=9 - b=9
Job #10:        a=10 - b=10
```

Success!