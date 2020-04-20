+++
aliases = ["kotlintest-pitfalls"]
categories = ["backend", "qa"]
date = 2020-03-14T20:45:41Z
description = "Common kotlintest pitfalls and how to deal with them"
images = ["/images/sailing-ship.jpg"]
publishDate = 2020-03-14T00:00:00Z
series = ["Kotlintest adoption"]
tags = ["kotlintest", "kotlin", "jvm"]
title = "Kotlintest adventures"

+++
_Common pitfalls and how to deal with them_

{{< figure alt="Kotlintest mutiny - pirate skull inviting only true pirates" src="/images/pirates-only.jpg" caption="Photo by Mateusz Dach on Pexels" >}}

## Intro

Our team currently started cruise to develop the next version of ‘the API' and a new set of services emerged during that journey. We have a free hand to sail to the open waters of technologies ocean, so we can choose the best fit for our project. Our ship wharfed to an island named Kotlin language. Very quickly we discovered Kotlintest library on this land. When fighting the battle of writing test cases to our production code, we had a mutiny incited by Kotlintest. In this article, I want to describe the course of the rebellion and how our brave team suppressed it.

## Why Kotlintest at all?

In terms of writing tests in Kotlin you have several test libraries to choose. The most common are Spek, JUnit5 (preferably with some assertion library like AssertJ or HamKrest) and of course Kotlintest. As Dear Reader probably already anticipated, in our project we decided to use Kotlintest. The authors of the library are inspired by the test library for Scala - Scalatest. Our team promotes functional programming, so do Kotlintest team. The library has many useful features like different styles of writing test specifications, soft assertions or data-driven testing. It is also quite popular in the community, actively developed and well supported.

## Minor but persistent

There are a few annoying things about Kotlintest. First of all, integration with IntelliJ - the IDE which we use on a daily basis, is not so good. It is impossible to execute a test case of one's choice from test specification. I tried the official IntelliJ plugin with ~~no avail~~. Thanks to my project manager I realized that the plugin to run things smoothly requires some additional configuration. And it wasn't straightforward for me. To set it up, first install a plugin named Kotlintest. After that, if you are using Gradle in IntelliJ preferences under `Build, Execution, Deployment > Build Tools > Gradle` select `Run tests using: IntelliJ IDEA`. Then upon running a test, you can choose to run it with the Kotlintest plugin. If it doesn't work try to use Restart/Invalidate cache option.

Before the moment I started using Kotlintest plugin, I had found in the documentation mention about prefixing test case with "f:". It should do the trick as it is done in popular JavaScript libraries. Prefix "f:" means focused here. Unfortunately this tip doesn't work well. Neither standard IntelliJ runner nor Gradle will execute any tests if I try to focus any test case. IntelliJ informs that there are no tests received.

{{< gist frenchu ed723987dabb9c52b4af80cdb8be6074 "FocusedStringSpec.kt" >}}

Surprisingly bang tests with exclamation mark "!" works. It excludes the test case from running. So if want to focus some test case I just exclude all the others. :stuck_out_tongue_winking_eye:

{{< gist frenchu ed723987dabb9c52b4af80cdb8be6074 "BangStringSpec.kt" >}}

Again this feature works perfectly fine with the Kotlintest plugin. This can be solved also by adding [Kotlintest Gradle plugin](https://plugins.gradle.org/plugin/io.kotlintest) to your project. Please compare the [GitHub issue](https://github.com/kotest/kotest/issues/605).

## Duplicated test name

There are more issues that will lead to a situation where tests should be executed, but they don't. It is a serious problem when a test certainly should fail. We observed that behavior when the name of the test case method was duplicated by mistake. Any test case of the specification sadly didn't run. It caused the whole build was green, but actually it didn't. The problem was masked and not easy to spot.

{{< gist frenchu ed723987dabb9c52b4af80cdb8be6074 "DuplicatedStringSpec.kt" >}}

The root cause of the issue was the tiny dependency hell we had in the project. :wink: To be more precise, Kotlin depends on arrow-core-data in version 0.9.0 while our project uses arrow 0.10.x. It leads to the replacement of the arrow-core-data version on which Kotlintest is dependent.

{{< gist frenchu ed723987dabb9c52b4af80cdb8be6074 "build.gradle.kts" >}}

## When Spring eventually comes...

Similar issue we could encounter while running integration tests with Spring. When the creation of the Spring context is failed then none of the test cases is executed from the test. The same as in the previous issue this leads to falsely green build and it is hard to notice. The cause of the issue was the same as in the [Duplicated test name](#duplicated-test-name).

One digression on testing with Spring. We don't like much autowiring lateinit var variables approach. Mutable state is not something we want to have in our functional code even if it is in tests. Hopefully Kotlintest allows us to wiring through constructor of test spec. To enable that Spring project extension needs to be added to the project. This approach has some drawbacks although. The project listeners/extensions are executed before test listeners. So if you have any test setup that the Spring context creation depends on, you cannot put it in the test listeners. In that case we use Spring test context configuration with post construct and post destroy lifecycle methods. In the last resort, one can go back to `autowired` `lateinit` vars.

## Strange behavior

Talking about test listeners there is one thing worth mentioning. If you take advantage of BehaviorSpec, most likely you want to run test listener methods before and after _whole_ given/when/then test case. Use `isTopLevel` method, otherwise `afterTestCase` and `beforeTestCase` methods will execute not only on `given`, but also on `when` and `then` blocks.

{{< gist frenchu ed723987dabb9c52b4af80cdb8be6074 "IsTopLevelSpec.kt" >}}

Another strange thing is the way how test results are reported in IntelliJ after running Gradle test task. Only "Then" names are listed on the report. So if you have several "Thens" labeled the same way in your Test specification, it is not easy to distinguish them. The problem can be solved with Kotlintest plugin. Running the tests using this plugin will show the nested structure of given/when/then on the report.

One can complain about the tree-like structure of BehaviorSpec. Indeed, it is a definite advantage of the library. Thanks to that Kotlin compiler itself can check the right order of clauses. In JUnit you can put labels or comments, but they don't have any meaning. While in Spock, a test library for Groovy, checking of labels order is added by the library.

## Are you still hiding something from me, Kotlintest?

Please be careful when you are going to use AssertSoftly. Normally all assertions in the assert softly block will be executed even if the first one fails. Dear reader can meet yet another concealed troubles there. Assert softly works well with Kotlintest matchers. But if you want to combine other testing tools like reactor-test for example, they most likely throw some kind of AssertionException when assertion fails. It will cause that assert softly block will be interrupted. If you don't always follow TDD completely you may not notice this "feature" in the first place.

{{< gist frenchu ed723987dabb9c52b4af80cdb8be6074 "AssertSoftlySpec.kt" >}}

To avoid that you probably want to implement your custom matchers. In your matcher you can catch AssertionException in the overridden test method. Then return boolean indicating success or failure of the assertion.

{{< gist frenchu ed723987dabb9c52b4af80cdb8be6074 "AssertSoftlyWithCustomMatcherSpec.kt" >}}

Another annoying thing is related to `.config` method usage. In behavioral tests, it is defined for `then` but not for `Then` method. I like starting test level names with capital letter since `when` is a reserved keyword in Kotlin. In order to compile your code, you need to put it in backticks: ``when``. So instead of `given`, ``when``, `then` one can use `Given`, `When`, `Then`. Except you want to add config to the test. In this case you have to use `Given`, `When`, `then` which looks silly.

{{< gist frenchu ed723987dabb9c52b4af80cdb8be6074 "ConfigOnThenSpec.kt" >}}

## Summary

Is our journey with Kotlintest over? I hope not. Probably we will discover even more issues, but despite its peculiarities Kotlintest is a promising library. The development is very active, and the releases are coming quite frequently. We optimistically see the future of the library. In the upcoming release library is changing the name from Kotlintest to Kotest. Beta versions for the 4.0.0 release are already available. In the next article I will describe the migration steps needed. And also I will check if the above issues have been fixed.

How about you? I strongly encourage you to check the library on your own. Maybe you have some other solutions to share? Feel free to post your comment below the article.

A complete project with code samples is available on my [GitHub](https://github.com/frenchu/kotlintest-pitfalls).

{{< figure alt="Kotlintest journey - sailing ship reaching a bay" src="/images/sailing-ship.jpg" caption="Photo by Pixabay on Pexels" >}}