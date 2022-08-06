# Launcher启动流程与ActivityThread分析

![image-20210809222510881](../img/image-20210809222510881.png)

## 从开机到SystemServer系统服务启动

![image-20210809222757936](../img/image-20210809222757936.png)

![image-20210809223105493](../img/image-20210809223105493.png)

 ## SystemServer系统服务启动过程

![image-20210809223448186](../img/image-20210809223448186.png)

![image-20210809223514950](../img/image-20210809223514950.png)

- SystemServer的main方法主要做了以下三件事情，即启动不同类型的系统服务

![image-20210809223643827](../img/image-20210809223643827.png)

- 下图是SystemServer进程中启动的核心系统服务示意图

![image-20210809223811894](../img/image-20210809223811894.png)

## Launcher应用的进程启动流程

![image-20210809224258325](../img/image-20210809224258325.png)

### Activity的任务栈模型

![image-20210809224633987](../img/image-20210809224633987.png)

### Launcher进程启动流程图

![image-20210809224806291](../img/image-20210809224806291.png)

![image-20210809225356923](../img/image-20210809225356923.png)

### Framework源码分析技巧

![image-20210809225725414](../img/image-20210809225725414.png)

### 系统如何识别已安装的应用哪个是Launcher应用？

![image-20210809230143174](../img/image-20210809230143174.png)

![image-20210809234032629](../img/image-20210809234032629.png)

## Launcher应用的桌面启动流程

### ActivityThread结构图

![image-20210809234257451](../img/image-20210809234257451.png)

### 状态机设计模式

![image-20210809235944084](../img/image-20210809235944084.png)

### HomeActivity创建流程

![image-20210810000341114](../img/image-20210810000341114.png)



## Activity与四大组件那些事



## 如何在源码中取其精华

![image-20210810000615597](../img/image-20210810000615597.png)

## 来自ActivityThread的启发与拓展

