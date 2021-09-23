---
title: `mockClear` and `mockReset` are very different functions in Jest
tags:
  - jest
creationDate: 1632247155
---

#  `mockClear` and `mockReset` are very different functions in Jest

According to documentation, `mockFn.mockReset()` does also remove the provided mock implementation. Be cautious with your `beforeEach` `mockReset()` :)

