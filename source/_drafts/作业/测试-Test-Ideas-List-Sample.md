---
title: 测试 Test Ideas List Sample
date: 2017-04-11
categories:
  - 测试
tags:
  - Test Ideas List
---

# A Sample of Test Ideas List

Given a simple topic: **User Sign In**

And the specification goes:

+ User can use a passport **(username, password)** to sign in.
+ User can distinguish whether the username is exist or not. 

| ID   | Motivation                        | Operation (Input)                 | Expectation (Output)               |
| ---- | --------------------------------- | --------------------------------- | ---------------------------------- |
| 1    | Correctly Sign In                 | Correct passport                  | User's Active Token (200)          |
| 2    | Exist username but wrong password | Exist username but wrong password | "Wrong username or password" (403) |
| 3    | Non-exist username                | Non-exist username                | "No such user" (404)               |