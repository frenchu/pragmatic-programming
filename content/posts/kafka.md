+++ 
draft = true
date = 2020-04-16T22:35:59+02:00
title = "Kafka"
description = "Apache Kafka"
tags = [
    "messaging", 
    "data store",
    "service integration"
]
categories = [
    "backend"
]
series = [
    "service integration"
]
+++

What problem Kafka or any other messaging system solves in a first place?

Let me elaborate more on coupling first. <- zdanie do wywalenia

In the era of microservices Kafka become a standard. It helps to decouple systems or services nicely from each other. Comparing to traditional Enterprise Service Bus 
(sometimes called Message Oriented Middleware) like Tibco or IBM WebSphere MQ, Kafka is fairly simple.
ESBs more often contain inside a lot of logic that can be applied to messages e.g. filters, selectors etc.
While Kafka leaves all message-processing logic to consumers. This shifting towards smart consumers/endpoints is one of microservices common characteristic as Martin Fowler states in his article[1].

* producer
* consumer
* point-to-point model
* publish-subscribe model

Message brokers not only decouples different parts of the system, but also make it more available and resilient. Let's imagine the message is sent by a producer to a consumer, but second one is down. 
Message broker can hold that message and pass it when the consumer is back on again. 