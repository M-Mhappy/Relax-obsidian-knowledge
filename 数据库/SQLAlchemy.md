**SQLAlchemy是Python生态中最著名、最全面的数据库工具包和对象关系映射器（ORM）**[-1](https://sqlalchemy.org.cn/features.html)[-5](https://pypi.org/project/SQLAlchemy/?source=post_page-----6e7fd7bf1769--------------------------------)[-8](https://sqlalchemy.org.cn/#bugs)。它的核心目标是让开发者既能最大限度地发挥SQL的灵活性和强大功能，又能用优雅的Python代码与数据库交互，且不丢失对底层细节的控制[-5](https://pypi.org/project/SQLAlchemy/?source=post_page-----6e7fd7bf1769--------------------------------)[-9](https://www.sqlalchemy.org/philosophy.html)。

简单来说，它就像一个“万能数据库适配器”，让你能用一套标准的Python代码，去操作PostgreSQL、MySQL、SQLite等多种不同的数据库[-1](https://sqlalchemy.org.cn/features.html)[-7](https://www.oreilly.com/library/view/sqlalchemy-he-xin-zhi-nan-di-2/9798341659087/preface02.html)。

### 🏗️ 两大核心组件

SQLAlchemy的设计非常灵活，由两个可以独立或协同使用的核心部分组成[-1](https://sqlalchemy.org.cn/features.html)[-2](https://docs.sqlalchemy.org.cn/en/20/intro.html#installation)：

1. **Core (核心)**：这是SQLAlchemy的基础，是一个功能完备的SQL抽象工具包[-1](https://sqlalchemy.org.cn/features.html)[-5](https://pypi.org/project/SQLAlchemy/?source=post_page-----6e7fd7bf1769--------------------------------)。你可以使用它提供的 **SQL表达式语言**，通过Python对象来程序化地构建SQL语句，而无需编写原始的SQL字符串。它还负责管理数据库连接池、处理数据类型转换等底层工作[-2](https://docs.sqlalchemy.org.cn/en/20/intro.html#installation)[-4](https://docs.sqlalchemy.org.cn/en/20/tutorial/index.html)。
    
2. **ORM (对象关系映射器)**：这是一个构建在Core之上的可选高级组件[-1](https://sqlalchemy.org.cn/features.html)[-2](https://docs.sqlalchemy.org.cn/en/20/intro.html#installation)。它的核心思想是 **“工作单元”** 模式，允许你**将Python类映射到数据库中的表，然后通过操作这些Python对象来读写数据库**[-1](https://sqlalchemy.org.cn/features.html)[-2](https://docs.sqlalchemy.org.cn/en/20/intro.html#installation)[-3](https://www.sqlalchemy.org/features.html)。ORM会自动将你对对象属性的修改，同步成数据库里的INSERT、UPDATE、DELETE语句，并在事务提交时批量执行，这极大地提高了开发效率[-1](https://sqlalchemy.org.cn/features.html)[-5](https://pypi.org/project/SQLAlchemy/?source=post_page-----6e7fd7bf1769--------------------------------)。

### Pydantic 是一个 Python 数据校验库，核心作用：
1.请求体校验:自动校验客户端发送的 JSON 数据是否符合模型定义
2.响应体序列化:将模型数据自动转换为 JSON 响应
## 定义 Pydantic 模型

创建一个继承 `BaseModel` 的类，使用 Python 标准类型声明字段：
## 实例

from pydantic import BaseModel  
class Item(BaseModel):  
    name: str               # 必填：商品名称  
    description: str | None = None  # 可选：商品描述  
    price: float            # 必填：商品价格  
    tax: float | None = None        # 可选：税费
Item就是一个pydantic模型

# Pydantic Schema — 数据验证与序列化
## 为什么需要 Schema（格式）？

ORM 模型（models.py）定义数据库结构，但不能直接用于 API 的请求/响应校验。
原因：ORM 模型包含数据库专用逻辑（如 relationship），且不希望将所有字段暴露给 API。
Pydantic Schema 是纯数据类，专门负责 API 的输入输出校验和序列化。
这三者的关系：
客户端 → [Pydantic Schema 校验] → 路由函数 → [ORM 模型] → 数据库
客户端 ← [Pydantic Schema 序列化] ← 路由函数 ← [ORM 模型] ← 数据库

| **  <br>** | Base                   | **BaseModel**                  |     |
| ---------- | ---------------------- | ------------------------------ | --- |
| **属于谁**    | SQLAlchemy ORM         | Pydantic                       |     |
| **作用**     | ORM 模型的**基类**，用来定义数据库表 | Pydantic 模型的**基类**，用来定义数据结构    |     |
| **继承方式**   | `class User(Base):`    | `class UserCreate(BaseModel):` |     |
| **实例代表 **  | 数据库表中的**一行记录**         | 一个**数据结构**（请求/响应/配置）           |     |

微信小程序（手机）
    ↓ HTTP POST JSON
[网关层] Nginx 反向代理
    ↓
[Pydantic 入境] 校验参数合法性
    ↓ 通过
[业务层] 检查库存、计算金额、生成订单号
    ↓
[SQLAlchemy ORM] 生成 INSERT/UPDATE SQL
    ↓
[数据库] PostgreSQL 提交事务（原子操作）
    ↓ 返回自增ID + 时间戳
[SQLAlchemy ORM] 刷新对象（回填数据库生成的值）
    ↓
[Pydantic 出境] 转换格式、隐藏敏感字段、格式化时间
    ↓
[FastAPI] 自动 JSON 序列化
    ↓ HTTP 200 OK
微信小程序展示"下单成功"