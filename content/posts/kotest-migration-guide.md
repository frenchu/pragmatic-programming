+++
title = "Kotest 4.0 migration guide"
description = "What's changed in the new Kotest version"
date = 2020-04-11
tags = [
    "kotlintest",
    "kotest",
    "kotlin",
    "jvm"
]
categories = [
    "backend",
    "qa"
]
series = ["Kotlintest adoption"]
aliases = ["kotlintest-migration-guide"]
images = ["/images/man-on-mountain-road.jpg"]
+++

_What's changed in the new version_

{{< figure alt="Migration to Kotest - a man on mountain road riding a bike towards rising sun" src="/images/man-on-mountain-road.jpg" caption="Photo by Pixabay on Pexels" >}}

## Intro

In the [Kotlintest adventures]({{< ref "posts/kotlintest-adventures.md" >}}) post
I described how my team solved different issues faced while implementing test cases in Kotlintest,
the testing library for Kotlin. Recently there was a major release of that library.
Let's see what has been changed and what steps to take in order to make use of the newest version of the library.

## What's your name, Kotlintest?

The most obvious change in the library is its name. The name was changed to Kotest, because of too much similarity with JetBrains `kotlin.test` package.

I started the migration from updating project dependencies. The current version of the library is 4.0.3. In addition to changing group name and module name, I had to add one more dependency for core assertions. Moreover, module names are suffixed with "-jvm".

{{< gist frenchu 5742f93a20c8c68d60c14b5f04bf6b58 "build.gradle.kts" >}}

Please bear in mind that if you are using Kotlintest Gradle plugin, its name currently is the same as before.

## Where the heck are all the assertions and specs?

Kotest creators restructured the project quite a bit. A lot of classes and functions changed their locations in packages. So the next step of the migration is to fix the imports. Spec types now resides in `io.kotest.core.spec.style` and functions like `should`, `shouldBe`, `shouldNot` were moved to `io.kotest.matchers` package. While `assertSoftly` can be found in a package named `io.kotest.assertions`.

I think what you can do to deal with it quickly, is to find and replace in the project scope:

    io.kotlintest.specs -> io.kotest.core.spec.style
    io.kotlintest.should -> io.kotest.matchers.should
    io.kotlintest.assertSoftly -> io.kotest.assertions.assertSoftly
    io.kotlintest.TestCase -> io.kotest.core.test.TestCase

For sure there are more changes like these, but for me it was enough.

## Project config

In the Kotest 4.0 we don't need to put project configuration class in the special package like it was before.
Instead of having project config in `io.kotlintest.provided` we can now create a configuration class wherever we want. Kotest will automatically scan the classpath to find it. An important note is `ProjectLevelConfig` class has been removed. I used a more general `Extension` interface to fill the gap.

{{< gist frenchu 5742f93a20c8c68d60c14b5f04bf6b58 "ProjectConfig.kt" >}}

## Last but not least

Finally, I needed to update the Kotlin language version. There was a problem in the runtime. The definition of the class `kotlin.time.MonotonicTimeSource` could not be found. Kotest is using experimental Kotlin features for profiling and measuring the time of test execution. I updated to the latest version of Kotlin language, and it fixed the issue.

## What about fixes?

I didn't have a chance to take a closer look, but I've noticed that the arrow dependency issue our team had to deal with has been now resolved. :tada: