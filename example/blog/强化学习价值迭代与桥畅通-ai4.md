---
title: 强化学习价值迭代与桥畅通
date: 2021-06-08 20:54:58.541
updated: 2022-01-16 18:35:23.271
url: https://leesdog.space/archives/ai4
summary: 第一题实际上就是要我们使用强化学习中的价值迭代法来更新完成整个矩阵的权值
categories: 
tags: 
- AI
- 算法
---

# 强化学习价值迭代与桥畅通
> 题目链接：https://inst.eecs.berkeley.edu//~cs188/sp20/project3/



## Q1:Value Iteration

第一题实际上就是要我们使用强化学习中的价值迭代法来更新完成整个矩阵的权值，由于我课上听的其实不是很懂，所以需要课下再学习一下价值迭代法：

首先我们已经在课上知道了马尔科夫决策的过程（MDP），总结起来就是三个部分：

- **状态空间State**（S）
- **决策空间Action**(A)
- **Action对State的影响和回报P**(State', Reward | State, Action)

其实比较关键的就是最后的P值如何计算，针对这一题每一轮的计算其实都是分成以下几个步骤

- 对于每个state，逐一尝试上、下、左、右四个Action

- - 记录Action带来的Reward、以及新状态 V(s')
  - 选择最优的Action，保存V(s) = Reward + V(s') （不是立刻更新）

这里有两个关键点容易搞错，也是价值迭代的关键所在

- 每一次的state迭代只在整个迭代完成后统一更新每一个state的value值，即做到当前state的计算只跟上一轮的状态有关，而不会因为本轮的计算而被影响
- state所拿到的Reward并不是走到以后拿到，而是在自己在当前位置时再走一步的时候拿到，这也是老师上课时讲的奖励滞后

最后套上价值迭代的公式即可
$$
V_{*}(s)=\max _{a} \sum_{s^{\prime}, r} p\left(s^{\prime}, r \mid s, a\right)\left[r+\gamma V_{*}\left(s^{\prime}\right)\right]
$$

```python
class ValueIterationAgent(ValueEstimationAgent):
    """
        * Please read learningAgents.py before reading this.*

        A ValueIterationAgent takes a Markov decision process
        (see mdp.py) on initialization and runs value iteration
        for a given number of iterations using the supplied
        discount factor.
    """
    def __init__(self, mdp, discount = 0.9, iterations = 100):
        """
          Your value iteration agent should take an mdp on
          construction, run the indicated number of iterations
          and then act according to the resulting policy.

          Some useful mdp methods you will use:
              mdp.getStates()
              mdp.getPossibleActions(state)
              mdp.getTransitionStatesAndProbs(state, action)
              mdp.getReward(state, action, nextState)
              mdp.isTerminal(state)
        """
        self.mdp = mdp
        self.discount = discount
        self.iterations = iterations
        self.values = util.Counter() # A Counter is a dict with default 0
        self.runValueIteration()

    def runValueIteration(self):
        # Write value iteration code here
        next_values = self.values.copy()
        for iter_nums in range(self.iterations):
            for state in self.mdp.getStates():
                if self.mdp.isTerminal(state):
                    continue
                #从上下左右的可能行动中找到Qvalue最大的一次
                next_values[state] = max([self.getQValue(state, action)
                                          for action in self.mdp.getPossibleActions(state)])
            self.values = next_values.copy()


    def getValue(self, state):
        """
          Return the value of the state (computed in __init__).
        """
        return self.values[state]


    def computeQValueFromValues(self, state, action):
        """
          Compute the Q-value of action in state from the
          value function stored in self.values.
        """
        successors = self.mdp.getTransitionStatesAndProbs(state, action)
        qval = 0
        for next_state, prob in successors:
            #根据公式计算q值
            qval += prob * (self.mdp.getReward(state, action, next_state) + self.discount * self.getValue(next_state))
        return qval
        util.raiseNotDefined()

    def computeActionFromValues(self, state):
        """
          The policy is the best action in the given state
          according to the values currently stored in self.values.

          You may break ties any way you see fit.  Note that if
          there are no legal actions, which is the case at the
          terminal state, you should return None.
        """
        policy = util.Counter()
        #从上下左右中可能的行动中获取在policy下的Q值，并使用封装好的argMax返回Q值最大的结果
        for action in self.mdp.getPossibleActions(state):
            policy[action] = self.getQValue(state,  action)

        return policy.argMax()
        util.raiseNotDefined()

    def getPolicy(self, state):
        return self.computeActionFromValues(state)

    def getAction(self, state):
        "Returns the policy at the state (no exploration)."
        return self.computeActionFromValues(state)

    def getQValue(self, state, action):
        return self.computeQValueFromValues(state, action)
```

执行

```shell
python autograder.py -q q1
```

![image-20210608202851742](https://image.leesdog.space/ai410.png)

执行

```shell
python gridworld.py -a value -i 100 -k 10
```

![image-20210608201112641](https://image.leesdog.space/ai402.png)

![image-20210608201245588](https://image.leesdog.space/ai403.png)

执行

```shell
python gridworld.py -a value -i 5
```

![image-20210608201344698](https://image.leesdog.space/ai407.png)

![image-20210608201327905](https://image.leesdog.space/ai406.png)



## Q2 Bridge Crossing Analysis

第二题其实就是让我们通过这样一个桥梁测试，同时只让我们改noise和discount中的一个参数；

**noise**就是其决策时的噪声，该值越大其决策的随机性越高；

**discount**其实就是γ，也就是每走一步所乘的权值

要通过该测试，其实分析一下题目的情况就可以了

![image-20210608202343748](https://image.leesdog.space/ai408.png)

可以看到我们希望他向右一直走，而不要向上下偏移，在这种情况下其实就是起点到终点**走路径最短的一条路**；要让程序如此行动，其实可以有两种方式，一种方式是将discount调整到使得正值偏大的一个地方，即使得下面式子整体偏大；还有一种方式是将noise调小，这样使得偏向走-100的概率变小，其实也是将整体Q值调大，这里将answerNoise调成0.001即可通过测试
$$
R_{t+1}+\gamma R_{t+2}+\gamma^{2} R_{t+3}+\gamma^{3} R_{t+4}+\ldots
$$
![image-20210608204123625](https://image.leesdog.space/ai412.png)



## Q3:Policies

这一题的情况如下

![image-20210608204416040](https://image.leesdog.space/AI415.png)

他需要我们调整参数以完成以下5个策略：

1. 更喜欢关闭退出（+1），冒着悬崖的风险（-10）
2. 更喜欢关闭退出（+1），但要避开悬崖（-10）
3. 偏爱远处出口（+10），冒着悬崖（-10）
4. 喜欢远处的出口（+10），避开悬崖（-10）
5. 避免出口和悬崖（因此一集永远不应该终止）

其实要完成这些结果，核心点在于去根据公式计算其最终的value是否是导致其朝着既定的方向走即可，只要首先把握一个大的方向即可：

当更喜欢冒着悬崖的风险的时候此时走的路径较短，所以我们应该加大每走一步的惩罚Reward；

当更喜欢远处出口的时候，可以加大每走一步的奖励Reward使其更愿意走向较远的出口；

同时noise其实都可以调到很小（甚至调成0也行）使其更愿意执行你所设定的方向

然后随便调一调就可以了

```python
def question3a():
    answerDiscount = 0.01
    answerNoise = 0
    answerLivingReward = -1
    return answerDiscount, answerNoise, answerLivingReward
    # If not possible, return 'NOT POSSIBLE'

def question3b():
    answerDiscount = 0.01
    answerNoise = 0.01
    answerLivingReward = 0.5
    return answerDiscount, answerNoise, answerLivingReward
    # If not possible, return 'NOT POSSIBLE'

def question3c():
    answerDiscount = 0.9
    answerNoise = 0
    answerLivingReward = -1
    return answerDiscount, answerNoise, answerLivingReward
    # If not possible, return 'NOT POSSIBLE'

def question3d():
    answerDiscount = 0.9
    answerNoise = 0.2
    answerLivingReward = 0
    return answerDiscount, answerNoise, answerLivingReward
    # If not possible, return 'NOT POSSIBLE'

def question3e():
    answerDiscount = 0
    answerNoise = 0.1
    answerLivingReward = 10
    return answerDiscount, answerNoise, answerLivingReward
```

最终结果：

![image-20210608204622649](https://image.leesdog.space/ai420.png)

