---
layout: series_index
title: "The 'dependency cycle' series"
seriesIndexId: "Dependency cycles"
seriesIndexOrder: 11
aliases:
  - /series/dependency-cycles.html
date: 2020-01-01
---

One of the most common complaints about F# is that it requires code to be in *dependency order*. That is, you cannot use forward references to code that hasn't been seen by the compiler yet.

In this series, I discuss dependency cycles, why they are bad, and how to get rid of them.
