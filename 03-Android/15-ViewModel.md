

# ViewModel

 ## 什么是ViewModel

- 具备宿主生命周期感知能力的数据存储组件

- ViewModel保存的数据，在页面因**配置变更导致页面销毁重建**之后依然也是存在的

  > 配置变更：横竖屏切换、分辨率调整、权限变更、系统字体样式变更

## ViewModel的用法

![image-20210524231418439](../img/image-20210524231418439.png)

![image-20210524231530397](../img/image-20210524231530397.png)

![image-20210524231637652](../img/image-20210524231637652.png)

![image-20210524231741207](../img/image-20210524231741207.png)

![image-20210524231856172](../img/image-20210524231856172.png)

## 配置变更ViewModel复用实现原理

![image-20210524232021885](../img/image-20210524232021885.png)

## SaveState
- SavedStatedHandle的数据存储与恢复。即便ViewModel不是同一个实例，它存储的数据也能做到复用
- SavedStateRegistryController：用于创建SaveStatedRegistry
- SavedStatedRegistry数据存储、恢复中心。
- SavedStateHandle:单个ViewModel数据存储、恢复

![image-20210525212017108](../img/image-20210525212017108.png)

![image-20210525212110826](../img/image-20210525212110826.png)

![image-20210525211935163](../img/image-20210525211935163.png)

![image-20210525212245860](../img/image-20210525212245860.png)

![image-20210525212945587](../img/image-20210525212945587.png)

![image-20210525213013630](../img/image-20210525213013630.png)



![image-20210525215618579](../img/image-20210525215618579.png)

用法一

![image-20210525215643726](../img/image-20210525215643726.png)

用法二

![image-20210525215738414](../img/image-20210525215738414.png)

## 总结

![image-20210525220253020](../img/image-20210525220253020.png)

