# SQL注入

## SQL注入的介绍

SQL注入一直是信息安全中老生常谈的话题，通过SQL注入可以查询到数据库的很多信息，造成数据库信息泄漏；可以通过操作数据库对特定的网页特定的信息进行篡改；执行恶意指令，控制相关的服务器；获取管理员的相关信息，从而进行进一步的攻击。注入攻击的本质就是把用户输入的数据当作代码来执行。这里有两个关键性的条件，第一个是用户能够控制输入；第二个是原本程序要执行的代码，拼接了用户输入的数据。

## SQL注入的方式

SQL注入有以下几种方式：

|    按数据类型     |           按返回类型            |
| :---------------: | :-----------------------------: |
| 数字型（Integer） |     显错注入（ERROR-BASE）      |
| 字符型（String）  | 盲注（Boolean/Time Base-Blind） |

在探测是否有注入点时，可以尝试使用显错注入或者盲注。

### 1. 显错注入

这里以Sqli-lib为例展示一下，在末尾加入单引号后会提示报错信息：

![aAXxaT.png](https://s1.ax1x.com/2020/07/28/aAXxaT.png)

可以推测执行的SQL语句为`SELECT XXX WHERE id='1'' LIMIT 0,1`，这种方式是从错误信息中得到攻击时所需要的信息。但是大部分情况下服务器并不会返回错误结果给用户，这个时候就需要用到盲注。盲注就是构造一个恒等式或者不等式来判断网页的不同显示情况。

### 2. 盲注

在正常情况下，网页应如下显示：

![aAjCRJ.png](https://s1.ax1x.com/2020/07/28/aAjCRJ.png)

在网址后面输入‘1=2’时，网页不会显示任何消息：

![aAj9G4.png](https://s1.ax1x.com/2020/07/28/aAj9G4.png)

由于‘1=2’为假，所以这个等式永远无法成立。通过上述方式可以判断存在SQL注入。这样的方式成为布尔盲注。

基于时间的盲注同理，也是通过构造相应的条件来判断服务器是否延时给出回应，从而推断是否存在注入点。

### 3. 攻击的常见流程

通过上述方式检测到具有注入点之后，便可以围绕注入点进行一系列的操作。

- 猜解当前表中有多少列存在，使用“order by n”。使用二分法逐渐缩小范围，只到猜解出正确的数值。

  当猜解为10列时，网页显示如下：

  ![aAXLMn.png](https://s1.ax1x.com/2020/07/28/aAXLMn.png)

  接下来取中间值5，仍然报错，可以确定范围在1-5之间。

  ![aAXbxs.png](https://s1.ax1x.com/2020/07/28/aAXbxs.png)

  对1-5之间的每一个数进行尝试，最终可以得出该表总共有3列。

- 已知有3个字段后，尝试使用联合查询来猜解出显示位。构造如下语句**http://xxxx/sqli/Less-1/?id=1' union select 1,2,3--+**，如果没有显示的话可以尝试将“id=1”替换为“id=-1”。

  ![aAXOrq.png](https://s1.ax1x.com/2020/07/28/aAXOrq.png)

  可以得出2，3列为显示位。这样就可以在2，3列插入要查询的信息来获得回显。在MySQL数据库中，information_schem数据库存储了数据库的各种相关数据，所以可以从该数据库入手进行相关查询。

  通过构造**http://xxxx/sqli/Less-1/?id=-1‘ union select 1,group_concat(schema_name),3 from information_schema.schemata--+**可以获取当前服务器存在的所有数据库的名字。

  ![aAXvZV.png](https://s1.ax1x.com/2020/07/28/aAXvZV.png)

- 已知了所有的数据库之后，可以去尝试去查询数据库里所有的表名，这里查询“security”数据库里所有的表。构造**http://xxxx/sqli/Less-1/?id=-1' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema=database()--+**

  ![aAXXq0.png](https://s1.ax1x.com/2020/07/28/aAXXq0.png)

- 可以判断在users表中包含了用户的登陆信息，接下来尝试查询users表中的所有信息，构造**http://xxxx/Less-1/?id=-1' union select 1,group_concat(column_name),3 from information_schema.columns where table_name='users'--+**可以获得表中的所有信息。

  ![aAXzIU.png](https://s1.ax1x.com/2020/07/28/aAXzIU.png)

- 尝试从当前表中获取到username和password，构造**http://xxxx/Less-1/?id=-1' union select 1,username,password from users where id=2--+**，更换不同的id值，可以查看不同用户的信息。

  ![aAjpiF.png](https://s1.ax1x.com/2020/07/28/aAjpiF.png)

