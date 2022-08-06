#  Gradle插件

<img src="../img/image-20211229231354887.png" alt="image-20211229231354887" style="zoom:50%;" />

## Gradle构建脚本基础

![image-20211229231430526](../img/image-20211229231430526.png)

## 项目构建生命周期

![image-20211229231501628](../img/image-20211229231501628.png)

![image-20211229231525382](../img/image-20211229231525382.png)

![image-20211229231557505](../img/image-20211229231557505.png)

![image-20211229231748131](../img/image-20211229231748131.png)

![image-20211229231828593](../img/image-20211229231828593.png)

![image-20211229231929952](../img/image-20211229231929952.png)

## Project

![image-20211229232003838](../img/image-20211229232003838.png)

![image-20211229232025552](../img/image-20211229232025552.png)

![image-20211229232155946](../img/image-20211229232155946.png)

![image-20211229232217944](../img/image-20211229232217944.png)

![image-20220103122507695](../img/image-20220103122507695.png)

### project相关api

![image-20220103133315115](../img/image-20220103133315115.png)

### 属性相关api

### ![image-20220103135607479](../img/image-20220103135607479.png)

### File相关api

![image-20220103150420210](../img/image-20220103150420210.png)

![image-20220103152005574](../img/image-20220103152005574.png)

![image-20220103152443908](../img/image-20220103152443908.png)

### 其他api

![image-20220103152922585](../img/image-20220103152922585.png)

#### 依赖相关api

![image-20220103153255043](../img/image-20220103153255043.png)

 ![image-20220103153848770](../img/image-20220103153848770.png)



![image-20220103154717314](../img/image-20220103154717314.png)

![image-20220103155028411](../img/image-20220103155028411.png)

![image-20220103155240480](../img/image-20220103155240480.png)

![image-20220103155634139](../img/image-20220103155634139.png)

![image-20220103155650889](../img/image-20220103155650889.png)

![image-20220103160638301](../img/image-20220103160638301.png)

#### 外部命令执行

![image-20220103161355230](../img/image-20220103161355230.png) 

## Task

![image-20211230221839943](../img/image-20211230221839943.png)

![image-20211230221822299](../img/image-20211230221822299.png)

![image-20211230221953690](../img/image-20211230221953690.png)

![image-20211230222017844](../img/image-20211230222017844.png)

![image-20211230222111409](../img/image-20211230222111409.png)

![image-20211230222149263](../img/image-20211230222149263.png)

![image-20211230222240659](../img/image-20211230222240659.png)

![image-20211230222306954](../img/image-20211230222306954.png)

![image-20220103170108283](../img/image-20220103170108283.png)

 ![image-20220103171217632](../img/image-20220103171217632.png)

动态依赖：

![image-20220103173730309](../img/image-20220103173730309.png)

```groovy
task handleReleaseFile {
  def srcFile = file('releases.xml')
  def destDir = new File(this.buildDir, 'generated/release/')
  doLast {
    println '开始解析对应的xml文件...'
    destDir.mkdir()
    def releases = new XmlParser().parse(srcFile)
    releases.release.each { releaseNode ->
      //解析每个release结点的内容
      def name = releaseNode.versionName.text()
      def versionCode = releaseNode.versionCode.text()
      def versionInfo = releaseNode.versionInfo.text()
      //创建文件并写入结点数据
      def destFile = new File(destDir, "release-${name}.txt")
      destFile.withWriter { writer -> writer.write("${name} -> ${versionCode} -> ${versionInfo}")
      }
    }
  }
}

task handleReleaseFileTest(dependsOn: handleReleaseFile) {
  def dir = fileTree(this.buildDir.path + 'generated/release/')
  doLast {
    dir.each {
      println 'the file name is:' + it
    }
    println '输出完成...'
  }
}
```

### Task输入输出

![image-20220103175205647](../img/image-20220103175205647.png)

```groovy
/**
 * 描述：版本发布文档自动维护脚本
 * 流程描述：  1、请求本次版本相关信息
 *           2、将版本相关信息解析出来
 *           3、将解析出的数据生成xml格式数据
 *           4、写入到已有的文档数据中
 **/
ext {
  versionName = rootProject.ext.android.versionName
  versionCode = rootProject.ext.android.versionCode
  versionInfo = 'App的第2个版本，上线了一些最基础核心的功能.'
  destFile = file('releases.xml')
  if (destFile != null && !destFile.exists()) {
    destFile.createNewFile()
  }
}

//挂接自定义task到构建过程中
this.project.afterEvaluate { project ->
    def buildTask = project.tasks.getByName('build')
    if (buildTask == null) {
        throw GradleExceptiion('the buld task is not found')
    }
    buildTask.doLast {
        writeTask.execute()
    }
}

task writeTask {
  inputs.property('versionCode', this.versionCode)
  inputs.property('versionName', this.versionName)
  inputs.property('versionInfo', this.versionInfo)
  outputs.file this.destFile
  doLast {
    //将输入的内容写入到输出文件中去
    def data = inputs.getProperties()
    File file = outputs.getFiles().getSingleFile()
    //将map转化为实体对象
    def versionMsg = new VersionMsg(data)
    //将实体对象写入到xml文件中
    def sw = new StringWriter()
    def xmlBuilder = new MarkupBuilder(sw)
    if (file.text != null && file.text.size() <= 0) {
      //没有内容
      xmlBuilder.releases {
        release {
          versionCode(versionMsg.versionCode)
          versionName(versionMsg.versionName)
          versionInfo(versionMsg.versionInfo)
        }
      }
      //直接写入
      file.withWriter { writer -> writer.append(sw.toString())
      }
    } else {
      //已有其它版本内容
      xmlBuilder.release {
        versionCode(versionMsg.versionCode)
        versionName(versionMsg.versionName)
        versionInfo(versionMsg.versionInfo)
      }
      //插入到最后一行前面
      def lines = file.readLines()
      def lengths = lines.size() - 1
      file.withWriter { writer ->
        lines.eachWithIndex { line, index ->
          if (index != lengths) {
            writer.append(line + '\r\n')
          } else if (index == lengths) {
            writer.append('\r\r\n' + sw.toString() + '\r\n')
            writer.append(lines.get(lengths))
          }
        }
      }
    }
  }
}

class VersionMsg {
  String versionCode
  String versionName
  String versionInfo
}

task readTask {
  //指定输入文件为上一个task的输出
  inputs.file this.destFile
  doLast {
    //读取输入文件的内容并显示
    def file = inputs.files.singleFile
    println file.text
  }
}

task taskZ {
  dependsOn writeTask, readTask
  doLast {
    println '输入输出任务结束'
  }
}
```



releases.xml

```xml
<releases>
  <release>
    <versionCode>100</versionCode>
    <versionName>1.0.0</versionName>
    <versionInfo>App的第1个版本，上线了一些最基础核心的功能.</versionInfo>
  </release>


<release>
  <versionCode>110</versionCode>
  <versionName>1.1.0</versionName>
  <versionInfo>App的第2个版本，上线了一些最基础核心的功能.</versionInfo>
</release>


<release>
  <versionCode>1</versionCode>
  <versionName>1.0.0</versionName>
  <versionInfo>第8个版本。。。</versionInfo>
</release>
</releases>
```

## Transform

![image-20211230222342511](../img/image-20211230222342511.png)

![image-20211230224349020](../img/image-20211230224349020.png)

## 如何自定义插件

![image-20211230224452715](../img/image-20211230224452715.png)

![image-20211230224646511](../img/image-20211230224646511.png)

![image-20211230225101575](../img/image-20211230225101575.png)

![image-20211230225215087](../img/image-20211230225215087.png)

![image-20211230225255794](../img/image-20211230225255794.png)

![image-20211230224818502](../img/image-20211230224818502.png)

![image-20211230225919508](../img/image-20211230225919508.png)

 ![image-20211230230128273](../img/image-20211230230128273.png)

![image-20211230230353275](../img/image-20211230230353275.png)

![image-20211230230414484](../img/image-20211230230414484.png)

![image-20211230230531035](../img/image-20211230230531035.png)

## Extension简介

![image-20211230230615986](../img/image-20211230230615986.png)

![image-20220109211907329](../img/image-20220109211907329.png)

## 字节码插装技术

![image-20211230230819894](../img/image-20211230230819894.png)

![image-20211230230841408](../img/image-20211230230841408.png)

![image-20211230231127716](../img/image-20211230231127716.png)

![image-20211230231143332](../img/image-20211230231143332.png)

...

![image-20211230231323728](../img/image-20211230231323728.png)

...

![image-20211230231357687](../img/image-20211230231357687.png)

## Gradle模块

![image-20220104213806285](../img/image-20220104213806285.png)

### sourceSets

```grovvy
sourceSets {
  main {
  	jniLibs.srcDirs = ['libs'] //修改so库存放位置

    res.srcDirs = [
      'src/main/res',
      'src/main/res-ad',
      'src/main/res-player']
    }
}
```









