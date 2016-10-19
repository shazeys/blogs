# Android Studio混淆规则
因为只用AS开发Android，所以我就不管Eclipse的混淆了
标签： 混淆 ProGuard

----
### 概念
>**起因**：因为Java代码是非常容易反编码的，况且Android开发的应用程序是用Java代码写的，为了很好的保护Java源代码，我们需要对编译好后的class文件进行混淆。

>**目的**：恶心反编译的人

>**ProGuard**：ProGuard是一个开源的Java代码混淆器，只混淆java代码；SDK已经集成了ProGuard工具，位于sdk文件夹的\tools\proguard下

>**作用**
1. 压缩: 移除无效的类、属性、方法等
2. 优化: 优化字节码，并删除未使用的结构
3. 混淆: 将类名、属性名、方法名混淆为难以读懂的字母，比如a,b,c

### 注意事项
>**不能混淆**

- 在AndroidManifest中配置的类，比如四大组件(避免ClassNotFoundException)
- JNI调用的方法
- 反射用到的类
- WebView中JavaScript调用的方法
- Layout文件引用到的自定义View
- 一些引入的第三方库（一般都会有混淆说明的） 

>**Log处理**

如果TAG的习惯写法是TAG = MainActivity.class.getSimpleName(),无法确定混淆后的文件名是啥，不利于Log的处理
所以，TAG = "MainActivity.class"就好了，爱怎么混淆爱怎么换名字去吧！

>**Crash信息处理**

代码混淆的时候记得加上在混淆文件里面记得加上这句keep住源文件以及行号 :
-keepattributes SourceFile,LineNumberTable

### 语法
>**保留**

- -keep {Modifier} {class_specification} 保护指定的类文件和类的成员
- -keepclassmembers {modifier} {class_specification} 保护指定类的成员，如果此类受到保护他们会保护的更好
- -keepclasseswithmembers {class_specification} 保护指定的类和类的成员，但条件是所有指定的类和类成员是要存在。
- -keepnames {class_specification} 保护指定的类和类的成员的名称（如果他们不会压缩步骤中删除）
- -keepclassmembernames {class_specification} 保护指定的类的成员的名称（如果他们不会压缩步骤中删除）
- -keepclasseswithmembernames {class_specification} 保护指定的类和类的成员的名称，如果所有指定的类成员出席（在压缩步骤之后）
- -printseeds {filename} 列出类和类的成员-keep选项的清单，标准输出到给定的文件

>**压缩**

- -dontshrink 不压缩输入的类文件
- -printusage {filename}
- -whyareyoukeeping {class_specification}

>**优化**

- -dontoptimize 不优化输入的类文件
- -assumenosideeffects {class_specification} 优化时假设指定的方法，没有任何副作用
- -allowaccessmodification 优化时允许访问并修改有修饰符的类和类的成员

>**混淆**

- -dontobfuscate 不混淆输入的类文件
- -obfuscationdictionary {filename} 使用给定文件中的关键字作为要混淆方法的名称
- -overloadaggressively 混淆时应用侵入式重载
- -useuniqueclassmembernames 确定统一的混淆类的成员名称来增加混淆
- -flattenpackagehierarchy {package_name} 重新包装所有重命名的包并放在给定的单一包中
- -repackageclass {package_name} 重新包装所有重命名的类文件中放在给定的单一包中
- -dontusemixedcaseclassnames 混淆时不会产生形形色色的类名
- -keepattributes {attribute_name,…} 保护给定的可选属性，例如LineNumberTable, LocalVariableTable, SourceFile, Deprecated, Synthetic, Signature, and InnerClasses.
- -renamesourcefileattribute {string} 设置源文件中给定的字符串常量

###通配符

| 通配符               | 规则                     |
| :------------------- | :---------------------- |
| ？                   | 匹配单个字符             |
| *       | 匹配类名中的任何部分，但不包含额外的包名 |
| ** |      匹配类名中的任何部分，并且可以包含额外的包名 |
| % |匹配任何基础类型的类型名|
|* | 匹配任意类型名 ,包含基础类型/非基础类型|
|... |匹配任意数量、任意类型的参数 |
| &lt;init>|匹配任何构造器 |
| &lt;ifield>|匹配任何字段名 |
| &lt;imethod>|匹配任何方法 |
| *(当用在类内部时)|匹配任何字段和方法 |
| $|指内部类 |


