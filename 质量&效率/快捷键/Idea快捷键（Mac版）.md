# Idea快捷键（Mac版）

> 转载：[idea快捷键（Mac版）](https://juejin.cn/post/6844903830807642125)

## 1. 快速修复

`option + enter`对象赋值变量

`TypeWildcard typeWildcard = new TypeWildcard();`，在编辑器中输入`new TypeWildcard()`后，然后按`option + enter`，就会生成`TypeWildcard typeWildcard = new TypeWildcard();`，再按`enter`完成对象的创建和赋值

## 2. 光标移动

1. 按页上下移动：`fn + ↑` / `fn + ↓`
2. 光标移动到当前行代码的首尾：`cmd + ←` / `cmd + →`
3. 光标移动到当前单词的首尾：`option + ←` / `cmd + →`
4. 高亮文件中当前元素`cmd + shift + F7`，然后`control + option + ↑` / `control + option + ↓`在高亮元素中快速跳转
5. 光标移动到代码块的开始或结束位置：`cmd + option + [` / `cmd + option + ]`
6. 光标跳转到上次修改的地方：`cmd + shift + delete`，跳转到下次修改的地方没有快捷键，可以使用**菜单-> Navigate-> Next Edit Location**
7. 光标跳转到上/下次浏览(光标停留)的地方：`cmd + option + ←` / `cmd + option + →`
8. 光标跳转到指定的行列：`cmd + L`
9. 光标跳转到当前行的上一行或者下一行，开始新的一行：`shift + enter` / `cmd + option + enter`

## 3. 代码缩进

1. 缩：`tab`，注意回到代码行的开始，才会生效
2. 进：`shift + tab`，这个没有**缩**的限制

## 4. 代码提示

编码时idea自带提示，当光标移到其他地方时，或者按了esc后，代码提示没有了， 此时按`option + /`就有了

快捷键设置：Keymap  ->  搜索basic(Completion)

## 5. 自动结束代码，行末自动添加分号

`cmd + shift + enter`，此功能键，还可以添加一些代码。

```java
String str = "abc";
// 可添加分号
if (str == "abc") {
    
}
// 输入完if (str == "abc")，使用快捷键可添加花括号
```

## 6. 显示方法的参数信息

`cmd + P`，加强版`cmd + 鼠标左键`可查看更多信息（所在类、返回值、参数信息）

## 7. 快速查看文档

`control + J`，查看光标所在元素的文档

## 8. 快速生成一些代码

`cmd + N`，类似的`cmd + O`覆盖方法（重写父类方法）,`cmd + I`实现方法（实现接口中的方法）

![2021-04-11-7XLFEK](https://image.ldbmcs.com/2021-04-11-7XLFEK.jpg)

## 9. 环绕代码

`cmd + option + T`（使用if..else, try..catch, for, synchronized等包围**选中**的代码） 或者生成包围标签

![2021-04-12-fOiash](https://image.ldbmcs.com/2021-04-12-fOiash.jpg)

## 10. 显示意向动作和快速修复代码

`option + enter`，移除只有一条语句的if的花括号等

## 11. 格式化代码

`cmd + option + L`

## 12. 优化import

`control + option + O`

## 13. 复制、剪切、粘贴、删除

1. `cmd + X`剪切当前行或选定的块到剪贴板
2. `cmd + C`复制当前行或选定的块到剪贴板
3. `cmd + D`复制当前行或选定的块
4. `cmd + V`从剪贴板粘贴
5. `cmd + delete`删除当前行或选定的块的行
6. `option + delete`删除到单词的开头
7. `option + fn + delete`删除到单词的结尾
8. `cmd + shift + delete`从最近的缓冲区粘贴

## 14. 折叠展开代码

1. `cmd + +` / `cmd + -`展开 / 折叠代码块
2. `cmd + shift + +`展开所有的代码块
3. `cmd + shift + -`折叠所有代码块

## 15. 关闭当前查看的tab编辑器选项卡

`cmd + W`

## 16. 大小写切换

`cmd + shift + U`

## 17. 查找、替换

1. 双击`shift`，查询任何符号
2. `cmd + F`：文件内查找
3. `cmd + R`：文件内替换
4. `cmd + shift + F`：全局查找（根据路径）
5. `cmd + shift + R`：全局替换（根据路径）
6. `cmd + G`：查找模式为向下查找
7. `cmd + shift + G`：查找模式为向上查找
8. 查找方法在何处调用，`option + F7`全局查找，`cmd + F7`文件查找

## 18. 方法调用层次

`control + option + H`

![2021-04-12-3sw8C7](https://image.ldbmcs.com/2021-04-12-3sw8C7.jpg)

入口：`find action→call hierarchy(control+option+H)`

## 19. 查看类继承结构

`control + H`

入口：`find action→hierarchy actions→hierarchy(control + H)`

## 20. 查看类图

普通的子类名右击`Diagrams→Show diagram(option+shift+command+U)`即可出现类图。

## 21. 查看maven依赖

`pom.xml`中右击选`Maven→show dependencies`可以看到所有的依赖关系。

可以按`command+f`进行搜索

可以右击`exclude`进行排除

## 22. 查看当前field、method大纲

navigate→File Structure（`cmd + F12`） 或者 `cmd + 7`

## 23. 复制文件名

直接点击文件名`cmd + C`在文本编辑区`cmd + V`即可

## 24. 复制文件全名

`cmd + shift + C` 再`cmd + V`即可

## 25. 复制多个文件名

多次`cmd + C`，再`cmd + shift + V`，从最近的缓冲区选择要粘贴哪些文件名

## 26. 快速赋值变量

`cmd + option + V`，将表达式赋值给变量

## 27. 省略中间变量

`cmd + option + N`，将中间过渡的变量省略

## 28. 输入文件名称，实现文件快速跳转

`cmd + shift + O`，输入文件名

## 29. 更改变量名称

`shift + F6`，更改当前文件中变量

## 30. 给选中内容添加双引号、圆括号、花括号

`Preferences | Editor | General | Smart Keys`中勾选**Surround selection on typing quote or brace**，选中内容，`shift + "`

## 31. 选择更大区域范围代码

`option + ↑`

## 32. 双击标识符，选中单词，而不是整个标识符

`Preferences | Editor | General | Smart Keys`中勾选**use CamelHumps words**

## 33. 查看某个类的所有子类

`cmd + option + B`

## 34. 当前文件内查找

`cmd + F12`

## 35. 折叠展开代码

`cmd + .`

**以上快捷键是选择 Mac OS X 10.5+ copy的情况**