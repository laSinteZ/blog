---
title: How to start Google Chrome without experimental flags
tags:
  - Google Chrome
  - Chromium
creationDate: 1625482817
---

#  How to start Google Chrome without experimental flags

Today I tried to integrate macOS's Screen Time with Google Chrome. This can be achieved by visitihg [chrome://flags/#screentime](chrome://flags/#screentime) and enabling Screen Time flag. This worked wonders and blocked websites for me, as it would do in Safari. However, apparently browser broke and I wasn't able to interact with __any__ on __any__ webpage. I decided to disable this cool flag, but apparently settings page is a webpage too and it wasn't working.

Thankfully, there's a [--no-experiments](https://peter.sh/experiments/chromium-command-line-switches/#no-experiments) command line switch which starts Chrome without any flags enabled. In Terminal app, I wrote

```
open -a "Google Chrome" --args --no-experiments
```

However, I still needed to visit [chrome://flags/#screentime](chrome://flags/#screentime) and disable this flag myself.


To sum up, today I learned:
- There a lot of feature flags in Chrome, including integration with macOS Screen Time.
- If you've went too far and crashed your chrome, `open -a "Google Chrome" --args --no-experiments` could save the day.
