---
title: wsl子系统重装
date: 2025-09-08 10:20:49
excerpt: 记录如何卸载重装wsl子系统
sticky: 
tags:
  - 子系统重装
categories:
  - [work,子系统重装]
author:
  - tang  
---

```powershell
# 卸载wsl子系统
wsl --unregister Ubuntu-24.04

#从tar包导入子系统
wsl --import Ubuntu-24.04 D:\linux\Ubuntu-24.04 D:\linux\Ubuntu-24.04.tar
```

