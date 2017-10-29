---
layout: post
title: Lightning Component Unit Test - System Error
---

While working on Lightning components recently discovered that LEX APEX controllers are tricky to test. Mainly because of use `aura.redirect` or other `aura.` calls. If you try testing methods that use `aura.redirect` call you will get a System error like this,

```
Internal Salesforce Error: 173417772-1404 (604027281) (604027281)
```
that does not tell anything usefull to a developer. Tech Support has translated this for me to some Missing ORACLE connection context. Given the fact that Salesforce platform is built on giant ORACLE instance it is not surprisig. At times I feel that main reason Salesforce has limts is to protect that giant ORACLE instance from fail.
Anyways lets see how to resolve this issue with our test code.

Digging bit deeper is WHY? Main reason for this error is `aura.<>` is a JavaScript frontend library, but here we are testing APEX controller back end that has to interface with front end JS library. Any such hybridization of programming languages can bring complex problems to light and I personally would avoid such transitions. 

One simple way I found to avoid this error and provide effective test coverage is NOT to RUN `aura.redirect` during unit tests. This means making conditional code that is a bit ugly but in this case needed to move forward.

