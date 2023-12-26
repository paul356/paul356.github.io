---
layout: default
title: Paper Digest of September 2023
tags: [Database, AI4DB]
nav_order: {{ page.date }}
---

| Title                                                | Authors                            | Synthesis                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | Publisher | Keywords |
| QD-Tree Learning Data Layouts for Big Data Analytics | Zongheng Yang, Rajeev Acharya, etc | This paper introduces a method called query-data routing tree, aka QD-Tree, to partition database tables. QD-Tree uses the predicates from query workloads to cut the data space into a tree structure. Every intermediate node in the tree represents a predicate. The tuples statisfy the predicate go to the left child. And the tuples against the predicate go to the right child. This tree is built to maximize the skipping ability for the given queries. And this paper introduces two methods to build this tree. One is a greedy method. The other is based on deep reinforcement learning. It can be proved that QD-Tree is submodular because every child node has a diminishing gain compared to its parent. | SIGMOD 20 |          |
