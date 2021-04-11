---
layout: single
author: Piotr Polak
title: "Best practices when maintaining SDK-friendly OpenAPI contracts"
read_time: true
toc_sticky: true
toc: true
date: 2021-04-08 17:52:56 +0200
author_profile: true
---

This document is a followup to [API First and code generation done right]({{ site.baseurl }}{% link _posts/2021-04-02-api-first-and-code-generation-done-right.markdown %}).

In order to make the code generation tools adoption successful it is very important to maintain high flexibility and
great attention to backward compatibility of the OpenAPI contact changes. An unsuspicious modification in the contract
metadata might break existing code even though the HTTP API contract is untouched.

## Rules for developing SDK-friendly OpenAPI contracts

- Respect Design First principle — think about all the API names, DTO names, data structures, operationIds, and
  parameter list upfront.
- When designing an API try to make it reusable, assume that your services will be consumed by multiple clients such as
  web applications, mobile clients, or even applications that you are not yet aware of.
- Treat **contracts as contracts** — a small change in the OpenAPI definition might lead to massive changes in the
  client code. Think of it as interface/binary compatibility. If you wouldn’t change your library’s interface method
  signature, you probably wouldn’t like to change the endpoint definition as both would lead to broken code at the
  library/SDK clients.
- Assign operationId as if they were **repository** methods and don’t change it over time. Whatever you define as
  operationId will become the SDK method name. For example: getUsers, getUser, saveUser, deleteUser.
- Make sure operationIds are unique to make sure client SDK can be built for any language (some languages do not support
  method overloading).
- Assign DTO names according to the business domain. Try to apply the same naming convention as if DTOs would be your
  business entities — this is not always possible, especially with some aggregate objects. Skip all the DTO-like
  prefixes as these can be added by the code generator.

  Example DTO names: User, PhoneNumber, Cart.
- Generate SDKs often and see if you find the generated code developer-friendly.
- Consider writing tests using your SDKs — if your tests fail to compile after a definition change, they will probably
  make a bad impression on your clients also.
- Provide good examples of the request and response payloads. The payload values should ideally satisfy all the
  validation constraints so that they can be used during demo. Keep response payloads as close to production values as
  possible as these can be used to generate your service stub/mock.