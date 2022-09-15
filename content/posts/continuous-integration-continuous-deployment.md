---
title: "Continuous Integration Continuous Deployment"
date: 2022-09-15T19:02:56+08:00
draft: true
tags: ["Continuous Integration", "Continuous Deployment", "DevOps"]
---
### DevOps
```
|---------|        |---------|  merge  |-------|          |--------------------------|
|local dev|------->| version |-------->| build |--------->|* security check          |
|---------| commit | control | trigger | agent | pipeline |* build / push artifact   |
    ^              |---------| webhook |-------| task     |* unit / integration test |
    |                                                     |--------------------------|
    |                                                                  |
|----------| analysis |------------| operation |----------------|      |release
| plan     |<---------| monitor    |---------->| staging        |<-----|
| action   |          | metrics log|           | production env |
|----------|          |------------|           |----------------|
```
