---
layout: default
title: Paper Digest of December 2023
tags: [Reinforcement Learning, QD-Tree]
nav_order: {{ page.date }}
---

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">Title</th>
<th scope="col" class="org-left">Authors</th>
<th scope="col" class="org-left">Synthesis</th>
<th scope="col" class="org-left">Publisher</th>
<th scope="col" class="org-left">Keywords</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">Neural Packet Classification</td>
<td class="org-left">Eric Liang, Ion Stoica</td>
<td class="org-left">This paper proposes using RL to construct a decision tree for packet classifiers, and it shows how to model formulate the MDP for constructing the decision tree.</td>
<td class="org-left">SIGCOMM 2019</td>
<td class="org-left">Reinforcement Learning, Decision Tree</td>
</tr>


<tr>
<td class="org-left">Detect, Distill and Update: Learned DB Systems Facing Out of Distribution Data</td>
<td class="org-left">Meghdad Kurmanji, Peter Triantafillou</td>
<td class="org-left">This paper presents a learning framework called DDUp for detecting OODs and updating learned database components. A statistical test is used to test the hypothesis that the new data obey the same distribution as the learned model. Knowledge Distillation is used to transfer knowledge from the old model to the new model if the hypothesis is rejected.</td>
<td class="org-left">SIGMOD 2023</td>
<td class="org-left">Knowledge Distillation, Transfer Learning</td>
</tr>


<tr>
<td class="org-left">From Large Language Models to Databases and Back - A discussion on research and education</td>
<td class="org-left">Sihem Amer-Yahia, .etc</td>
<td class="org-left">A panel discussion on LLM and database research. They discussed about the potential impact of LLM towards databases, and what are advantages and limitations of incoperating LLM in databases.</td>
<td class="org-left">DASFAA 2023</td>
<td class="org-left">LLM, Database, ChatGPT</td>
</tr>


<tr>
<td class="org-left"><b>Fine-grained Partitioning for Aggressive Data Skipping</b></td>
<td class="org-left">Liwen Sun, Michael J. Franklin, Sanjay Krishnan, Reynold S. Xin</td>
<td class="org-left">This paper presents a feature based partitioning framework. A number of features are selected to calculate a feature vector for each tuple. The conjunction of all feature vectors in a block is used for data skipping. And data are partitioned to maximize the data skipping gain. The contributions are (1) a workload analyzer, which generates a set of features from a query log, (2) a partitioner, which computes a blocking scheme by solving a optimization problem, (3) a feature-based block skipping framework used in query execution.</td>
<td class="org-left">SIGMOD 2014</td>
<td class="org-left">Partitioning, NP-Hard</td>
</tr>


<tr>
<td class="org-left">Small Materialized Aggregates: A Light Weight Index Structure for Data Warehousing</td>
<td class="org-left">Guido Moerkotte</td>
<td class="org-left">This paper shows how to speed up data warehouses with summaries called Small Materialized Aggregates. These SMAs are lightweight and easy to generate. They are similiar to views or data cubes but only compact information is generated for each data bucket.</td>
<td class="org-left">VLDB 1998</td>
<td class="org-left">SMA, Data Cube, Data Warehouse</td>
</tr>


<tr>
<td class="org-left">Different Cube Computation Approaches: Survey Paper</td>
<td class="org-left">Dhanshri S. Lad, Rasika P. Saste</td>
<td class="org-left">This paper survey the different algorithms to compute data cubes. The authors also propose use MR to speed up the data cube computation.</td>
<td class="org-left">IJCSIT 2014</td>
<td class="org-left">Data Cube, Mapreduce</td>
</tr>


<tr>
<td class="org-left">High-Diemnsional OLAP: A Minimal Cubing Approach</td>
<td class="org-left">Xiaolei Li, Jiawei Han, Hector Gonzalez</td>
<td class="org-left">This paper proposes to decompose the data cube computation by precomputing small sized groups called fragements and a value-list inverted index. All dimensions are divided into 3/4 dimension groups called fragments. For each fragments all data cubes are computed as lists of tuple ids using the inverted index. This paper also shows how to serve point queries and subcube queries with these fragments.</td>
<td class="org-left">VLDB 2004</td>
<td class="org-left">Data Cube, Shell Fragment, OLAP</td>
</tr>


<tr>
<td class="org-left">Data Cube: A Relational Aggregation Operator - Generalizing Group-By, Cross-Tab, and Sub-Totals</td>
<td class="org-left">Jim Gray, Adam Bosworth, Andrew Layman, Hamid Pirahesh</td>
<td class="org-left">This visionary paper explains to us what are data cubes and why we need them in analytics. It also shows us how to generate data cubes with the SQL group-by, proposes what enhancements we need make to group-bys. And it categorifies the aggregate functions. It shows how to compute data cubes with distributive and algebraic aggregates.</td>
<td class="org-left">Data Mining and Knowledge Discovery 19997</td>
<td class="org-left">Data Cube, Group-By, Cross-Tab, Roll-Up, Drill-Down</td>
</tr>
</tbody>
</table>

