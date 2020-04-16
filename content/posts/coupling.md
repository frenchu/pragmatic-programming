+++ 
draft = true
date = 2020-04-16T22:12:24+02:00
title = "Few words about coupling"
description = "Types of coupling"
tags = []
categories = ["design"]
series = []
+++

## What is coupling?

When you make a call/request in your code you it means coupled with a recipient. Coupling is a property which describes how much two objects are bound together. 
Tight coupling between objects means the bound is strong. On contrary, loosely coupled objects are not very bounded.
Coupling manifests on different levels: classes, packages, modules, services, systems, etc.

Is coupling something bad? To some extend it is good, because you have to couple things together to make them work. Pieces of code need to cooperate in order to build higer level structures.
On the other hand, if you couple your parts of code to much, you will end up with tangled code which is difficult to maintain, extend and evolve.

The caller and the recipient can be coupled in different ways. Let's see how. 

## Inheritance
First, most tight way of coupling is inheritance. Child class not only can call the methods from its parent, but also inherits the state from it.
Think about this. Making changes in parent class you can break easily child classes.

## Aggregation
Second way is aggregation where you execute the method on a delegate/member. In a good object oriented design typically you don't have access to state of the delegate. 
However, you are bounded to the particular implementation of the delegate. If you want to change the delegate class to different one, you will also need to change aggregating class.

## Using interface instead of concrete implementation
Aggregation type of coupling can be loosen when concrete implementation of delegate is replaced with an interface, ie. Dependency Inversion Principle is applied.
Aggregating class don't know much details about the delegate. It knows its behavior, but not the exact type.
This property allows you to change the implementations, without making any changes in aggregating object. And this is basically how Strategy pattern works in practice.
Temporal coupling is some form of this and it manifests for example in REST APIs or RPCs.

## Message passing
The last way of coupling, massage passing, is most lose. Systems based on actor model or message broker utilize this approach. When you send a message you don't need to know anything about recipient, 
you don't even know many recipients can read your massage. Using this approach it is very easy to extend your system by only adding new receipients/consumers.

