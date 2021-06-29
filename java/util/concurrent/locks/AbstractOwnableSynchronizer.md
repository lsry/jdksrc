## AbstractOwnableSynchronizer   

### O、概述
该类是一个线程独占的抽象同步器，描述了资源所有权的概念及相关行为，由子类去实现相关行为。

### 一、行为   
+ exclusiveOwnerThread：描述当前资源所属线程；
+ setExclusiveOwnerThread
+ getExclusiveOwnerThread  