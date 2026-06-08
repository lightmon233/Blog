---
title: "Linux cp和mv命令 对于目录复制到目录的情况 的 所有情况示例"
date: 2024-11-15T22:12:20+08:00
draft: false
description: ""
---
{{< katex >}}


# `cp` 和 `mv` 命令的行为总结表

假设以下路径设置：

- **源路径**：`/nihao` 或 `/nihao/`
  - `/nihao` 包含文件和子目录：`file1`, `dir1/`, `file2`
- **目标路径**：`/nima` 或 `/nima/`

## 表格

### `cp` 命令行为

| **命令**                              | **目标路径存在？** | **最终路径结构**                     |
|---------------------------------------|--------------------|---------------------------------------|
| `cp -r /nihao /nima/`                 | 是                 | `/nima/nihao/`                       |
| `cp -r /nihao/ /nima/`                | 是                 | `/nima/file1`, `/nima/dir1/`, `/nima/file2` |
| `cp -r /nihao /nima`                  | 是（目录）         | `/nima/nihao/`                       |
| `cp -r /nihao /nima`                  | 否（文件/不存在）  | `/nima/`（重命名为 `/nima`）         |
| `cp -r /nihao/ /nima`                 | 是（目录）         | 同第一条 |
| `cp -r /nihao/ /nima`                 | 否（文件/不存在）  | `/nima/file1`, `/nima/dir1/`, `/nima/file2` |

### `mv` 命令行为

| **命令**                              | **目标路径存在？** | **最终路径结构**                     |
|---------------------------------------|--------------------|---------------------------------------|
| `mv /nihao /nima/`                    | 是                 | `/nima/nihao/`                       |
| `mv /nihao/ /nima/`                   | 是                 | `/nima/nihao/`                       |
| `mv /nihao /nima`                     | 是（目录）         | `/nima/nihao/`                       |
| `mv /nihao /nima`                     | 否（文件/不存在）  | `/nima/`（重命名为 `/nima`）         |
| `mv /nihao/ /nima`                    | 是（目录）         | `/nima/nihao/`                       |
| `mv /nihao/ /nima`                    | 否（文件/不存在）  | `/nima/`（重命名为 `/nima`）         |
