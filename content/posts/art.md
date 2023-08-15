---
title:  "Deny async task resubmission pattern"
date: 2023-08-15
categories:
  - development
tags:
  - spring
  - async
---

This small recipe documents how you can easily deny a Spring async task resubmission, if an execution is already in progress.
It can be considered as a crude means of throttle control. 
In my case, I used this method to ensure that a batch task that is meant to be executed daily and is triggered from a rest endpoint, will not be resubmitted or queued by mistake when a task is already in progress.
A resubmission might cause undesirable effects on the database if two batch jobs run at the same time,
or incur unneeded load if the job gets queued and runs after the first has finished.
Obviously, there are quite a few ways to enforce this, but this recipe provides an almost declarative method to achieve this.

**DISCLAIMER**: This method will not work if two or more service instances exist and the rest endpoint is invoked through a load balancer.

## Async task

How to submit async tasks with Spring  boot

## Executor definition

How to configure the async task execute

## Duplicate submission

What should happen if you resubmit the task
