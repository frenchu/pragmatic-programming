+++ 
draft = true
date = 2020-04-16T22:37:10+02:00
title = "GRPC"
description = ""
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

gRPC stands for Google Remote Procedure Call. RPC allows to call remote code as a local function or procedure. The idea of RPC is not new, some of you probably remember COBRA or Java RMI.
It sounds tempting to execute some code on a remote server in the same way as regular function. RPC tries to hide from the user fact that it is actually a remote call with all consequences.
One can be easily deceived by fallacies of distributed computing[1]. Ofcourse, In this article I don't want deter anybody from using gRPC.
In my opinion, it is worth to see what this technology offers and what are the benefits in comparison to traditional REST API or more moder GraphQL approach.

## Schemas