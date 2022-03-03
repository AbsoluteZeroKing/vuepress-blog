---
title: ReflexAgent与Minimax算法
date: 2021-05-13 01:13:35.825
updated: 2022-01-16 18:35:41.843
url: https://leesdog.space/archives/ai2
summary: 我们需要做的就是设计一个更好的评估函数，主要的计算思想是计算自己当前位置与食物以及ghost的曼哈顿距离
categories: 
tags: 
- AI
- 算法
---

# ReflexAgent与Minimax算法



## Q1:Reflex Agent

​	第一题事实上读了文档以后可以知道我们需要做的就是设计一个更好的评估函数，主要的计算思想是计算自己当前位置与食物以及ghost的**曼哈顿距离**，距离食物越近最终决定决策的score越大；距离ghost越近score越小，现在的关键在于这里**越小越大的权重标准**是多少，也就是说两者的影响因子各是多少的问题

其实如果经过不断的尝试，可以发现最终的score有很多种取值方式，这里我就以能通过q1为标准写一个，具体代码如下：

```python
    def evaluationFunction(self, currentGameState, action):
        """
        这里提示我们要设计一个更好的评估函数
        """
        # Useful information you can extract from a GameState (pacman.py)
        successorGameState = currentGameState.generatePacmanSuccessor(action)
        newPos = successorGameState.getPacmanPosition()
        newFood = successorGameState.getFood()
        newGhostStates = successorGameState.getGhostStates()
        newScaredTimes = [ghostState.scaredTimer for ghostState in newGhostStates]

        #计算每个剩余食物与自己的曼哈顿距离，距离越小score当然应该越大
        foodPosition = newFood.asList()
        distance1 = []
        for food in foodPosition:
            distance1.append(util.manhattanDistance(food, newPos))

        #计算每个ghost与自己的曼哈顿距离，距离越小score当然越小，如果距离为0则不可这么走
        distance2 = []
        for ghost in newGhostStates:
            distance = util.manhattanDistance(ghost.getPosition(), newPos)
            if distance == 0:
                distance2.append(-float('inf'))
            else:
                distance2.append(distance)
        #只剩最后一个食物时如果走这一步能吃掉食物且没有ghost，则走这一步
        if len(distance1) == 0 and min(distance2)!=0:
            return float('inf')
        else:
        #只剩最后一个食物时如果走这一步能吃掉食物且有ghost，则不走这一步
            if len(distance1) == 0:
                return -float('inf')
        #score取的是MD2/MD1^0.95+3score
        score =min(distance2)/min(distance1)**0.95+3*successorGameState.getScore()
        return score
```

最终结果：

![](https://image.leesdog.space/ait1.png)





## Q2:Minimax

这一题其实就是实现minimax算法，minimax算法老师上课讲的其实比较清楚了

Minimax算法又名极小化极大算法，是一种找出失败的最大可能性中的最小值的算法。Minimax算法常用于棋类等由两方较量的游戏和程序，这类程序由两个游戏者轮流，每次执行一个步骤。我们众所周知的五子棋、象棋等都属于这类程序，所以说Minimax算法是基于搜索的博弈算法的基础。该算法是一种零总和算法，即一方要在可选的选项中选择将其优势最大化的选择，而另一方则选择令对手优势最小化的方法。


![img](https://image.leesdog.space/ai22.png)



具体到这一题的Q2，其实题目中也已经写了，即使我们使用minimax算法，但是输也是正常现象，我们需要做的是实现一个不同深度的minimax算法：

```python
#注意pacman不应采取stop操作，因为autograder认为有比stop更优的解
class MinimaxAgent(MultiAgentSearchAgent):
    def getAction(self, gameState):

        GhostIndex = [i for i in range(1, gameState.getNumAgents())]
   	#Minimax算法根据伪代码有两部门组成，一个是min，即拿到下属节点的最小值并赋值
        def min_value(state, d, ghost):

            if state.isWin() or state.isLose() or d == self.depth:
                return self.evaluationFunction(state)
            value = float('inf')
            #这里相当于将每一个ghost走过一步后的state遍历一遍
            for action in state.getLegalActions(ghost):
                #注意pacman不应采取stop操作，因为autograder认为有比stop更优的解
                if ghost == GhostIndex[-1]:
                    value = min(value, max_value(state.generateSuccessor(ghost, action), d + 1))
                else:
                    value = min(value, min_value(state.generateSuccessor(ghost, action), d, ghost + 1))
            return value
	#一个是max，即拿到下属节点的最大值并赋值
        def max_value(state, d):  # maximizer

            if state.isWin() or state.isLose() or d == self.depth:
                return self.evaluationFunction(state)

            value = -float('inf')
            for action in state.getLegalActions(0):
                value = max(value, min_value(state.generateSuccessor(0, action), d, 1))
            # print(v)
            return value
        
        #pacman作为max玩家需要在ghost做出对于ghost最优的action下找到最优应对
        result = [(action, min_value(gameState.generateSuccessor(0, action), 0, 1)) for action in
               gameState.getLegalActions(0)]
        result.sort(key=lambda k: k[1])

        return result[-1][0]

```

可以看到结果上有时赢有时输

![image-20210513010828035](https://image.leesdog.space/ai28.png)

![image-20210513005124250](https://image.leesdog.space/ai24.png)



## Q3:Alpha-Beta Pruning

其实alpha-beta剪枝算法就是将上述minimax算法中的不必要判断的节点提早进行剪枝处理，这样可以大大提高算法效率减小计算，整个剪枝算法主要就是在上述的max和min算法上引入α和β两个参数用于判断是否可以剪枝

![Alpha-Beta实施](https://inst.eecs.berkeley.edu/~cs188/sp20/assets/images/alpha_beta_impl.png)

具体代码如下：

```python
class AlphaBetaAgent(MultiAgentSearchAgent):

    def getAction(self, gameState):
        now_value = -float('inf')
        alpha = -float('inf')
        beta = float('inf')
        nextAction = Directions.STOP

        legalActions = gameState.getLegalActions(0).copy()

        for next_action in legalActions:
            nextState = gameState.generateSuccessor(0, next_action)

            next_value = self.get_node_value(nextState, 0, 1, alpha, beta)
            if next_value > now_value:
                now_value, nextAction = next_value, next_action
            alpha = max(alpha, now_value)
        return nextAction
        util.raiseNotDefined()
#这个函数用于对节点做出是否要进行α，β剪枝操作的判断
    def get_node_value(self, gameState, cur_depth=0, agent_index=0, alpha=-float('inf'), beta=float('inf')):
        """
        使用alpha beta剪枝来确定合适的操作
        """
        max_party = [0, ]
        min_party = list(range(1, gameState.getNumAgents()))

        if cur_depth == self.depth or gameState.isLose() or gameState.isWin():
            return self.evaluationFunction(gameState)
        elif agent_index in max_party:
            return self.alpha_value(gameState, cur_depth, agent_index, alpha, beta)
        elif agent_index in min_party:
            return self.beta_value(gameState, cur_depth, agent_index, alpha, beta)
        else:
            print('发生错误')

    def alpha_value(self, gameState, cur_depth, agent_index, alpha=-float('inf'), beta=float('inf')):
        value = -float('inf')
        legalActions = gameState.getLegalActions(agent_index)
        for index, action in enumerate(legalActions):
            next_v = self.get_node_value(gameState.generateSuccessor(agent_index, action),
                                         cur_depth, agent_index + 1, alpha, beta)
            value = max(value, next_v)
            # 直接剪枝
            if value > beta:
                return value
            alpha = max(alpha, value)
        return value

    def beta_value(self, gameState, cur_depth, agent_index, alpha=-float('inf'), beta=float('inf')):
        """
        min_party, search for minimums
        """
        value = float('inf')
        legalActions = gameState.getLegalActions(agent_index)
        for index, action in enumerate(legalActions):
            if agent_index == gameState.getNumAgents() - 1:
                next_v = self.get_node_value(gameState.generateSuccessor(agent_index, action),
                                             cur_depth + 1, 0, alpha, beta)
                # 开始下一个深度
                value = min(value, next_v)
                # 直接剪枝
                if value < alpha:
                    return value
            else:
                next_v = self.get_node_value(gameState.generateSuccessor(agent_index, action),
                                             cur_depth, agent_index + 1, alpha, beta)
                # 开始下一个深度
                value = min(value, next_v)
                # 在当前深度判断下一步是否需要被剪枝
                if value < alpha:
                    return value
            beta = min(beta, value)
        return value
```

结果：

![ai27](https://image.leesdog.space/ai27.gif)

其实从结果上看我们可以很明显的看出经过剪枝的算法在得到同样结果的情况下大大减少了自己需要展开的节点数，但是由于正反方两者都是假设对方总是做出最优解，所以直接导致限制了自己所能采取的步骤，如上图中很多时候会停止；我个人认为此类算法的最重要因素在于其评价函数的设定 ，也就是score采用一种什么样的形式去进行计算对最终的行动与判定有着至关重要的作用，这里其实我只是设计了一种满足当前问题求解的一种函数，其实并不具有广泛性和可解释性，纯粹的是经过尝试后刚好满足罢了，如果想要更好的效果可能需要深层次研究一下各个参数取值与结果的整体图像然后取得最优点才可以
