---
title: Q-learning与ε-greedy
date: 2021-06-14 22:50:19.709
updated: 2022-01-16 18:34:59.088
url: https://leesdog.space/archives/ai5
summary: 这一题在经过之前课上的讲解以后比较清楚了,关键公式就是下面的式子：
categories: 
tags: 
- AI
- 算法
---

# Q-learning与ε-greedy
> 题目链接：https://inst.eecs.berkeley.edu//~cs188/sp20/project3/



## Q6:Q-Learning

这一题在经过之前课上的讲解以后比较清楚了,关键公式就是下面的式子：

![](https://cdn.mathpix.com/snip/images/GsNSYWyu5Uq_vivGWWbCMRHRUVWzNtf_YKUroSoElHc.original.fullsize.png)

这个式子老师课上也讲过了，其实前半部分就是Q值，后半部分是带入V值进行更新，后半部分的V值中的R（S,a）就是眼前利益，$\max _{a} Q\left(S^{\prime}, a\right)$ 其实就是记忆中的利益，从这个公式我们也可以分析出γ越小选择越重视眼前利益，反之越重视以往经验

```python
	def update(self, state, action, nextState, reward):
		# Q(S,A)
        QValue = self.Q_value[(state, action)] #Q值
        # (R(S,a)+γmaxQ(S',a)-Q(s,a))
        VValue = reward + (self.discount * self.computeValueFromQValues(nextState)-QValue)  # V值
        #Q(S,A)=Q(S,A)+α{R(S,a)+γmaxQ(S',a)-Q(s,a)}
        self.Q_value[(state, action)] =  QValue + self.alpha * VValue  #更新Q值
```



```python
    def update(self, state, action, nextState, reward):
        #Q(S,A)
        QValue = self.Q_value[(state, action)] #Q值
        #(R(S,a)+γmaxQ(S',a))
        VValue = reward + (self.discount * self.computeValueFromQValues(nextState))  # V值
        #Q(S,A)=（1-α）Q(S,A)+α{R(S,a)+γmaxQ(S',a)}
        self.Q_value[(state, action)] = (1 - self.alpha) * QValue + self.alpha * VValue  #更新Q值
```

其他部分的内容和之前类似，最终测试结果：

![image-20210614222152551](https://image.leesdog.space/ai501.png)





## Q7 Epsilon Greedy

这里使用的是ε-greedy方法，即每个状态以**ε的概率进行探索**，此时将随机选取action，而**剩下的1-ε的概率则进行正常选择**，即按上述方法，选取当前状态下效用值较大的动作，事实上就是在getaction这里加入几率搜索：

```python
        # Pick Action
        legalActions = self.getLegalActions(state)
        action = None
        # 这一步是在得到动作，但因为加入的探索值所以有几率进行探索
        if util.flipCoin(self.epsilon):
            action = random.choice(legalActions)
        else:
            action = self.getPolicy(state)
        return action
        util.raiseNotDefined()
```

![image-20210614224633392](https://image.leesdog.space/ai502.png)
