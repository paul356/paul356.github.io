---
layout: default
title: Paper Digest of June 2023
tags: [UB-Tree, B+-Tree, Z-Curve]
nav_order: {{ page.date }}
---


# Paper Digest of June 2023


## Generalized Multi-dimensional Data Mapping and Query Processing

Author: Rui Zhang, et al.  
Publisher: TODS 2005  
This paper propose a framework GiMP can be customized to different data mapping schemas, for example, the UB-Tree, the Pyramid technique, the iMinMax and the iDistance.


## Integrating the UB-Tree into a Database System Kernel

Author: Frank Ramsak, et al.   
Publisher: VLDB 2000  
The paper presents the experience of intergrating the UB-Tree into a database system.

1.  The UB-Tree is just like the B-Tree except the key is calculated from multidimentional attributes using methods like the Z-Order.
2.  Page splitting is also a little different. It is better to split the page near the middle point. So that the performance is better on average.

