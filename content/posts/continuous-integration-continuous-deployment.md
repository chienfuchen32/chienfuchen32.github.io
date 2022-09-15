---
title: "Continuous Integration Continuous Deployment"
date: 2022-09-15T19:02:56+08:00
draft: true
tags: ["Continuous Integration", "Continuous Deployment", "DevOps"]
---
### DevOps concept & practice
```
|---------|        |---------|  merge  |-------|          |--------------------------|
|local dev|------->| version |-------->| build |--------->|* security check          |
|---------| commit | control | trigger | agent | pipeline |* build / push artifact   |
    ^              |---------| webhook |-------| task     |* unit / integration test |
    |                                                     |--------------------------|
    |                                                                          |
|---------------------| analysis |------------| operation |----------------|   |release
| * next release plan |<---------| monitor    |---------->| staging        |<--|
| * issues action     |          | metrics log|           | production env |
|---------------------|          |------------|           |----------------|
```
