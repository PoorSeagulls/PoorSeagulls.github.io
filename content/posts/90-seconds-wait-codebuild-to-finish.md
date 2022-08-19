---
title: "90 Seconds, wait codebuild to finish"
author: "Sabato Luca Guadagno <o0th@pm.me>"
date: 2022-08-19T08:59:10+02:00
draft: false
tags: [90seconds, nushell, aws, aws-codebuilds]
---

A couple of years ago, I came across an article regarding a Russian developer
who had one brilliant rule: if anything requires more than 90 seconds, write
a script to automate it.


There is where my mind went yesterday when I was executing a series of
operations inherited from a "state of the art" service currently running in
our production environment: run a build, wait for it to finish, and run a
second build.

This operation takes me, on average, 10 to 15 minutes to complete. Of course,
during this time, I can, and I do, something else, and most of the time, I
forget to run the second build. So to avoid furthermore this waste of time,
I decide to automate this process.

Since a blog post is not where to share a complete script that nobody cares to
run except me, I will share the only challenging problem I faced: waiting for
an AWS CodeBuild Project to be done.

```shell
#!/usr/bin/env nu

def await [id] {
  let batch = (
    aws codebuild batch-get-builds --ids $id
    | complete
  )

  if ($batch | get exit_code) != 0 { exit 1 }

  let status = (
    $batch
    | get stdout
    | from json
    | get builds
    | get 0
    | get buildStatus
  )

  if $status == 'IN_PROGRESS' {
    sleep 10sec
    await $id
  }
}

def main [project] {
  let build = (
    aws codebuild start-build --project-name $project
    | complete
  )

  if ($build | get exit_code) != 0 { exit 1 }

  let id = (
    $build
    | get stdout
    | from json
    | get build
    | get id
  )

  await $id
}
```

References
- [Now that's what I call a hacker](https://www.jitbit.com/alexblog/249-now-thats-what-i-call-a-hacker/)
- [nushell](https://github.com/nushell/nushell)
