# SCUtils - 用操作 DOM 的方式操作玩家！
### 许可
**请注意，本插件使用 GPLv3 进行许可。不在每个文件加那段话只是为了美观而不代表该插件就不是 GPLv3 了。**

## 这是啥玩意
**SCUtils** 是一个基于 BukkitAPI 的 Minecraft 服务端综合性插件。**SCUtils** 完全由 SCLeo (hhttll) 独立开发。目前而言，**SCUtils** 提供了以下三大功能。

1. MineQuery
2. 实用命令
3. 对外实用类

## MineQuery
**MineQuery** 是一门受到 jQuery 启发的 **表达式语言**，用于强大、快速、方便地从服务器地在线玩家中选择出所需要地玩家。其编译器及解析器位于 net.minegeck.plugins.scutils.minequery。**MineQuery** 与 jQuery 看起来很像，但实际上完全不一样。jQuery 是基于 JavaScript 的一个库，而 **MineQuery** 本身是一门独立的语言，与 JavaScript 同级。**MineQuery** 从头到尾只是一个表达式，没有语句的概念，因为其所有目的就是为了计算并表示一个玩家的集合。现在让我们来看一看 **MineQuery** 的几个例子吧。

```
$(#SCLeo)
```

上面这段代码的作用是选择所有名字是 `SCLeo` 的玩家。很明显，在 jQuery 里面，选择器的内容需要用引号包起来，因为那是一个传入 $ 这个函数的字符串。但是在 **MineQuery** 里并不需要那么做，因为在 **MineQuery** 里 `#SCLeo` 是一个 **ID 选择器**，`$(...)` 是一个 **匿名查询选择器** 就好比 "3" 在 JavaScript 里是一个数字一样，是被语言规范所明确的。**匿名查询选择器** 的完整格式是 `$(选择器表达式1, 选择器表达式2, ...)`。每一个 **选择器表达式** 有可能会是一个 **查询选择器单元**，也有可能是若干个 **查询选择器单元** 通过 **选择器表达式运算符** 连接。每一个 **查询选择器单元** 都会包括若干个 **选择器** 和 **限制器**。**选择器** 有很多种，比如刚开始的 **匿名查询选择器** 就是选择器的一种（是的没错，他是可以无限嵌套的）。除此之外，**选择器** 还有如下几种：

- 名字选择器
- ID 选择器
- 类选择器
- 属性选择器
- 查询选择器

### 名字选择器 (Name Selector) 与 ID 选择器 (ID Selector)
**名字选择器** 与 **ID 选择器** 十分相似。他们的唯一区别在于，**名字选择器** 的选择标准是 Bukkit 的 getPlayer(String name) 方法，而 **ID 选择器**的标准是 getExactPlayer(String name) 方法。**名字选择器** 允许模糊匹配（SC 可匹配 SCLeo，scleo2 等等，但不可匹配 CSLeo）。**ID 选择器** 则要求用户名完全一致，完全一致包括大小写也应该正确。

**名字选择器** 的格式非常简单: 直接书写名字。是的你没有看错，直接写名字。比如要匹配 SCLeo，可以直接写 `SC`。

**ID 选择器** 的格式稍微复杂: 你需要在被匹配的 ID 前加上一个井号，如 `#SCLeo`。

### 类选择器 (Class Selector)
**类选择器** 的效果是选择拥有指定类的玩家。至于什么是类，大体上可以理解就是 Minecraft 官方的 scoreboard 里的 tag。添加及删除类的方法将在之后介绍 class 命令的时候给出。

**类选择器** 的格式和 **ID 选择器** 很像: 在你需要匹配的类名前加上一个点，如 `.foo` 就可以匹配所有拥有 *foo* 这个类的玩家。

### 属性选择器 (Property Selector)
**属性选择器** 是所有基础选择器里面最复杂的。他将会匹配属性符合一定条件的玩家。一个最简单的例子是 `[level == 5]` 将会选择所有等级是 5 的玩家。当然，你还可以通过 `[level == 5 && gamemode == 0]` 来选择处于生存模式且等级是 5 的玩家。具体的语法将在后面介绍 **MineQuery** 表达式的时候给出。

**属性选择器** 的格式比较复杂: 你需要用中括号将你的条件包裹起来，如果你有多个条件（需要全部满足才匹配），你需要用逗号分割这几个条件（当然，多条件也可以用 `&&` 操作符完成）。这里是一个使用多个条件的例子: `[level > 3, hp < 15]`。

### 查询选择器 (Query Selector)
**查询选择器** 相当于函数，你需要事先通过 `qs` 命令来将一个选择器保存到 **查询选择器** 然后你才能使用这个 **查询选择器**。举个例子，你可以使用 `/qs set anyname .a` 来将 `.a` 这个类选择器保存到 `anyname` 这个查询选择器，然后你就可以使用 `$anyname` 来代替 `.a`。**查询选择器** 在代替极其复杂的复合选择器时十分有用，因为你不用多次重复这个复合选择器。更重要的是，**查询选择器** 是预编译的，也就是说它只在创建的时候以及服务器重新载入的时候被编译。之后就直接调用的是编译后的结果。这将极大地提高性能。如果可能的话，你应该将任何在命令方块中执行的选择器替换为 **查询选择器**。

**查询选择器** 的格式并不复杂，你只需要在目标 **查询选择器** 的名字前加一个美元符号，如 `$anyname` 就会调用名为 `anyname` 的 **查询选择器**。

### 限制器 (Limiter)
**限制器** 是在选择结果中进行进一步筛选的选择器（不过它不是严格意义上的选择器因为它依赖于别的选择器）。限制器的格式是 `:限制器类型(目标数量)`，如果目标数量是 1，那么就可以省略为 `:限制器类型`。**限制器** 目前一共只有三种: TOP, BOTTOM 和 RANDOM。顾名思义，TOP 就是从上一个选择结果中选择最顶端的那几个玩家出来。RANDOM 则选择随机的几个。BOTTOM 选择最底下的那几个。限制器不区分大小写。例如 `.asd:top(5)` 就会选择拥有 asd 这个类的前 5 个玩家。

当 **限制器** 被使用时，那么他和被限制的选择器就构成了查询选择器单元。例如其实刚刚例子中的 `.asd:top(5)` 就是一个查询选择器单元。

### 查询选择器单元 (Query Selector Unit)
**查询选择器单元** 是由若干个选择器和限制器构成的。每一个 **查询选择器单元** 必须要有至少一个任意类型的选择器，**限制器** 则是可有可无的。**查询选择器单元** 的工作原理是，让第一个选择器在所有玩家范围内筛选出符合条件的玩家，然后让第二个选择器在第一个选择器的选择结果中筛选出符合第二个条件的玩家，以此类推。当所有选择器都被依次执行后，其最后的结果会被依次送入限制器中。例如 `.asd.qwe:top(3):random` 这个 **查询选择器单元** 首先会在所有玩家中挑选出有 asd 这个类的所有玩家，然后，在这批玩家中挑选出拥有 qwe 这个类的玩家 （也就是说如果玩家数量很大，第二步操作会比第一步负担小很多，因为挑选的范围小了）。比如这时候，同时拥有 asd 和 qwe 的玩家还有 10 个。那么这 10 个玩家就会被送入 :top(3) 这个限制器中，那么这 10 个玩家中的后 7 个就会被淘汰。最后，剩下的三个被送入 :random 这个限制器中，随机挑选出一个作为这个查询选择器单元的最终结果。

**查询选择器单元** 的格式是: `选择器1 选择器2 ... 选择器n 限制器1 限制器2 ... 限制器 n`。需要注意的是，限制器永远在选择器后面。

### 二元选择器 (Binary Selector)
**二元选择器** 是对于多个 **查询选择器单元** 结果的集合操作。请注意，虽然都是正规的集合操作，二元选择器使用 四则运算符 来表示不同的操作，这个在数学中是没有的。（数学中有自己的集合运算符，但是很难输入）如果你要用，你会被你的数学老师打死的。

**二元选择器** 非常类似普通的加减乘除，甚至还有运算优先级（可惜没有括号，因为括号可以通过匿名查询选择器来实现）。**二元选择器** 一共有以下 4 种: 并集，使用符号 `+`; 差集，使用符号 `-`; 交集，使用符号 `*`; 对等差集，使用符号 `/`。如果你不知道这四个操作是什么意思，那么请跳过这个选择器，因为我不打算教你怎么弄集合。

并集和差集的优先级小于交集和对等差集，也就是说 `.a + .b * .c` 将会先求 .b 和 .c 的交集，然后再与 .a 求并集。如果要先求 .a 和 .b 的并集，你可以这样: `$(.a + .b) * .c`。

请再次注意，二元选择器的输入是两个 **查询选择器单元**，也就是说 .a.b + .c 会被理解为是一个要求同时 a 和 b 的查询选择器单元 与 c 这个单元求并集。

### 匿名查询选择器 (Anonymous Query Selector)
**匿名查询选择器** 也就是复合查询选择器。 **匿名查询选择器** 需要提供若干个 **二元选择器** 或 **查询选择器单元** 作为输入，然后返回所有输入的选择器的并集。这里有一个例子: `$(.a, .b)` 将会返回有 a 或者 b 的类的玩家。当然，这和 `$(.a + .b)` 是等效的。我管这个叫语法糖 233。**匿名查询选择器** 还有一种类型就是不接受任何输入，就像这样: `$()`，在这种情况下，括号可以被省略（`$`），现在，这个匿名查询选择器的作用就是选择所有的玩家。

好了，以上就是选择器的所有内容，接下来开始介绍 MineQuery 表达式。

### MineQuery 表达式
一个表达式返回一个值，这句话永远不会错。不过需要注意的是我们把一个集合，一个列表也当一个值。目前，MineQuery 还不支持集合或者列表。MineQuery 只支持以下三种类型的数据: 数字， 字符串 和 布尔值。也就是说，表达式就是一个用来表示这三种数据类型中的一种的方式。

比如: `1` 就是一个合法的表达式，它返回了数字 1。

`"asd"` 也是一个合法的表达式，它返回了字符串 "asd"。

`5 > 3` 也是一个合法的表达式，它返回了布尔值 True（真，即 5 确实大于 3。）请注意，和别的编程语言一样，MineQuery 可以直接写数字常量和字符串常量。但是与别的语言不一样的是，MineQuery 不允许 布尔值的常量。也就是说你写 true，会被解析器认为是一个叫做 true 的变量。

说到变量，你一定很好奇，变量是怎么来的。那么我就来告诉你，变量相当于是玩家的属性。比如 `level + 3` 返回的就是玩家的等级加上 3。那么这个玩家究竟是谁呢，这个有点难理解，大概意思就是说同一个表达式对于不同的玩家有不同的计算结果。这也就是和 **属性选择器** 对上的地方了。**属性选择器** 要求提供一个表达式，那么你就得填一个表达式，比如 `[level > 5]`。那么，这个 **属性选择器** 就会依次对于每一个玩家计算 `level > 5`，对于等级大于 5 的玩家，这个表达式就会返回 True，而对于等级小于等于 5 的玩家，这个表达式就会返回 False。因此 `[level > 5]` 这个 **属性选择器** 会选择所有等级大于 5 的玩家。

MineQuery 表达式一共支持以下这些二元操作符（就是写在两个值之间的）：

```
&& 与操作符 优先级 3
|| 或操作符 优先级 2
> 大于操作符 优先级 7
>= 大于或等于操作符 优先级 7
== 等于操作符 优先级 7
<= 小于或等于操作符 优先级 7
< 小于操作符 优先级 7
!= 不等于操作符 优先级 7
+ 加法操作符 优先级 10
- 减法操作符 优先级 10
* 乘法操作符 优先级 20
/ 除法操作符 优先级 20
% 求余操作符 优先级 20
// 整除操作符 优先级 20
& 字符串连接操作符 优先级 10
```

需要注意的有两个，一个是整除操作符，其作用是获得整数部分，例如 5//2 = 2 因为 5/2 = 2...1。还有一个是字符串连接使用 & 而不是 +。

MineQuery 表达式还支持两个一元操作符（就是写在一个值之前的）：

```
- 取负操作符
! 取反操作符
```

具体每个操作符是做什么的，我真的不想解释了，这个解释起来就是一学期的编程课啊...

### 什么时候可以使用玩家选择器
所有 SCUtils 提供的命令中凡是需要填写玩家的地方都可以使用玩家选择器。部分命令只能接受 1 个玩家作为输入，这时候就需要用 限制器 来限制数量了。在部分 SCUtils 命令中，命令的参数可以是变量。比如在 **/velocity** 中就是这样。`/velocity $ level` 就可以把所有玩家的垂直速度设置为他的等级。等级越高飞的速度越快。

如果配置文件中的 **为所有命令启用 MineQuery 选择器** 选项被开启且使用者拥有 `scutils.globalqueryselector` 这个权限节点，那么所有命令都将可以使用 MineQuery 选择器。但是这里有个特殊要求就是所有的选择器必须是 **匿名查询选择器** 或者 **查询选择器**，也就是说，在别的命令中使用的选择器都是要用 $ 开头的。变量在这里也是支持的，使用 `{%变量名%}` 来使用变量。比如 `/effect $ 1 {%level%}` 会给所有玩家以其等级为时长的 1 级速度效果。请注意，这个选项仅对玩家输入的命令有效，服务器后台或者命令方块中如果要使用请使用 **exec** 命令。

在 exec 和 命令序列中，MineQuery 选择器总是启用的。无论是否开启 **为所有命令启用 MineQuery 选择器**， `/exec effect $ 1 {%level%}` 都将工作正常。在 exec 和 命令序列中，MineQuery 选择器的用法和上一条一样。

### 变量
刚刚提到了变量，我相信很多人现在就开始好奇了变量到底有哪些。那么这个部分就会来介绍变量到底是如何工作的。

首先，一个变量是由两个部分组成的: 命名空间 和 变量名。如果命名空间被省略则默认使用 `property`。也就是说你刚刚用的那个变量 **level** 实际上是 **property:level** 的简写。命名空间目前有三个（有计划做成可扩展的，但是还没有实现）: property, data 和 bukkit。

#### 命名空间 property
Property 是一个玩家的基础属性，以下是一个完整的 property 中的变量列表:

##### level, lvl, lv
玩家的等级

##### exp, xp
玩家的经验

##### flying, fly
玩家是否在飞行

##### sleeping, sleep
玩家是否在睡觉

##### op
玩家是否是 op

##### canfly
玩家是否可以飞

##### gamemode
玩家的游戏模式编号

##### name
玩家的名字

##### customname
玩家的自定义名字

##### displayname
玩家的显示名字

##### playerlistname
玩家在玩家列表中显示的名字

##### falldistance
玩家的掉落距离

##### food, foodlevel
玩家的饥饿度

##### hp, health
玩家的生命值

##### walkspeed
玩家的行走速度

##### flyspeed
玩家的飞行速度

##### insidevehicle
玩家是否在车内

##### sneaking, sneak
玩家是否在潜行

##### sprinting, sprint, running, run
玩家是否在奔跑

##### xpos, xposition
玩家的 X 坐标

##### ypos, yposition
玩家的 Y 坐标

##### zpos, zposition
玩家的 Z 坐标

##### yaw
玩家的水平转向角度

##### pitch
玩家的垂直转向角度

##### world, worldname
玩家所在世界名

##### xvel, xvelocity
玩家的 X 轴速度

##### yvel, yvelocity
玩家的 Y 轴速度

##### zvel, zvelocity
玩家的 Z 轴速度

#### 命名空间 bukkit
bukkit 中的所有变量都是你的服务端原生提供的。本插件使用反射调用，直接获取返回结果。因为 bukkit 的那些方法有些不只是 getter，还会有 setter 之类的，所以这个命名空间是极其危险的。（就目前而言，由于 setOp 之类的方法需要提供输入值，所以会被本插件忽略。但是我不敢保证未来一定没有不需要参数的危险操作出现。如果过于危险了，请告诉我，我会增加更细的权限控制到这里来。）

bukkit 命名空间内的变量直接使用方法名，你可以在这里获取所有可以用的方法名: https://hub.spigotmc.org/javadocs/spigot/org/bukkit/entity/Player.html。

这是一个使用 bukkit 命名空间的一个例子: `[bukkit:getSaturation > 3]`。这个选择器会选择所有饱食度大于 3 的玩家。

#### 命名空间 data
data 相当于每一个玩家独立的变量，你可以通过 /data set 命令来给一个玩家设置一个数据。设置完成后就可以在这里读取了。具体的设置办法放在后面讲 data 这个命令的地方介绍。

`[data:haha > 3]` 会选择 haha 这个变量大于 3 的玩家。需要注意的是，如果一个玩家没有 haha，那么在为他计算时，`data:haha` 会返回 False。

## 实用命令
**请注意，你随时可以使用命令 `/uhelp <命令>` 来查询一个命令的完整帮助信息。`/uhelp` 来获得命令列表。**

### class 命令
**别名: uclass, scuclass**

**class** 命令用于查看或编辑一个玩家的类。该命令一共有 10 种形式：

#### 为一个或多个玩家添加一个类
如果玩家名被忽略，则使用使用命令的玩家，之后所有命令基本上都是这样，就不多解释了。

```
/class add <类名>
/class add <玩家> <类名>
```

#### 为一个或多个玩家删除一个类

```
/class remove <类名>
/class remove <玩家> <类名>
```

#### 为一个或多个玩家切换一个类的状态
如果原来有，切换完就没了。如果原来没有，切换完就有了。

```
/class toggle <类名>
/class toggle <玩家> <类名>
```

#### 列出一个玩家的所有类
如果选择了多个玩家，请使用限制器限制到 1 个玩家。s

```
/class
/class <玩家>
/class list
/class list <玩家>
```

### cs 命令
**别名: ucs, scucs, scucommandsequence, ucommandsequence, commandsequence**

**cs** 命令用于操作命令序列。命令序列相当于一个命令列表。执行这个命令序列将会依次执行序列中的每一个命令。若要执行命令序列，请使用命令 **rcs**。该命令一共有 8 种形式。

#### 列出所有可用的命令序列。

```
/cs list
```

#### 删除一个命令序列

```
/cs delete <序列名>
```

#### 创建一个命令序列

```
/cs create <序列名>
```

#### 在一个命令序列的指定命令之后插入一条命令

```
/cs insert <序列名> <位置> <命令>
```

#### 从一个命令序列中删除一个命令

```
/cs remove <序列名> <位置>
```

#### 替换一个命令序列中的命令

```
/cs replace <序列名> <位置> <命令>
```

#### 在一个命令序列中追加一个命令

```
/cs append <序列名> <命令>
```

#### 列出一个命令序列中的所有命令

```
/cs list <序列名>
```

### data 命令
**别名: udata, scudata**

**data** 命令用于修改或查看一个或多个玩家的数据。该命令一共有 8 种形式。

#### 设置一个或多个玩家的某个数据
如果值填写 null 则删除该值。请注意值可以是个表达式。比如 `/data set $ time data:time + 1` 可以把所有玩家 time 这个数据 + 1。（蛤？）

```
/data set <变量名> null
/data set <变量名> <值>
/data set <玩家> <变量名> null
/data set <玩家> <变量名> <值>
```

#### 列出一个玩家的所有数据
请保证只选择了一个玩家。

```
/data
/data <玩家>
/data list
/data list <玩家>
```

### exec 命令
**别名: uexec, scuexec**

**exec** 命令相当于一个用来执行命令的命令。其主要作用有两个，一是用来在命令方块中使用 MineQuery 选择器；二是在一个命令中执行多个简单命令（使用竖线（` | `）封割，竖线前后需要有空格。）。（如果要更复杂的命令，请使用命令序列。）该命令一共有 1 种形式，那就是:

```
/exec <命令>
```

举例: `/exec say a | say b`

### explain 命令
**别名: uexplain, scuexplain**

**explain** 命令用于展示一个 MineQuery 选择器或表达式的 AST（抽象语法树）。如果你不知道 AST 是什么，跳过即可。该命令一共有 4 种形式。

#### 分析一个 MineQuery 选择器

```
/explain sel <选择器>
/explain selector <选择器>
```

#### 分析一个 MineQuery 表达式

```
/explain exp <表达式>
/explain expression <表达式>
```

### uhelp 命令
**别名: scuhelp**

**uhelp** 命令用来列出所有可用的命令或者获得一个命令的详细用法。该命令一共有 3 种形式。

#### 列出所有的命令

```
/uhelp
/uhelp <页码>
```

#### 获得一个命令的详细信息

```
/uhelp <命令>
```

### nocmd 命令
**别名: unocmd, scunocmd**

**nocmd** 命令用来禁止或解除禁止一个玩家使用命令。**nocmd** 在制作小游戏的时候应该非常有效。请注意如果一个玩家被禁止使用命令，为了防止自己禁止自己，凡是拥有 `scutils.overridernocmd` 这个权限节点的玩家不会被禁止使用命令。该命令一共有 6 种形式。

#### 使一个或多个玩家禁止使用命令

```
/nocmd on <玩家>
/nocmd on <玩家> <原因>
```

#### 使一个或多个玩家取消禁止使用命令

```
/nocmd off <玩家>
/nocmd off <玩家> <原因>
```

#### 使一个或多个玩家切换是否禁止使用命令

```
/nocmd toggle <玩家>
/nocmd toggle <玩家> <原因>
```

### qs 命令
**别名: uqs, scuqs, scuqueryselector, uqueryselector, querselector**

**qs** 命令用于操作存储的查询选择器。查询选择器的用法已在上方介绍查询选择器的地方介绍，这里只介绍如何操作存储的查询选择器。该命令一共有 6 种形式。

#### 列出所有保存的查询选择器

```
/qs
/qs list
```

#### 显示一个查询选择器的内容

```
/qs show <查询选择器名>
/qs showast <查询选择器名>
```

#### 删除一个查询选择器

```
/qs remove <查询选择器名>
```

#### 设置一个查询选择器的内容

```
/qs set <查询选择器名> <内容>
```

### rcs 命令
**别名: urcs, scurcs, scuruncommandsequence, uruncommandsequence, runcommandsequence**

**rcs** 命令用于运行命令序列。该命令一共有 2 种形式。

#### 以自己的身份运行一个命令序列

```
/rcs <序列名>
```

#### 以 Console 的身份运行一个命令序列

```
/rcs console <序列名>
```

### select 命令
**别名: uselect, scuselect**

**select** 命令用于试验一个玩家选择器，他只有一种形式:

```
/select <玩家选择器>
```

### tokenize 命令
**别名: utokenize, scutokenize**

和 **explain** 一样，**tokenize** 是一个内部调试用的命令。他可以用来查看一个语句 tokenize 的结果。该命令只有一种形式:

```
/tokenize <MineQuery 语句>
```

### velocity 命令
**别名: uvelocity, scuvelocity, vel**

**velocity** 命令用于设置一个或多个玩家的运动速度。请注意，这个命令是允许使用变量的，例如 `/velocity $ level`。该命令一共有 4 种形式。

```
/velocity <X 速度> <Y 速度> <Z 速度>
/velocity <Y 速度>
/velocity <玩家> <X 速度> <Y 速度> <Z 速度>
/velocity <玩家> <Y 速度>
```

## 权限
本插件的权限相当简单明了，首先每一个命令都有一个权限:

```
scutils.command.class
scutils.command.cs
scutils.command.data
scutils.command.exec
scutils.command.explain
scutils.command.help
scutils.command.nocmd
scutils.command.qs
scutils.command.rcs
scutils.command.select
scutils.command.tokenize
scutils.command.velocity
```

允许被 nocmd 以后还使用命令的权限:

```
scutils.overridernocmd
```

允许在所有命令中使用 MineQuery Selector 的权限（需要在配置文件中开启，虽然默认是开的）

```
scutils.globalqueryselector
```

以上所有权限节点默认 OP 可用。以下几个复合节点默认没有任何人拥有。

```
scutils.*
scutils.command.*
```

## 其他信息
本项目以 GPLv3 发布，所有文件头部标有 许可 = "GPLv3" 的文件都应该视为受 GPLv3 条款限制。（不加那个注释的原因是那个注释太长了，请不要玩文字游戏，谢谢。）

本项目除了以下类外，均为原创:

 - net.minegeck.plugins.utils.StringSimilarity (59 行)

所有来自外部的类均已在头部标明来源。

截止到 v0.0.1 本项目总共含有 5712 行代码（不含注释及空行）。