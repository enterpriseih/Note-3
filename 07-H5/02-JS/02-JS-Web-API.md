# DOM

## DOM节点操作

### 获取DOM节点

<img src="img/image-20221121232859560.png" alt="image-20221121232859560" style="zoom:50%;" />

### DOM节点的property

- 修改的是DOM元素的js变量的属性，不会对节点标签产生影响，不会体现到html中，可能会引起DOM重新渲染

<img src="img/image-20221121233423206.png" alt="image-20221121233423206" style="zoom:50%;" />

### DOM节点的attribute

- 修改的标签的属性，会改变html属性，可能会引起DOM重新渲染

<img src="img/image-20221121234020838.png" alt="image-20221121234020838" style="zoom:50%;" />



## DOM结构操作

### 新增/插入节点

<img src="img/image-20221121230704852.png" alt="image-20221121230704852" style="zoom:50%;" />

### 获取子元素列表&获取父元素

<img src="img/image-20221121231028563.png" alt="image-20221121231028563" style="zoom:50%;" />

### 删除节点

<img src="img/image-20221121231630214.png" alt="image-20221121231630214" style="zoom:50%;" />





## DOM性能

- DOM操作非常‘昂贵’，避免频繁的DOM操作

- 对DOM查询做缓存

  <img src="img/image-20221121231907680.png" alt="image-20221121231907680" style="zoom:50%;" />

- 将频繁操作改为一次性操作

  <img src="img/image-20221121232127285.png" alt="image-20221121232127285" style="zoom:50%;" />

# BOM


