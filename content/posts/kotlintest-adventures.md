+++
date = 2020-03-14T22:45:41+02:00
publishDate = 2020-03-14
title = "Kotlintest adventures"
description = "Common kotlintest pitfalls and how to deal with them"
tags = [
    "kotlintest",
    "kotlin",
    "jvm"
]
categories = [
    "backend",
    "qa"
]
series = ["Kotlintest adoption"]
aliases = ["kotlintest-pitfalls"]
+++

*Common pitfalls and how to deal with them*

## Intro

Our team currently started cruise to develop next version of ‘the API’ and new set of services emerged during that journey. 
We have free hand to sail to the open waters of technologies ocean, so can choose the best fit to our project. 
Our ship wharfed to island named kotlin language. Very quickly we discovered Kotlintest library on this island. 
When fighting the battle of writing test cases to our production code we had a mutiny incited by Kotlintest. 
In this article I want to describe the course of the rebellion and how our brave team suppressed it.

## Why Kotlintest at all?

In terms of writing tests in kotlin you have several test libraries to choose. 
The most common are Spek, JUnit5 (preferably with some assertion library like AssertJ or HamKrest) and ofcourse Kotlintest. 
As you probably already anticipated, in our project we decided to use Kotlintest. 
The authors of the library are inspired by test library for Scala - Scalatest. Our team promote functional programming, so do Kotlintest. 
The library has many useful features like different styles of writing test specifications, soft assertions or data-driven testing. 
It is also quite popular in the community, actively developed and well supported.

## Minor but persistent

There are few annoying things about Kotlintest. First of all, integration with IntelliJ - the IDE which we use on daily basis, is not so good. 
It is impossible to execute test case of ones choice from test specification. I tried official IntelliJ plugin with ~~no avail~~. 
Thanks to my project manager I realised that the plugin to run things smoothly requires some additional configuration.
And it wasn’t straightforward for me. To set it up, first install plugin named Kotlintest. 
After that if you are using gradle in IntelliJ preferences under `Build, Execution, Deployment > Build Tools > Gradle` 
select Run tests using: IntelliJ IDEA. Then upon running a test, you can choose to run it with the Kotlintest plugin. 
If it doesn’t work try to use Restart/Invalidate cache option.

Before the moment I started using Kotlintest plugin, I had found in the documentation mention about prefixing test case with “f:”. 
It should do the trick as it is done in popular JavaScript libraries. Prefix “f:” means focused here. Unfortunately this tip doesn’t work. 
Neither standard IntelliJ runner nor Gradle will not execute any tests if I try to focus some test case. 
IntelliJ informs that there are no tests received. Surprisingly bang tests with exclamation mark “!” works. 
It excludes the test case from running. So if want to focus some test case I just exclude all the others. :P 
Again this feature works perfectly fine with the plugin. 
This can be solved also by adding [Kotlintest gradle plugin](https://plugins.gradle.org/plugin/io.kotlintest) to your project. 
Please compare [GitHub issue](https://github.com/kotest/kotest/issues/605).

## Duplicated test name

There are more issues which will lead to situation where tests should be executed, but they don’t. 
It is a serious problem when a test actually should fail. 
We observed that behaviour when the name of the test case method was duplicated by mistake.
Any test case of the specification sadly didn’t run.
It lead to situation where the whole build is green, but actually it isn’t. The problem was masked and not easy to spot.

The root cause of the issue was the tiny dependency hell we had in the project. ;) 
To be more precise, kotlin depends on arrow-core-data in version 0.9.0 while our project uses arrow 0.10.x. 
It leads to replacement of arrow-core-data version on which Kotlintest is dependent.

## When Spring eventually comes...

Similar issue we could encounter while running integration tests with Spring. 
When creation of spring context is failed then none of the test cases is executed from the test. 
The same as in previous issue this leads to falsely green build and it is hard to notice. 
The cause of the issue was the same as in [Duplicated test name](#duplicated-test-name).

One digression on testing with Spring. We don’t like much autowiring lateinit var variables approach. 
Mutable state is not something we want to have in our functional code even if it is in tests. 
Hopefully Kotlintest allows to wiring through constructor of test spec. 
To enable that Spring project extension needs to be added to the project. This approach has some drawback although. 
The project listeners/extension are executed before test listeners. 
So if you have any test setup that spring context creation depends on you cannot put it in test listeners. 
In that case we use spring test context configuration with post construct and post destroy lifecycle methods. 
In the last resort one can go back to autowired lateinit vars.

## Strange behavior

Talking about test listeners there is one thing worth to mention. 
If you take advantage of BehaviorSpec, most likely you want to run test listener methods before and after *whole* 
given/when/then test case. Use isTopLevel method, otherwise afterTestCase and beforeTestCase methods will execute not only on given, 
but also on when and then blocks.

Other strange thing is the way how tests results are reported in IntelliJ after running gradle test task. 
Only Then names are listed on the report. So if you have several Thens labeled the same way in your Test specification, 
it it is not easy distinguish them. Problem can be solved with Kotlintest plugin. 
Running the tests using this plugin will show nested structure of given/when/then on the raport.

One can complain about tree-like structure of BehaviorSpec. Actually it is a definite advantage of the library. 
Thanks to that kotlin compiler itself can check the right order of clauses. 
In JUnit you can put labels or comments, but they don’t have any meaning. 
While in Spock, a test library for Groovy, checking of labels order is added by the library itself.

## Are you still hiding something from me, Kotlintest?

Please be careful when you are going to use AssertSoftly. 
Normally all assertions in the assert softly block will be executed even if the first one fails. 
Dear reader can meet yet another concealed troubles there. Assert softly works well with Kotlintest matchers. 
But if you want to combine other testing tools like reactor-test for example, 
they most likely throw some kind of AssertionException when assertion fails. 
It will cause that assert softly block will be interrupted. 
If you don’t always follow TDD completely you may not notice this “feature” at the first place. 
To avoid that you probably want to implement your custom matchers. 
In your matcher you can catch AssertionException in the overridden test method. 
Then return boolean indicating success or failure of the assertion.

Another annoying thing is related with `.config` method usage. 
In behavioral tests it is defined for `then` but not for `Then` method. 
I like starting test level names with capital letter since `when` is reserved keyword in kotlin. 
In order compile your code, you need to put it in backticks: `` `when` ``. 
So instead of `given`, `` `when` ``, `then` one can use `Given`, `When`, `Then`. 
Except you want to add config to the test, in this case you have to use Given, When, then which looks silly.

## Summary

Is our journey with Kotlintest over? I hope not. Probably we will discover even more issues, 
but despite its peculiarities Kotlintest is a promising library. 
The development is very active and the releases are coming quite frequently. 
We optimistically see the future of the library. 
In the upcoming release library is changing name from Kotlintest to Kotest. 
Beta versions for the 4.0.0 release are already available. In the next article I will describe migration steps needed. 
And also I will check if the above issues has been fixed.

How about you? I strongly encourage you to check the library on your own. Maybe you have some other solutions to share? Feel free to post your comment below the article.

Complete project with code samples is available on my [GitHub](https://github.com/frenchu/kotlintest-pitfalls).
