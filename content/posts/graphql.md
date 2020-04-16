+++ 
draft = true
date = 2020-04-16T22:31:06+02:00
title = "GraphQL"
description = "GraphQL"
tags = [
    "service communication", 
    "service integration"
]
categories = [
    "backend"
]
series = [
    "service integration"
]
+++

## Intro

Creators of GraphQL want to it become a standard for the whole Internet in the near future. It originates from Facebook and quickly gain a lot o popularity.
But what actually GraphQL is? What does it offer comparing to REST style?

## Modes

* query
* mutation
* subscription

## Benefits

Query schemas allows to chose only the properties you really need and reduce the amount of data pushed through the wire. Small pause here...
Recently, I observed there is a quite strong trend in the IT industry to develop solutions utilizing hardware resources better. I'm not sure why.
Is it because of growing costs of cloud computing? Or maybe care of saving natural environment and reducing carbon footprint is the reason? I don't now.
Just to give you a few examples. One of them is GraphQL, obviously. Another one is Micronaut[1], Java framework competing with Spring Boot in making ease of microservices development.
However, Micronaut apps consumes less memory and starts much faster comparing to similar Spring Boot apps[2]. 
Most astonishing example is GraalVM, a new virtual machine based on Oracle HotSpot allowing to run blazingly fast programs written in almost all popular languages. End of digression.