## 实用插件

### 外观插件

#### 主题插件 Vuesion Theme

> 代码显示蓝绿清新

#### 主题插件 Gradianto

> 纯绿色的主题

### 主题插件 hibNet 

> 紫色主题

#### 图标插件 Atom Material ICons

#### 翻译插件 Translation

#### 背景图片插件 BackgroundImagePlus

#### 代码特效插件 **Activate-power-mode**



### 代码插件

#### 直接查看jar包插件 File Expander

#### 快捷键提示插件 Key Promoter X 

#### 右侧代码缩略图插件 CodeGlance

#### 自动生成set方法插件 GenerateAllSetter

> 快捷键为alt+enter

#### 关键字搜索插件 codesearch

> 可以直接复制搜索百度谷歌

#### Mybatis xml和代码切换插件 Free-idea-mybatis

> 要在ultimate版本安装

#### Mybatis 插件 mybatisx

> mybatis文件生成工具

#### 搜索url对应的实体类插件 RestfulToolkit

#### 花括号颜色匹配插件 Rainbow Brackets

#### 阿里代码规范检查插件 Ailibaba Java Coding Guidelines

#### java在线诊断工具插件 arthas idea

#### maven依赖搜索插件 maven-search

#### 代码智能补全插件 codota

#### 单元测试插件 Squaretest

#### 单元测试插件 TestMe

#### JSON 插件 json paser

> 可以对 JSON 字符串进行格式化
>

#### JSON 插件 Java Bean to Json

> 支持将 Java Bean 转成 JSON
>

#### JSON插件 GsonFormatPlus

> JSON转成JAVA bean

#### 代码注释插件 Easy Javadoc



## LeetCode插件

### 模板配置

#### CodeFileName 类名

```java
$!velocityTool.replace(${question.title}," ","")

```

#### CodeTemplate 类内容配置

```java
package leetcode.editor.cn;

${question.content}
public class $!velocityTool.replace(${question.title}," ",""){
	public static void main(String[] args) {
		Solution solution = new $!velocityTool.replace(${question.title}," ","")().new Solution();
		
	}
${question.code}
}

```

#### 特别注意

> 在生成的自定义代码中包含两行关键信息:
> //leetcode submit region begin(Prohibit modification and deletion):提交到leetcode进行验证的代码开始标记
> //leetcode submit region end(Prohibit modification and deletion):提交到leetcode进行验证的代码结束标记
> 这两行标记标示了提交到leetcode服务器进行验证的代码范围,在此范围内只允许有出现与题目解答相关的内容，出现其他内容可能导致leetcode验证不通过。
> 除了此范围内，其他区域是可以任意填写的，内容不会提交到leetcode，可以增加一些可以本地调试的内容，例如:import java.util.Arrays;
> 所以，这两行内容是不能被删除和修改的，否则将识别不到提交的内容。
