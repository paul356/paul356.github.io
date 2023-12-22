---
layout: default
title: Paper Digest of November 2023
tags: [Database, AI4DB, Data Analytics]
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
<td class="org-left">A Reinforcement Learning Based R-Tree for Spatial Data Indexing in Dynamic Environments</td>
<td class="org-left">Tu Gu, Sheng Wang, etc</td>
<td class="org-left">This paper presents how to generate R-Tree cuts using reinforcement learning instead of arbitrary huristics. The value function is formulated against a base tree which is used as reference. The reinforcement method learns to choose actions maximizing the ratio against query costs on the base tree. The supported actions are chooseSubTree and cutSubTree. The R-Tree is constructed by inserting data points.</td>
<td class="org-left">SIGMOD 2023</td>
<td class="org-left">R-Tree, RL</td>
</tr>


<tr>
<td class="org-left">PAW: Data Partitioning Meets Workload Variance</td>
<td class="org-left">Zhe Li, Man Lung Yiu, Tsz Nam Chan</td>
<td class="org-left">This paper presents tries to optimize QD-Tree which performs good on seen workload but may perform badly on queries with variances. By introducing future queries during training PAW generates cuts which allows for some variances for future queries.</td>
<td class="org-left">ICDE 2022</td>
<td class="org-left">Partitioning, QD-Tree</td>
</tr>


<tr>
<td class="org-left">Efficient Online Reinforcement Learning with Offline Data</td>
<td class="org-left">Phillip J. Ball, Laura Smith, Ilya Kostrikov, Sergey Levine</td>
<td class="org-left">This paper presents a method to directly use offline data in online reinforcement learning, and show offline data can be used in reinforcement learning without offline training.</td>
<td class="org-left">arXiv 2023</td>
<td class="org-left">Reinforcement Learning, Offline Data</td>
</tr>


<tr>
<td class="org-left">Vertical Partitioning for Database Design: A Graphical Algorithm</td>
<td class="org-left">Shamkant B. Navathe, Minyoung Ra</td>
<td class="org-left">This paper presents a nonparametric method to cluster table attributes using a new undirected graph method. This method can group nodes with similar edge values between them.</td>
<td class="org-left">SIGMOD 1989</td>
<td class="org-left">Graph Algoritm, Vertical Partitioning</td>
</tr>
</tbody>
</table>

