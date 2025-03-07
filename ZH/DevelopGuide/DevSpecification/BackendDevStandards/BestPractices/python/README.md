
# Python

Python 🐍 最佳实践、优化思路、工具选择。

# 目录 <!-- omit in toc -->
  - [内置数据结构](#内置数据结构)
    - [BBP-1001 避免魔术数字](#bbp-1001-避免魔术数字)
    - [BBP-1002 不要预计算字面量表达式](#bbp-1002-不要预计算字面量表达式)
    - [BBP-1003 优先使用列表推导或内联函数](#bbp-1003-优先使用列表推导或内联函数)
  - [内置模块](#内置模块)
    - [BBP-1004 使用 operator 模块替代简单 lambda 函数](#bbp-1004-使用-operator-模块替代简单-lambda-函数)
      - [替代相乘函数](#替代相乘函数)
      - [替代索引获取函数](#替代索引获取函数)
      - [替代属性获取函数](#替代属性获取函数)
    - [BBP-1005 logging 模块：尽量使用参数，而不是直接拼接字符串](#bbp-1005-logging-模块尽量使用参数而不是直接拼接字符串)
    - [BBP-1006 使用 `timedelta.total_seconds()` 代替 `timedelta.seconds()` 获取相差总秒数](#bbp-1006-使用-timedeltatotal_seconds-代替-timedeltaseconds-获取相差总秒数)
    - [BBP-1007 在协程中使用 `asyncio.sleep()` 代替 `time.sleep()`](#bbp-1007-在协程中使用-asynciosleep-代替-timesleep)
    - [BBP-1008 当在**非测试**代码中使用 `assert` 时，妥善添加断言信息](#bbp-1008-当在非测试代码中使用-assert-时妥善添加断言信息)
  - [生成器与迭代器](#生成器与迭代器)
    - [BBP-1009 警惕未激活生成器的陷阱](#bbp-1009-警惕未激活生成器的陷阱)
    - [BBP-1010 使用现代化字符串格式化方法](#bbp-1010-使用现代化字符串格式化方法)
    - [BBP-1011 在有分支判断时使用yield要记得及时return](#bbp-1011-在有分支判断时使用yield要记得及时return)
  - [函数](#函数)
    - [BBP-1012 统一返回值类型](#bbp-1012-统一返回值类型)
    - [BBP-1013 增加类型注解](#bbp-1013-增加类型注解)
    - [BBP-1014 不要使用可变类型作为默认参数](#bbp-1014-不要使用可变类型作为默认参数)
    - [BBP-1015 优先使用异常替代错误编码返回](#bbp-1015-优先使用异常替代错误编码返回)
  - [面向对象编程](#面向对象编程)
    - [BBP-1016 使用 dataclass 定义数据类](#bbp-1016-使用-dataclass-定义数据类)
      - [在数据量较大的场景下，需要在结构的便利性和性能中做平衡](#在数据量较大的场景下需要在结构的便利性和性能中做平衡)
  - [异常处理](#异常处理)
    - [BBP-1017 避免含糊不清的异常捕获](#bbp-1017-避免含糊不清的异常捕获)
  - [工具选择](#工具选择)
    - [BBP-1018 使用 PyMySQL 连接 MySQL 数据库](#bbp-1018-使用-pymysql-连接-mysql-数据库)
    - [BBP-1019 使用 dogpile.cache 做缓存](#bbp-1019-使用-dogpilecache-做缓存)
    - [BBP-1020 使用Arrow来处理时间相关转换](#bbp-1020-使用arrow来处理时间相关转换)
  - [风格建议](#风格建议)
    - [BBP-1021 对条件判断，在不需要提前返回的情况下，尽量推荐使用正判断](#bbp-1021-对条件判断在不需要提前返回的情况下尽量推荐使用正判断)


## 内置数据结构

### BBP-1001 避免魔术数字

不要在代码中出现 [Magic Number](https://en.wikipedia.org/wiki/Magic_number_(programming))，常量应该使用 Enum 模块来替代。

```python
# BAD
if cluster_type == 1:
    pass


# GOOD
from enum import Enum

Class BCSType(Enum):
    K8S = 1
    Mesos = 2

if cluster_type == BCSType.K8S.value:
    pass
```


### BBP-1002 不要预计算字面量表达式

如果某个变量是通过简单算式得到的，应该保留算式内容。不要直接使用计算后的结果。

```python
# BAD
if delta_seconds > 950400:
    return

# GOOD
if delta_seconds > 11 * 24 * 3600:
    return
```

### BBP-1003 优先使用列表推导或内联函数

使用列表推导或内联函数能够清晰知道要生成一个列表，并且更简洁

```python
# BAD
list_two = []
for v in list_one:
    if v[0]:
        list_two.append(v[1])

# GOOD one
list_two = [v[1] for v in list_one if v[0]]

# GOOD two
list_two = list(filter(lambda x: x[0], list_one))
```

## 内置模块

### BBP-1004 使用 operator 模块替代简单 lambda 函数

在很多场景下，`lambda` 函数都可以用 `operator` 模块来替代，后者效率更高。

#### 替代相乘函数

```python
# BAD
product = reduce(lambda x, y: x * y, numbers, 1)

# GOOD
from operator import mul
product = reduce(mul, numbers, 1)
```

#### 替代索引获取函数

```python
# BAD
rows_sorted_by_city = sorted(rows, key=lambda row: row['city'])

# GOOD
from operator import itemgetter
rows_sorted_by_city = sorted(rows, key=itemgetter('city'))
```

#### 替代属性获取函数

```python
# BAD
products_by_quantity = sorted(products, key=lambda p: p.quantity)

# GOOD
from operator import attrgetter
products_by_quantity = sorted(products, key=attrgetter('quantity'))
```

### BBP-1005 logging 模块：尽量使用参数，而不是直接拼接字符串

在使用 `logging` 模块打印日志时，请尽量 **不要** 在第一个参数内拼接好日志内容（不论是使用何种方式）。正确的做法是只在第一个参数提供模板，参数由后面传入。

在大规模循环打印日志时，这样做效率更高。

参考：https://docs.python.org/3/howto/logging.html#optimization

```python
# BAD
logging.warning("To iterate is %s, to recurse %s" % ("human", "divine"))

# BAD，但并非不可接受
logging.warning(f"To iterate is {human}, to recurse {divine}")

# GOOD
logging.warning("To iterate is %s, to recurse %s", "human", "divine")
```

### BBP-1006 使用 `timedelta.total_seconds()` 代替 `timedelta.seconds()` 获取相差总秒数

```python
from datetime import datetime
dt1 = datetime.now()
dt2 = datetime.now()

# BAD
print((dt2 - dt1).seconds)

# GOOD
print((dt2 - dt1).total_seconds())
```

在源码中，seconds 的计算方式为：days, seconds = divmod(seconds, 24*3600)

表达式右侧 seconds 是总秒数，被一天的总秒数取模得到 seconds

```python
@property
def seconds(self):
    """seconds"""
    return self._seconds

# in the `__new__`, you can find the `seconds` is modulo by the total number of seconds in a day
def __new__(cls, days=0, seconds=0, microseconds=0,
            milliseconds=0, minutes=0, hours=0, weeks=0):
    seconds += minutes*60 + hours*3600
    # ...
    if isinstance(microseconds, float):
        microseconds = round(microseconds + usdouble)
        seconds, microseconds = divmod(microseconds, 1000000)
        # ! 👇
        days, seconds = divmod(seconds, 24*3600)
        d += days
        s += seconds
    else:
        microseconds = int(microseconds)
        seconds, microseconds = divmod(microseconds, 1000000)
        # ! 👇
        days, seconds = divmod(seconds, 24*3600)
        d += days
        s += seconds
        microseconds = round(microseconds + usdouble)
    # ...
```

`total_seconds` 可以得到一个准确的差值：
```python
def total_seconds(self):
    """Total seconds in the duration."""
    return ((self.days * 86400 + self.seconds) * 10**6 +
        self.microseconds) / 10**6
```

### BBP-1007 在协程中使用 `asyncio.sleep()` 代替 `time.sleep()`

`time.sleep()` 是阻塞的，协程执行到此会导致整体事件循环卡住

`asyncio.sleep()` 非阻塞，事件循环将运行其他逻辑

```python
import time
import asyncio

# BAD
async def execute_task(task_id: int):
    print(f"task[{task_id}] hello")
    time.sleep(1)
    print(f"task[{task_id}] world")

# GOOD
async def execute_task(task_id: int):
    print(f"task[{task_id}] hello")
    await asyncio.sleep(1)
    print(f"task[{task_id}] world")
```

上述例子将通过以下代码执行：

```python
import asyncio
async def main():
    await asyncio.gather(task(1), task(2))
await main()
```

`BAD` 将输出以下内容，`task[1]` 执行完 `hello`  后被 `time.sleep()` 阻塞
```
task[1] hello
task[1] world
task[2] hello
task[2] world
```

`GOOD` 将输出以下内容，`task[1]` 执行完 `hello` 后，`await asyncio.sleep(1)` 将挂起 `task[1]`，开始执行 `task[2]`
```
task[1] hello
task[2] hello
task[1] world
task[2] world
```

### BBP-1008 当在**非测试**代码中使用 `assert` 时，妥善添加断言信息

```python
>>> assert 1 == 0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError
```

在大段的日志中，单独存在的 `AssertionError` 不利于日志检索和问题定位，所以添加上可读的断言信息是更推荐的做法。

```python
# BAD
assert "hello" == "world"

# GOOD
assert "hello" == "world", "Hello is not equal to world"
```

## 生成器与迭代器

### BBP-1009 警惕未激活生成器的陷阱

调用生成器函数后，拿到的对象是处于“未激活”状态的生成器对象。比如下面的 `get_something()` 函数，当你调用它时并不会抛出 `ZeroDivisionError` 异常：

```python
>>> def get_something():
...     yield 1 / 0
...
>>> get_something()
<generator object get_something at 0x10d2301b0>
```

如果对该对象使用布尔判断，将永远返回 `True`。

```python
>>> bool(get_something())
True
```

激活生成器可以使用下面这些方式：

```python
# 使用 list 内建函数
>>> list(get_something())
Traceback (most recent call last):
... ...
ZeroDivisionError: division by zero

# 使用 next 内建函数
>>> next(get_something())
Traceback (most recent call last):
... ...
ZeroDivisionError: division by zero

# 使用 for 循环
>>> for i in get_something(): pass
...
Traceback (most recent call last):
... ...
ZeroDivisionError: division by zero
```

### BBP-1010 使用现代化字符串格式化方法

在需要格式化字符串时，请使用 [str.format()](https://docs.python.org/3/library/stdtypes.html#str.format) 或 [f-strings](https://docs.python.org/3/reference/lexical_analysis.html#f-strings)。

```python
# BAD
metric_name = '%s_clsuter_id' % data_id

# GOOD
metric_name = '{}_cluster_id'.format(data_id)

# BEST
metric_name = f'{data_id}_cluster_id'
```

当需要重复键入格式化变量名时，使用 `f-strings`。

```python
# BAD
"{a}-{b}-{c}-{d}".format(a=a, b=b, c=c, d=d)

# GOOD
f"{a}-{b}-{c}-{d}"
```


### BBP-1011 在有分支判断时使用yield要记得及时return

有时我们常会把 yield 类同于 return，为了减少一些循环次数，我们常会把 return 直接替换成 yield，然而这时候就很容易出现 bug，尤其是当一个函数中有多个条件分支时。

```python
# BAD
def test(a: int):
    if a > 1:
        yield "a"
    yield "b"

list(test(2)) # 预期是 ["a"]， 实际是 ["a", "b"]，因为 yield 仅是让度 CPU 而非结束当前函数

# GOOD
def test(a: int):
    if a > 1:
        yield "a"
        return # 控制好函数的生命周期，以达到预期效果
    yield "b"
```

## 函数

### BBP-1012 统一返回值类型

单个函数应该总是返回同一类数据。

```python
# BAD
# 可能返回 bool 或者 None
def is_odd(num):
    if num % 2 > 0:
        return True


# GOOD
def is_odd(num):
    if num % 2 > 0:
        return True
    return False
```

### BBP-1013 增加类型注解

对变量和函数的参数返回值类型做注解，有助于通过静态检查减少类型方面的错误。

```python
# BAD
def greeting(name):
    return 'Hello ' + name

# GOOD
def greeting(name: str) -> str:
    return 'Hello ' + name
```

### BBP-1014 不要使用可变类型作为默认参数

函数作为对象定义的时候就被执行，默认参数是函数的属性，它的值可能会随着函数被调用而改变。

```python
# BAD
def foo(li: list = []):
    li.append(1)
    print(li)

# GOOD
def foo(li : Optional[list] = None):
    li = li or []
    li.append(1)
    print(li)
```
调用两次foo函数后，不同的输出结果：
```python
# BAD
[1]
[1,1]

# GOOD
[1]
[1]
```

### BBP-1015 优先使用异常替代错误编码返回

当函数需要返回错误信息时，以抛出异常优先。

```python
# BAD
def disable_agent(ip):
    if not IP_DATA.get(ip):
        return {"code": ErrorCode.UnknownError, "message": "没有查询到 IP 对应主机"}


# GOOD
def disable_agent(ip):
    """
    :raises: 当找不到对应信息时，抛出 UNABLE_TO_DISABLE_AGENT_NO_MATCH_HOST
    """
    if not IP_DATA.get(ip):
        raise ErrorCode.UNABLE_TO_DISABLE_AGENT_NO_MATCH_HOST
```

## 面向对象编程


### BBP-1016 使用 dataclass 定义数据类

对于需要在初始化阶段设置很多属性的数据类，应该使用 dataclass 来简化代码。但同时注意不要滥用，比如一些实例化参数少、没有太多“数据”属性的类仍然应该使用传统 `__init__` 方法。

```python
# BAD
class BcsInfoProvider:
    def __init__(self, project_id, cluster_id, access_token , namespace_id, context):
        self.project_id = project_id
        self.cluster_id = cluster_id
        self.namespace_id = namespace_id
        self.context = context

# GOOD
from dataclasses import dataclass

@dataclass
class BcsInfoProvider:
    project_id: str
    cluster_id: str
    access_token: str
    namespace_id: int
    context: dict
```
#### 在数据量较大的场景下，需要在结构的便利性和性能中做平衡

很多场景下，我们会得到比较大的原始数据（比如数万个嵌套的 `dict`），为了更便利地操作这些数据，往往会选择通过 `class` 进行实例化，但基于 Python 孱弱的 CPU 计算性能，这一操作可能会耗时过于久。

所以需要在两方面做平衡：
- 保持原始数据，获得最好的性能，但是不方便操作
- 使用 `NamedTuple` 类似的结构，获得一定的结构便利性，但相较于原始数据，会牺牲一定性能
- 使用 `dataclass` 或者 `class` 等方式，保证最大的结构便利性，但会非常影响性能


## 异常处理

### BBP-1017 避免含糊不清的异常捕获

不要捕获过于基础的异常类，比如 Exception / BaseException，捕获这些会扩大处理的异常范围，容易隐藏其他本来不应该被捕获的问题。

尽量捕获预期内可能出现的异常。

```python
# BAD
try
    func_code()
except:
    # your code

# GOOD
try:
    func_code()
except CustomError as error:
    # your code
except Exception as error:
    logger.exception("some error: %s", error)
```

## 工具选择

### BBP-1018 使用 PyMySQL 连接 MySQL 数据库

建议使用纯 Python 实现的 [PyMySQL](https://github.com/PyMySQL/PyMySQL) 模块来连接 MySQL 数据库。如果需要在项目中完全替代 MySQL-python 模块，可以使用模块提供的猴子补丁功能：

```python
import pymysql
pymysql.install_as_MySQLdb()

# 如果版本号无法通过框架检查，可以在导入模块后动态修改
setattr(pymysql, 'version_info', (1, 2, 6, "final", 0))
```

### BBP-1019 使用 dogpile.cache 做缓存

`dogpile.caches` 扩展性强，提供了一套可以对接多中后端存储的缓存 API，推荐作为项目中缓存的基础库。

样例代码：

```python
from dogpile.cache import make_region

region = make_region().configure('dogpile.cache.redis')  # 其他参数官方文档


@region.cache_on_arguments(expiration_time=3600)
def get_application(username):
    # your code


@region.cache_on_arguments(expiration_time=3600, function_key_generator=ignore_access_token) # function_key_generator 参考官方文档
def get_application(access_token, username):
    # your code
```

### BBP-1020 使用Arrow来处理时间相关转换

如果想要转换一个时间，可以将任意对象扔给arrow，然后转成对应的函数格式

常见的时间格式:

  - object: datetime对象(`datetime.datetime.now()`)，区分时区
  - int: 整数timestamp(`int(time.time())`)，不区分时区
  - string: 代表时间的字符串(`2022-11-08 22:57:22`)(`str(datetime.datetime.now())`)，区分时区

这些对象统一都可以扔给arrow.get(任意对象)，得到一个arrow对象`arr`

  - 转成datetime对象: arr.

    ```python
    # 带时区
    In [18]: arr.datetime
    Out[18]: datetime.datetime(2022, 11, 8, 22, 57, 22, 171057, tzinfo=tzlocal())

    # 不带时区
    In [19]: arr.naive
    Out[19]: datetime.datetime(2022, 11, 8, 22, 57, 22, 171057)
    ```    

  - 转成timestamp整数: 
  
    ```python
    In [20]: arr.timestamp
    Out[20]: 1667919442
    ```

  - 转成字符串: 

    ```python
    In [21]: arr.strftime("%Y-%m-%d %H:%M:%S")
    Out[21]: '2022-11-08 22:57:22'
    ```

最佳实践（普通时间 → Unix时间戳(Unix timestamp)）

```python
# BAD
In [22]:  int(time.mktime(time.strptime('2022-11-08 22:57:22+0800', '%Y-%m-%d %H:%M:%S%z')))
Out[22]: 1667919442

# GOOD
In [23]: arrow.get('2022-11-08 22:57:22+0800').timestamp
Out[23]: 1667919442
```

无须自己指定时间格式，直接转换即可


更多请查看:
- 文档：https://arrow.readthedocs.io/en/latest/
- 仓库：https://github.com/arrow-py/arrow

## 风格建议

### BBP-1021 对条件判断，在不需要提前返回的情况下，尽量推荐使用正判断

在判断结果前加否，代码可读性变差，让人的理解成本增加，后续维护也不方便

```python
# BAD
if not validated_data["data_type"] == "cleaned":
    kafka_config = data_id_info["mq_config"]
else:
    kafka_config = data_id_info["result_table_list"][0]["shipper_list"][0]

# GOOD
if validated_data["data_type"] == "cleaned":
     kafka_config = data_id_info["result_table_list"][0]["shipper_list"][0]
else:
    kafka_config = data_id_info["mq_config"]  
```
