---
title: "Coroutines"
date: 2023-08-17T10:47:19+03:00
draft: true
---

# Kotlin Coroutines

* Are part of the Kotlin standard library but the `org.jetbrains.kotlinx:kotlinx-coroutines-core` provides additional usability functions
* Need a coroutines scope to run coroutines
* `runBlocking {}` creates a coroutine scope and runs everything included up to completion in the same thread
* `launch {}` creates a coroutine scope and runs the included lambda in the background. Returns a Job object that can be used to await termination of the coroutine or to cancel.
* `yield()` and `delay()` trigger a context switch
* To use `yield()` and `delay()` you have to mark a functions as `suspend fun`
* To run in different threads (so possibly in parallel instead of just concurrently), provide a coroutines context
  * `Dispatchers.Default`: 2 threads or number of processors
  * `Dispatchers.IO`: grows in size if threads
are blocked on IO and more tasks are created.
  * `Dispatchers.Main`: run tasks in the main thread
  * `asCoroutineDispatcher()`: Create a coroutine context from a JDK executor service

Calls to `await()` block the flow of execution but don’t block
the thread of execution

Inifinte sequences make use of continuations allowing the generation of an element to be processed as needed.