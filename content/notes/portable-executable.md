---
title: Portable Executables
draft: true
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