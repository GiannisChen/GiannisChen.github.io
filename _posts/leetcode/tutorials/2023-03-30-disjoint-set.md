---
title: 并查集
categories: [LeetCode, LeetCode Tutorial]
tags: [leetcode, golang, disjoint set]

toc: true

math: true
mermaid: true
pin: false

img_path: "/assets/img/posts/2023-03-30-disjoint-set/"
render_with_liquid: false
---

# 并查集（Disjoint-Set Data Structure）

## 介绍

![img](disjoint-set.svg){: w="400"}

- **并查集**是一种用于管理元素所属集合的数据结构，实现为一个森林，其中每棵树表示一个集合，树中的节点表示对应集合中的元素，本质上是一棵棵父亲树。

  ```go
  type dsu struct {
      pa []int
  }
  ```

- **并查集**支持两种操作：

  - **合并**（`union`）：合并两个元素所属集合（合并对应的树）：

    ![img](disjoint-set-merge.svg){: w="400"}

    ```go
    func (d *dsu) unite(x int, y int) {
        d.pa[d.find(x)] = d.find(y)
    }
    ```

  - **查询**（`find`）：查询某个元素所属集合（查询对应的树的根节点），这可以用于判断两个元素是否属于同一集合：

    ![img](disjoint-set-find.svg){: w="300"}

    ```go
    func (d *dsu) find(x int) int {
        if d.pa[x] == x {
            return x
        }
        return d.find(d.pa[x])
    }
    ```

  - **查询时路径压缩**：

    ![img](disjoint-set-compress.svg){: w="400"}

    ```go
    func (d *dsu) find(x int) int {
        if d.pa[x] == x {
            return x
        }
        d.pa[x] = d.find(d.pa[x])
        return d.pa[x]
    }
    ```

- 快速合并图的不同的独立部分
- 判断两个部分是否连通

## 并查集的例题

| ID   | LeetCode 题号 | 描述 | 思路 |
| ---- | ------------- | ---- | ---- |
| 1    | [6365. Minimum Reverse Operations](https://leetcode.cn/problems/minimum-reverse-operations/) | 最少的翻转操作 | 利用并查集剪枝 |

| 题                        | 题   | 题   | 题   |
| ------------------------- | ---- | ---- | ---- |
| 305. Number of Islands II |      |      |      |
