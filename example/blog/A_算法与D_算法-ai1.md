---
title: A*算法与D*算法
date: 2021-05-13 01:18:38.844
updated: 2022-01-16 18:25:33.599
url: https://leesdog.space/archives/ai1
summary: A*算法相较深度优先广度优先，最大的改进在于其有方向性的试探以及展望和回顾的结合
categories: 
tags: 
- AI
- 算法
---

# A*算法与D*算法


## A*算法

A*算法相较深度优先广度优先，最大的改进在于其有方向性的试探以及展望和回顾的结合

深度搜索（下图）：

![https://image.leesdog.space/100.gif](https://image.leesdog.space/100.gif)

```python
def depthFirstSearch(problem):

    "*** YOUR CODE HERE ***"
    #可以看出这个函数是用来得到当前位置的
    print("Start:", problem.getStartState())
    #这个函数是用来判断当前位置是否是终点
    print("Is the start a goal?", problem.isGoalState(problem.getStartState()))
    #这个函数是用来得到后继节点的 例如((5, 4), 'South', 1)
    print("Start's successors:", problem.getSuccessors(problem.getStartState()))
    start = [problem.getStartState(), 0, []]
    # 创建一个栈用于存放深度序列，同时创建一个访问过的节点序列
    result = util.Stack()
    visited = []
    # 将开始的节点状态放入结果栈内
    result.push(start)
    while result.isEmpty()!=True:
        [state, cost, path] = result.pop()
        #如果到达终点，则返回path
        if problem.isGoalState(state):
            return path
        #如果未被访问，则访问并计算路径并压入所有后继节点进栈
        if not state in visited:
            visited.append(state)
            for child_state, child_action, child_cost in problem.getSuccessors(state):
                new_cost = cost + child_cost
                new_path = path + [child_action]
                result.push([child_state, new_cost, new_path])
```





宽度搜索（下图）：

![](https://image.leesdog.space/2%2000_00_00-00_00_30.gif)



```python
def breadthFirstSearch(problem):
    """Search the shallowest nodes in the search tree first."""
    "*** YOUR CODE HERE ***"
    #在之前的数据结构中我们其实就学了，深度和宽度代码上区别就是使用队列和栈的区别巧妙实现
    start = [problem.getStartState(), 0, []]
    result = util.Queue()
    result.push(start)  
    visited = []
    while not result.isEmpty():
        [state, cost, path] = result.pop()
        if problem.isGoalState(state):
            return path
        if state not in visited:
            visited.append(state)
            for child_state, child_action, child_cost in problem.getSuccessors(state):
                new_cost = cost + child_cost
                new_path = path + [child_action]
                result.push([child_state, new_cost, new_path])
```



A*搜索（下图）：

![](https://image.leesdog.space/3%2000_00_00-00_00_30.gif)



```python
def aStarSearch(problem, heuristic=nullHeuristic):
    """Search the node that has the lowest combined cost and heuristic first."""
    "*** YOUR CODE HERE ***"
    #A*算法的核心在于展望+回顾，我个人感觉事实上A*就是典型的类动态规划问题，有一点贪心的感觉
    #其核心在于f(n)=g(n)+h(n)
    #g(n)代表距离起点的代价，这个代价是局部确认的
    #h(n)代表距离终点的代价，这个代价是预估的
    #如果要对A*算法本身做优化，可能需要在这个预估上做优化
    #核心思想在于找到f(n)最优的路径
    result = util.PriorityQueue()
    start = [problem.getStartState(), 0, []]
    p = 0
    result.push(start, p)  
    visited = []
    while not result.isEmpty():
        [state, cost, path] = result.pop()
        #print(state)
        if problem.isGoalState(state):
            #print(path)
            return path 
        if state not in visited:
            visited.append(state)
            for child_state, child_action, child_cost in problem.getSuccessors(state):
                new_cost = cost + child_cost
                new_path = path + [child_action, ]
                result.push([child_state, new_cost, new_path], new_cost + heuristic(child_state, problem))
    util.raiseNotDefined()
```

可以看到基本的A*算法相较深度和广度优先来说能**找到路径的最优解**去做行动，而深度优先和广度优先算法虽然能走到最终的位置，但是**并没有找到最优解**

主要的一个因素在于A*算法在运算过程中，每次从优先队列中选取f(n)值最小（优先级最高）的节点作为下一个待遍历的节点。$ f(n)=g(n)+h(n)$，

- f(n)是节点n的综合优先级。当我们选择下一个要遍历的节点时，我们总会选取综合优先级最高（值最小）的节点。
- g(n) 是节点n距离起点的代价。
- h(n)是节点n距离终点的预计代价，这也就是A*算法的**启发函数**。

其实我们需要注意的是启发函数：

- 在极端情况下，当启发函数**h(n)始终为0**，则将由g(n)决定节点的优先级，此时算法就退化成了**Dijkstra算法**。
- 如果h(n)始终**小于等于节点n到终点的代价**，则A*算法**保证一定能够找到最短路径**。但是当h(n)的值越小，算法将遍历越多的节点，也就导致算法越慢。
- 如果h(n)完全等于节点n到终点的代价，则A*算法将找到最佳路径，并且速度很快。但并非所有场景下都能做到这一点。因为在没有达到终点之前，我们很难确切算出距离终点还有多远，多数情况下只是进行估计。
- 如果h(n)的值比节点n到终点的代价要大，则A*算法不能保证找到最短路径，不过此时会很快。
- 在另外一个极端情况下，如果h(n)相较于g(n)大很多，则此时只有h(n)产生效果，这也就变成了**最佳优先搜索**。

一般来说，如果只允许图形向上下左右四个方向运动，这种情况用**曼哈顿距离**作为启发函数居多

其实自己看一下上面图的效果，里面的背景颜色代表其试探的路径次数，颜色越深代表试探次数越多，在A*算法中虽然我们通过默认的曼哈顿距离作为启发函数能做到最短距离的获取，但其试探次数相较深度优先其实是多了不少的，这里可能是A\*算法可能的一种优化



仔细观察图可以发现，这几张地图多在一些死角墙多的地方会重复的进行试探遍历，在这张地图以及某些地图上这个问题就比较突出，我和其他几个朋友讨论过，他们选择了在计算cost时在最开始计算初始位置到目标位置直线墙的数量，然后采取一种增加权值的策略达到趋向走墙少的这样一种试探方式，我最开始觉得比较有道理，但是后面仔细一想并非如此，因为这种策略通过直接的控制权值影响了f（n）中g和h所占比重，如果地图的结果并不是墙多的时候距离长，则会导致结果出现问题或冗余的试探计算，也就是说这种策略和地图长什么样有很大关系，并不是一种广义上的改进策略



## D*算法

在我搜索是不是有方法基于A*同时更优秀的时候，我发现了D\*算法，最初的D\*算法其实只是一种知情的增量搜索算法，后面基于A\*，基于动态SWSF-FP又进化成一种增量启发式搜索算法D\* Lite，貌似效果很好，但由于我有些地方看的还是不太懂所以没有成功把他复现出来，之后可以抽时间继续研究一下

我在youtube上搜索到了关于D*和A\*对比的演示视频

#### A*:

![](https://image.leesdog.space/A%2000_00_00-00_00_30.gif)



#### D*

![](https://image.leesdog.space/D%2000_00_00-00_00_30.gif)





D*的主要几个思想一是在于其是**从终点开始搜索**，而不是起点；二是在于维护了一个新的openlist，其中的节点有如下几个状态

- NEW：意味着它从未被列入OPEN list
- OPEN：意味着它当前在OPEN list中
- CLOSED：意味着它不在OPEN list中
- RAISE：意味着它的成本比上次OPEN list时要高
- LOWER：意味着它的成本比上次OPEN list时要低

该算法通过迭代地从OPEN list中选择一个节点并对其求值来工作。然后，它将节点的变化传播到所有相邻节点，并将它们放到OPEN list中。这种传播过程称为“扩张”。与从始至终遵循路径的A\*不同，D*从目标节点开始向后搜索。每个扩张节点都有一个反向指针，它指向指向目标的下一个节点，每个节点都知道目标的确切成本。当开始节点是下一个要展开的节点时，算法就完成了，只需遵循反向指针就可以找到目标的路径。当在指定的路径上检测到障碍物时，**所有受影响的点将再次被放到OPEN列表中**，这次标记为RAISE。然而，在一个RAISED的节点增加成本之前，算法会**检查它的邻居，并检查它是否可以降低节点的成本**。如果没有，则提升状态传播到所有节点的后代，即具有反向指针的节点。然后评估这些节点，并且传递RAISE状态，形成波。 当RAISED节点可以减少时，它的反向指针会更新，并将LOWER状态传递给它的邻居。这些RAISE和LOWER的状态波是D\*的核心。到这个时候，一系列其他的点就不会被波浪“碰触”了。



