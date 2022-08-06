# 领京豆备忘录

## JDDevice.height != window.innerHeight

- 在web中JDDevice.height仅表示不包括状态栏和导航栏的高度

## H5 color接口传参

- body无参时可不传或者传{}
- 

## H5挽留弹窗

站内挽留弹窗需要在switchQuery接口中mock以下字段：

```json
{
          "appVersionDown": "",
          "appVersionUp": "",
          "name": "wbWebControlBack",
          "status": 1,
          "value": "h5.m.jd.com,jm-webview;plus.m.jd.com,1398XHSIWsjshwe12;carry.m.jd.com,3KSjXqQabiTuD1cJ28QskrpWoBKT;bunearth.m.jd.com;jdsupermarket.jd.com;h5.m.jd.com,2B8kKnT21JEjo9JMcuHsdwUepJJ;h5.m.jd.com,2H5Ng86mUJLXToEo57qWkJkjFPxw;pro.m.jd.com;prodev.m.jd.com;h5.m.jd.com,jx5KuRH4XVKHqDWpt7sBcMGxp3i;sfgala.jd.com;plus.m.jd.com,renewalGift;prodev.m.jd.com,8tHNdJLcqwqhkLNA8hqwNRaNu5f;wbbny.m.jd.com;h5.m.jd.com,fegzUHaydYypy;h5.m.jd.com,2DC7QCKjLiTq;h5.m.jd.com,2umFErnCStpf9;u.jr.jd.com,order-cash-back;utest.jr.jd.com,order-cash-back;h5.m.jd.com,pFAZGLdzVhpi;h5.m.jd.com,3LYvkC7wvijnhQXscB;smc.m.jd.com,monthIndex;xspace.m.jd.com;h5.m.jd.com,nBGPS1afbRtmBKchokEkGj71HXg"
        }
```



## 领京豆内置H5

1、苹果审核互动类H5需要内置，故领京豆首页RN转H5采用内置方案；

2、940以下版本RN不支持新特性的lottie库，所以940以下版本需要展示H5，通过投放入口版本控制和ARES平台进行版本控制实现；

3、RN加载失败时H5用于兜底；



##  iOS内置包集成注意

1、需要H5单独的集成权限；

2、hybrid平台-高级设置-实时性检查 选项不要勾选；

3、选择正式版本，正式分支

4、选择上传文件，尽量是特有的js文件，公共js文件会共用；

5、上传完文件返回发布的时候需要重新选择最新的发布版本；



## 领京豆PV埋点新增自定义参数统计H5占比

格式：

0/1_1.0

说明：

第一位0代码RN，1代表H5； 第二位代表功能版本号。例如1.0、1.1、1.2。。。

示例：

0_1.0（RN的1.0功能版本），0_1.1（RN的1.1功能版本），1_1.0（H5的1.0功能版本），1_1.1（H5的1.1功能版本）

数据统计计算方法（新功能可表示为RN转H5的适配需求）：

H5占比 = （第一位为1的PV）/（第一位为0的PV + 第一位为1的PV）

H5新功能覆盖率 = （H5新功能版本号PV之和）* 切量比例（如无默认为1）/（H5新功能版本号PV之和 + RN新功能版本号PV之和）

tips: 添加PV参数之前的内置包统计不到相关数据