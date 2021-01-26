---
layout: post
title:  "Distributed Tracing for Microservice Architecture"
date:   2015-10-18 14:00:00
categories: programming
tags: microservice debugging go devops
---
What is distributed tracing? Distributed [tracing](https://opentracing.io/docs/overview/what-is-tracing/) is a method used to profile and monitor applications, especially those built using a microservices architecture. Distributed tracing helps pinpoint where failures occur and what causes poor performance.

Let’s have a look at a simple prototype. A user fetches information about a shipment from `logistic` service. `logistic` service does some computation and fetches the data from a database. `logistic` service doesn’t know the actual status of the shipment, so it has to fetch the updated status from another service `tracking` `tracking` service also needs to fetch the data from a database and to do some computation.

In the screenshot below, we see a whole life cycle of the request issued to `logistics` service:
{% include image.html src="https://habrastorage.org/getpro/habr/upload_files/0ac/d6c/c78/0acd6cc7800f3b5b5cddab7b9c36bab6" %}

1. GET request hits endpoint `/shipment/:idin` the service `logistics`
2. The service does some internal processing that takes 10 ms (span `logistics processing`).
3. The service fetches data from the database for 30 ms (span `SQL select`).
4. `logistics` fetches the status of the shipment from another service tracking by sending a GET request to endpoint `/tracking:/:id` (span `tracking`). We have detailed information about this part of the journey as well, i.e. how much time `tracking` service used for the processing or how much time the service used for the database operation.

From the screen below, we can see that the request was successful and returned 200.
{% include image.html src="https://habrastorage.org/getpro/habr/upload_files/5b0/52e/c9e/5b052ec9eaaa4b41d520080dae8415d9" %}

It is especially handy as we didn’t log the response code or created the metadata explicitly: it comes for free from the tracing middleware.

Let’s say we want to profile our logistics service. As we can see, we spend most of the time on the database operation. It would be helpful to see the exact query we issued:
{% include image.html src="https://habrastorage.org/getpro/habr/upload_files/f9a/8c3/1d9/f9a8c31d98dd0fd61f6e1fcf436324ed" %}

Now we can check whether the query was optimal and which tables it affected. The span contains the information about the caller, including the line number and file name, so it is easy to navigate in the source code.

Speaking in general, we can profile any part of the code be it a module or a single function. Such data helps to spot the bottlenecks and make informed decisions about the optimization.

Tracing also automatically creates a snapshot of our architecture (DAG):
{% include image.html src="https://habrastorage.org/getpro/habr/upload_files/fcf/ee6/7a0/fcfee67a083984aa3b1f396e3f3bc0fa" %}

Obviously, the picture above doesn’t look that impressive. But what happens when we have not 2 services, but 30? Let’s say a user logs in and we need to fetch the information from services such as `users`, `settings`, `billing`, etc. In that case, this diagram would be very handy to understand the actual interactions between our services. The tracing would also reveal which of our services takes the most time or which one is the least reliable.

Tracing also helps to debug the problems in the microservice architecture. Let’s say each of our microservices is up and behaves well on an individual level. However, we have a problem on a system-wide level. I.e. there is a particular combination of input and output data between the microservices that causes a problem. In that case, individual logs from each of the microservices won’t reveal a problem. We need to tie all the logs from the affected microservices and see the whole picture. Tracing is the right instrument to solve such kind of problems.

## Distributed Tracing APIs

### OpenTracing

What is OpenTracing? It’s a vendor-agnostic API to help developers easily instrument tracing into their codebase. This means that if a developer wants to try out a different distributed tracing system, then instead of repeating the whole instrumentation process for the new distributed tracing system, the developer can simply change the configuration of the Tracer.

Most mental models for tracing descend from Google’s Dapper paper. OpenTracing uses similar nouns and verbs ([source](https://opentracing.io/docs/overview/)):
{% include image.html src="https://habrastorage.org/getpro/habr/upload_files/748/8c3/a59/7488c3a598b392cb0140c747cf05071e" %}
