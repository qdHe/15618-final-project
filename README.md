# CUDA forced-directed graph drawing
# Project Proposal
Parallel network data visualization using GPU.

## URL
https://qdhe.github.io/15618-final-project/

## Summary

We are going to parallelize a forced-directed graph algorithm that considers force between two nodes to draw a aesthetically-pleasing graph on NVIDIA GPUs. We will compare the speedup and quality of our parallel algorithm with the sequential version.

## Background

_Graph drawing_ shows a graph based on the topological relationship between vertices and edges. One category of typical algorithms to draw graphs in an aesthetically-pleasing way is forced-directed method. The idea of a force-directed layout algorithm is to consider a force between any two nodes. 
In this project, we want to implement and optimize a specific version called Fruchterman-Reingold. The nodes are represented by steel rings and the edges are springs between them. The attractive force is analogous to the spring force and the repulsive force is analogous to the electrical force. The basic goal is to minimize the energy of the system by moving the nodes and changing the forces between them. The following image is an example - Social network visualization.

<img src="https://github.com/qdHe/15618-final-project/blob/master/images/SocialNetworkAnalysis.png" alt="img0" width="800" align="middle" />

Suppose _k_ is the constant describing the optimal length of edge, and _d_ is the distance between two nodes. 

Attractive force is _fa(x) := return x^2/k_

Repulsive force is _fr(x) := return k^2/x_

The algorithm initially assigns each vertex a random position. It iteratively calculates the repulsive force between all pairs of vertices and the attractive force for each edge. Position of vertex is changed based on the resultant force vector it receives. The algorithm ends when all vertex positions do not change or the number of iterations exceeds a certain threshold.

Pseudocode:
~~~ java
area := W * L; //W and L are the width and length of the frame
G := (V, E); //the vertices are assigned random initial positions     
for i := 1 -> iterations
    //calculate repulsive forces
    for v in V //each vertex has two vectors: .pos and .disp
        v.disp := 0; 
        for u in V
            d := v.pos - u.pos
            v.disp += (d/|d|) * fr(|d|)
    //calculate attractive forces
    for e in E //each edge is an ordered pair of vertices .v and .u 
        d := e.v.pos – e.u.pos 
        e.v.disp –= (d/|d|) * fa(|d|) 
        e.u.disp += (d/|d|) * fa(|d|)
    //limit the maximum displacement to the temperature t
    //prevent from being displaced outside frame
    for v in V
        v.pos += (v.disp/|v.disp|) * min(v.disp, t)
        v.pos.x = min(W/2, max(-W/2, v.pos.x))
        v.pos.y = min(L/2, max(–L/2, v.pos.y)) 
    //reduce the temperature as the layout approaches a better configuration
    t := cool(t)
~~~




## Challenges
The first challenge is how to handle the massive dataset. Assume we have n nodes, each iteration we need to calculate forces between n^2 pairs of nodes and the number of iterations it takes to converge is approximately n. In other words, the time complexity of computation is O(ln^2) for _l_ iterations, which is unacceptable for large datasets. To achive nearly realtime computation for those dataset, we need to trade off between performance and quality, by applying approximation methods like Barnes–Hut.

The second challenge is how to assign jobs evenly. Since the connectivity of nodes varies from each other. The amount of work for a node with many edges is different from a node with few edges when computing attractive power. And the position of each node affects the caculation it needs, too.  Since nodes will move during two iterations, the cost and communication patterns will also change over time. 

### Dependency
The next position of one node depends on every other node, as the accumulated force calculation depends on distances.

### Memory Access
To update one node, we need to calculate all nodes within a certain distance. However, the positions of each node change frequently. As a result, time locality and space locality is poor. To reduce cache misses, we plan to order the node array or construct a Barnes–Hut tree before each iteration.

### Communication
Sychronization needs to be done after each iteration. When computing movement of a node, the previous positions of other nodes are needed, which will be communicated through shared memory. When dataset is large, the edges between nodes might be too large to fit in shared memory. So there will also be communication between global memory and shard memory inside each CUDA blocks. 



## Resources
We will build the program from scratch in C++. We follow the guidance from the following references:

[1] FRUCHTERMAN T M J, REINGOLD E M. Graph drawing by force-directed placement[J]. Software, practice & experience, 1991, 21(11): 1129-1164.

[2] Jacomy M, Venturini T, Heymann S, et al. ForceAtlas2, a continuous graph layout algorithm for handy network visualization designed for the Gephi software[J]. PloS one, 2014, 9(6): e98679.

[3] https://www.boost.org/doc/libs/1_55_0/boost/graph/distributed/fruchterman_reingold.hpp

We also need NVIDIA GPU resourse as we want to parallel the algorithm through CUDA.


## Goals and Deiverables
### Plan to achieve
- 1: Write the CUDA parallelized version of force-directed algorithm and run on GPU.
- 2: The complexity of the original algorithm is O(ln^2) for l iterations, we will reduce the total complexity and do some speed-quality tradeoff.

### Hope to achieve
- 1: Process massive data in real time.
- 2: Write the parallelized CPU version of force-directed algorithm with OpenMP and run on Xeon Phi. Compare the performance of parallel GPU version and CPU version.

### Demo
- 1: We will show our speedup graphs which compare the performance of different versions of algorithms/different size of input data.
- 2: We will show the output visualization images of our program and guarantee the quality is similar to sequential version.
- 3: We may also show how the visualization images evolve with iterations.

## Platform

We plan to use the Latedays cluster to run our code, which will use Tesla K40 GPU. One of the main disadvantage of force-directed algorithms is high running time with large amounts of data. Therefore, we want to parallelize the algorithm with GPU to imrpove the performance as GPU has good ability of computing and parallel. Also, Latedays has Xeon Phi which enables parallel through OpenMP as well. For consistency, we will also run the sequential version on Latedays. 


## Schedule

* **Week 1 11.5--11.11**

> Understand the algorithm and the code in C++ boost library. Start the sequential implementation in C++.

* **Week 2 11.12--11.18**

> Finish the sequential version of the program and analyze the performance by fine-grained timing. Identify the bottleneck and come up with an approach to parallelize the program. Start programming the parallel version of the program.

* **Week 3 11.19--11.25**
> Iterate on the parallel version. Try to optimize the complexity through algorithms such as Barnes-Hut, k-d tree or quad-tree.

* **Week 4 11.26--12.2**

> Iterate on the parallel version. Try some other methods shuch as spatial hashing.

* **Week 5 12.3--12.9**

> Finish parallelizing the program and generate results. If there is time left, write the parallel CPU version using OpenMP and compare performance.

* **Week 6 12.10--12.16**

> Wrap up the project. Write final report. Prepare video and poster for demo.



## Authors

* **Qidu He** -  [qiduh]
* **Di Jin** -  [djin2]
