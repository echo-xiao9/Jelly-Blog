---
title: graph lab —— dijstra 算法及堆优化
date: 2021-08-20 15:28:49
tags: 
- algorithm 
- lab
photos: 
- https://tva1.sinaimg.cn/large/008i3skNly1gtnaps5bsaj60m80cijtb02.jpg
category: project
---


# 最短路径（Shortest Path）

最短路径是图论研究中的一个经典问题，旨在寻找图（由节点和路径组成的）中两个节点之间的最短路径。两种比较典型的方式是求单源最短路径（Single Source Shortest Path）和全源最短路径（All Pairs Shortest Path）。单源最短路径是给定一个点，求该点到剩余其他点的最短路径，可以用Dijkstra或者Bellman-ford算法解决；全源最短路径要求求出图中所有点之间的最短路径，可以将每个点看作做源点执行若干次计算单源最短路径，也可以使用Floyd-Warshall算法直接计算全源最短路径。 利用图的最短路径可以进一步分析出图的一些属性，从而了解图的一些特点。本次Lab中我们将利用最短路径来计算图中点的中心性（Centrality）。中心性是社交网络图分析中常用的属性，在现实生活中具有重要意义。

## 度中心性（Degree Centrality）

$𝐷𝐶(𝑣)= 𝑑𝑒𝑔𝑟𝑒𝑒(𝑣)$

度中心性是在网络分析中刻画节点中心性最简单直接的方法，一个节点的度越大就意味着这个节点的度中心性越高，该节点在网络中知名度越高，因为和他有直接关系的人最多。

`实现`：直接求邻接表的size就可以

## 亲近中心性（Closeness Centrality）

![image_1](https://tva1.sinaimg.cn/large/008i3skNly1gtnaf5afsoj60cg032t8l02.jpg)
图中一个节点的亲近中心度的计算方法如上式所示，其中N是该节点所在的连通分量中点的个数（Lab假设给定的社交网络图都是一个连通图，因此相当于是图中所有的节点）
$𝑑𝑖𝑠𝑡(𝑣,𝑣′)$是节点V和V’之间的最短距离。前文中的度中心性只利用了网络的局部性质，而亲近中心性则能表示一个点在整个网络中的位置。一个节点的亲近中心度越大，则其越接近整个图的几何中心。这一类人未必具有很高的知名度，但是同样在社交网络中扮演着很重要的角色，通过他们，信息可以更快地传递到网络中的每个人

`实现：` 写了一个函数  `double Graph::shortestPath(int src,vector<int> &dist)`，输入一个src,可以求出到每个点的最短路径长度，所以，对于每个点，求出这些长度之和，找到和最小的那个就可以

## 中介中心性（Betweenness Centrality）


<figure>
  <img src="https://tva1.sinaimg.cn/large/008i3skNly1gtnag8kjipj60ke030jre02.jpg" alt="ORB-SLAM流程图">
</figure>

如上式所示，一个点的中介中心度是它出现在其他点对之间最短路径的次数（两个点之间的最短路径可能有多条，只要点v在点vi和vj的多条最短路径中出现一次，即可计算一次，出现多次也只计算一次。因为是无向图，任意两个点之间的最短路径是对称的，所以在求和之后除以2）。与亲近中心性相同，中介中心性同样是对网络全局属性的一个刻画。一个点的中介中心度高则在网络中有很多人之间的间接关系会依赖于它，这意味着该点对其他成员有较强的控制和制约关系。

`实现` : 在`shortestPath`  函数里，其实是找到了一个点到其他点的最短路，这个图的每个线条的方向都定下来了，也就是说每个点的father都唯一确定了，所以针对这个新的生成的有向图，我们可以使用一个father数组去记录前继father(儿子可能有多个)。然后遍历每个点，都可以找回到src的一条路径，路径上的每个点的betweenness++

# 实验要求

输入 参数一是文本文件的路径，通过main函数的argv[1]接收。该文件包含多行，第一行为总节点个数。其后的d每一行由三个无符号32位整型v1，v2和weight组成，由逗号隔开，v1和v2代表两个节点之间的编号（节点编号是连续的，从0开始），weight是这两个节点之间边的权重。社交网络一般不会十分稠密，所以本次Lab的输入满足|E| << |V|²，请选择合适的算法。 输出 依次输出三个分别具有最高度、接近中心度和中介中心度的节点的编号及对应值，值相同选择编号小的节点。

如图1所示，则输入的文件内容如下： 

5 

0,3,2 

3,4,2 

1,4,1 

1,2,1 

2,3,5 

需要的输出如下，节点3的度最高为3，节点4的接近中心度最高为0.444444（任意两点之间的最短距离值小于无符号32位整型的上限，接近中心度计算和输出都使用double类型，保留6位小数），节点4的中介中心度最高为4（中介中心度的值不会超过无符号32位整型）。 

3 3 

4 0.444444 

4 4
<figure>
  <img src="https://tva1.sinaimg.cn/large/008i3skNly1gtnagqvvfxj60c40ckdg102.jpg" alt="ORB-SLAM流程图">
</figure>

<!-- ![Data%20Structure%20%E2%80%94%E2%80%94%20graph%20lab%2041f1903983554c0d84b16a7db54da67f/2021-05-21_8.03.02.png](Data%20Structure%20%E2%80%94%E2%80%94%20graph%20lab%2041f1903983554c0d84b16a7db54da67f/2021-05-21_8.03.02.png) -->

# 代码

```cpp
//
//  main.cpp
//  graphlab
//
//  Created by 康艺潇 on 2021/5/20.
//

#include <iostream>
#include <list>
#include <vector>
#include <queue>
#include <fstream>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Program to find Dijkstra's shortest path using
// priority_queue in STL
using namespace std;
# define INF 0x3f3f3f3f
//const int maxVertices=200000;
// iPair ==> Integer Pair
typedef pair<int, int> iPair;

// This class represents a directed graph using
// adjacency list representation
class Graph
{
    int V; // No. of vertices
    // In a weighted graph, we need to store vertex
    // and weight pair for every edge
    list< pair<int, int> > *adj;
    
    
    int* bewteenVec; // The number of the ith node in the shortest path of the other nodes
public:
    Graph(int V); // Constructor
    
    // function to add an edge to graph
    void addEdge(int u, int v, int w);
    
    // prints shortest path from s
    double shortestPath(int src,vector<int> &dist);
    void getMaxDegreeCentrality(int &maxDegreeCentrality, int &maxDegreeCentralityIndex);
    void BetweennessCentrality(
                               int &maxBetweennessCentrality,
                               int &maxBetweennessCentralityIndex);
    void getClosenessCentrality(double &maxClosenessCentrality,
                                int &maxClosenessCentralityIndex);
};

// Allocates memory for adjacency list
Graph::Graph(int V)
{
    this->V = V;
    adj = new list<iPair> [V];
    bewteenVec = new int[V+5];
    for(int i=0;i<V+5;i++)bewteenVec[i]=0;
    
}

void Graph::addEdge(int u, int v, int w)
{
    adj[u].push_back(make_pair(v, w));
    adj[v].push_back(make_pair(u, w));
}

// Prints shortest paths from src to all other vertices
double Graph::shortestPath(int src,vector<int> &dist)
{
    // Create a priority queue to store vertices that
    // are being preprocessed. This is weird syntax in C++.
    priority_queue< iPair, vector <iPair> , greater<iPair> > pq;
    int lastNode=0;
    int v=0;int tmp=0;
    
    int *father = new int[V+5];
    for(int i=0;i<V+5;i++)father[i]=-1;
    // Create a vector for distances and initialize all
    // distances as infinite (INF)
    //    vector<int> dist(V, INF);
//    cout<<"src:"<<src<<endl;
    // Insert source itself in priority queue and initialize
    // its distance as 0.
    pq.push(make_pair(0, src));
    dist[src] = 0;
    
    /* Looping till priority queue becomes empty (or all
     distances are not finalized) */
    while (!pq.empty())
    {
        // The first vertex in pair is the minimum distance
        // vertex, extract it from priority queue.
        // vertex label is stored in second of pair (it
        // has to be done this way to keep the vertices
        // sorted distance (distance must be first item
        // in pair)
        int u = pq.top().second;
        pq.pop();
        
        // 'i' is used to get all adjacent vertices of a vertex
        list< pair<int, int> >::iterator i;
        for (i = adj[u].begin(); i != adj[u].end(); ++i)
        {
            // Get vertex label and weight of current adjacent
            // of u.
            v = (*i).first;
            int weight = (*i).second;
            
            // If there is shorted path to v through u.
            if (dist[v] > dist[u] + weight)
            {
                // Updating distance of v
                dist[v] = dist[u] + weight;
                father[v]=u;
                pq.push(make_pair(dist[v], v));
                lastNode = u;
            }
        }
        
        }
    for(int i=0;i<V;i++){
        tmp=father[i];
        while (tmp!=-1 && father[tmp]!= -1) {
            bewteenVec[tmp]++;
            tmp=father[tmp];
        }
    }
    
    // Print shortest distances stored in dist[]
    int sumDist=0;
    for (int i = 0; i < V; ++i){
        sumDist+=dist[i];
    }
    //        printf("%d \t\t %d\n", i, dist[i]);
    return sumDist;
}

void Graph::getMaxDegreeCentrality(int &maxDegreeCentrality, int &maxDegreeCentralityIndex){
    int maxDegree=0;
    int maxDegreeIndex=0;
    for(int i=0;i<V;i++){
        int size = adj[i].size();
        if(size>maxDegree){
            maxDegree = size;
            maxDegreeIndex= i;
        }
    }
    maxDegreeCentrality = maxDegree;
    maxDegreeCentralityIndex = maxDegreeIndex;
}

void Graph::BetweennessCentrality(
                                  int &maxBetweennessCentrality,
                                  int &maxBetweennessCentralityIndex){
    
    for(int i=0;i<V;i++){
        if(bewteenVec[i]>maxBetweennessCentrality){
            maxBetweennessCentrality=bewteenVec[i];
            maxBetweennessCentralityIndex = i;
        }
    }
    maxBetweennessCentrality/=2;
    
}

void Graph::getClosenessCentrality(double &maxClosenessCentrality,
                                   int &maxClosenessCentralityIndex){
    vector<int>sumDist;
    int minDistIndex=0;
    int minDist=INF;
    for(int i=0;i<V;i++){
        vector<int> dist(V, INF);
        sumDist.push_back(shortestPath(i,dist));
    }
    for(int i=0;i<sumDist.size();i++){
        if(minDist> sumDist[i]){
            minDist=sumDist[i];
            minDistIndex = i;
        }
    }
    maxClosenessCentralityIndex = minDistIndex;
    maxClosenessCentrality = float(float(V-1)/float(minDist));
}

// Driver program to test methods of graph class
int main(int argc, char* argv[])
{
    ifstream infile;
    int n=0;
    infile.open(argv[1]);
    int maxDegreeCentrality=0, maxDegreeCentralityIndex=0;
    double maxClosenessCentrality=INF;
    int maxClosenessCentralityIndex=0;
    int maxBetweennessCentralityIndex=0,maxBetweennessCentrality=0;
    /* 节点总数 */
    infile >> n;

    Graph g(n);
    char* line = new char[n];
    int64_t v1, v2, weight;
    while (infile >> line) {
        
        char *p = strtok(line, ",");
        v1 = atoi(p);
        p = strtok(NULL, ",");
        v2 = atoi(p);
        p = strtok(NULL, ",");
        weight = atoi(p);
        g.addEdge(v1, v2, weight);
    }
    g.getMaxDegreeCentrality(maxDegreeCentrality, maxDegreeCentralityIndex);
    cout<<maxDegreeCentralityIndex<< ' '<<maxDegreeCentrality<<endl;
    g.getClosenessCentrality(maxClosenessCentrality, maxClosenessCentralityIndex);
    cout<<maxClosenessCentralityIndex<< ' '<<maxClosenessCentrality<<endl;
    g.BetweennessCentrality(maxBetweennessCentrality, maxBetweennessCentralityIndex);
    cout<<maxBetweennessCentralityIndex<<' '<<maxBetweennessCentrality<<endl;
    return 0;
}
```