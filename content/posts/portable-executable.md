---
layout: single
title: Portable Executables
date: 2025-12-21
classes: wide
---

Portable executables are composed of two main areas - a header and sections. The header provides general information about the portable executables.

```
|------------|------------|--------------------|
|            |            |  DOS Header        |
|            |            |  PE Header         |
|            |   Header   |  Optional Header   |
|            |            |  Data Directories  |
|            |            |  Sections Table    |
|  main.exe  |------------|--------------------|
|            |            |  Code              |
|            |  Sections  |  Imports           |
|            |            |  Data              |
|------------|------------|--------------------|
```

- https://www.youtube.com/watch?v=WIdkwzKV6Zk