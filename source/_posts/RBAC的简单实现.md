---
title: RBAC 的简单实现
date: 2017-05-07
categories:
  - 系统分析
tags:
  - RBAC
---

> 以下文章描述了一个构建 RBAC Demo 的过程，全文上上下下都透露出了一种“偷懒”的意志——明确目的，能不做就不做。

RBAC 有多种实现方式，下面使用其中最基础的一种(RBAC0)来构建系统。

根据本文所述，只使用不到 100 行标准 ES6 代码实现了 RBAC0，参见我的 [GitHub Repo: zccz14/RBAC-DEMO](https://github.com/zccz14/RBAC-DEMO)。

也有一个线上部署的 Demo：[https://zccz14.com/RBAC-DEMO](https://zccz14.com/RBAC-DEMO)

<!--more-->

## 分析

为了简化实现与部署，方便读者理解 RBAC，demo有以下特性：

+ 采用文件进行 RBAC 配置

  数据库在 RBAC 中不是必须的，RBAC 的数据模式才是必须的。数据库是一种数据持久化的方式，比起直接用文件管理有诸多优点，例如可以通过约束保证数据的一致性；有完整的操作方法与操作理论；可以管理大规模的数据；可以进行并发控制；……数不胜数。**但是**，作为 RBAC 的 demo 而言，不需要考虑那些使用数据库的好处。

  参见下图中的 RBAC 鉴权时序图中的数据库与其他部分间的交互接口：**系统向数据库请求权限指派，而数据库返回对应角色的权限集**。

  ```java
  // Java 描述的 RBAC 存储库的接口 (示意)
  public interface RBACRepository {
    Set<Permission> getPermission(Role role);
  }
  ```

  服务器拿到权限集以后判断用户会话是否包含用户请求的权限，如果有就许可，否则驳回。

  ![RBAC](https://zccz14.com/images/2017/05/08/RBAC_s0.svg)

  那么很简单的一个基于接口替换：

  ![RBAC_s1](https://zccz14.com/images/2017/05/08/RBAC_s1.svg)

  简单地用配置文件取代了数据库，为什么可以这样呢？

  首先角色、权限、用户指派与权限指派都与业务逻辑有关，而业务逻辑是一个相对稳定的东西。

  配置文件使其可配置以应对一定的变化，同时又提供了足够简单的操作接口（文件系统）方便系统管理员直接维护配置。

  如果系统规模较大，当然是选择数据库啦！但是 Demo 嘛，现在提供一个不随大流的非数据库想法，我觉得甚好。

+ 基于 Web 的纯前端实现

  没有后端，意味着交互逻辑将大大地被简化。使用 Web 是为了原生跨平台。另外，部署难度与所需资源都会降低到最低。

  将所有逻辑都放到前端上，RBAC 配置也作为一个资源文件被浏览器读取并解析。

+ 不考虑前端被绕过的情况

  在 Web 应用中，前后端通过 HTTP 协议通讯，高级(skilled)用户完全可以绕过前端设定的逻辑直接给服务器发送请求。这可能会绕过前端的检查逻辑，进而对系统的一致性造成破坏。在 demo 中，不存在后端，因此每个用户使用的系统都是独立的，高级用户没有恶意破坏系统的动机，因为这不会带来任何的利益。他们可能只是任着好奇心来进行类似拆开电视机的行动。

+ 不假设系统的角色范围

  一般的 RBAC 都会分普通用户、管理员、超级管理员来讲述，但 RBAC 并不假设这些，但 RBAC0 确实隐式要求有一个超级管理员能进行配置。

  根据上述描述，超级管理员直接操作配置文件了，因此这个 RBAC Demo 中的超级管理员就被隐藏在配置文件之后了。

## 数据逻辑建模

遵循着最简原则进行建模，但不失拓展性。

### 角色

仅包含角色 ID。在真实系统的需求分析阶段就确定角色集合。

### 权限

仅包含权限 ID。在真实系统的设计阶段确定权限集合。

### 用户

仅包含用户 ID。因为 Demo 并不关心其他的，甚至不关心你的密码。

在真实系统中还是要验证用户身份的。

### 权限指派

仅包含角色 ID 与权限 ID。由业务逻辑决定，相对稳定。

### 用户指派

仅包含用户 ID 与角色 ID。变动较频繁，数据量也较大。

## 数据物理建模

实际上，如果我们不关心用户、角色、权限的其他属性，可以简单地使用两个指派表来表示所有的数据。

比如这样的：

> UA.csv (User Assignment)

```csv
uid, rid
1, 1
2, 3
3, 2
1, 2
*, *
```

> PA.csv (Permission Assignment)

```csv
rid, pid
1, 1
1, 2
1, 3
2, 1
2, 2
2, 4
3, 5
*, 1
*, 2
*, 3
*, 4
*, 5
```

上面这些数据是个啥意思呢？

**用户具有的角色**以及**角色具备的权限**。

另外我们可以统计出：

有 4 个用户，ID 分别为 1, 2, 3, *

有 4 个角色，ID 分别为 1, 2, 3, *

有 5 个权限，ID 分别为 1, 2, 3, 4, 5

CSV，是一种简易的表示二维表的文件格式，方便易读，Excel 也支持它。

## 实现细节

### 读取 RBAC 配置

```javascript
var RBAC = {};

const fetchCSV = (url) => fetch(url)
    .then(v => v.text())
    .then(v => v.split('\n'))
    .then(v => v.slice(1)
        .filter(v => v)
        .map(v => v.split(',')
            .map(v => v.trim())
        )
    );

fetchCSV('UA.csv').then(v => { RBAC.UA = v; });
fetchCSV('PA.csv').then(v => { RBAC.PA = v; });
```

这段代码是加载 CSV 格式的 RBAC 配置的，主要就是整理数据用的。

### RBAC 资源组件

我们要在web文档中设定要通过 RBAC 控制的资源组件：

```html
<div class="rbac" data-pid="1">
  <button>required permission 1</button>
</div>
```

比如说这样的，一个 DOM 节点具备 `rbac` 这个类名，并且有 `data-pid` 属性。

上面这个 DOM 在用户会话具备权限1时才会被用户看到，否则这个 DOM 节点对用户不可见。

### 角色激活事件

我们需要发布一个自定义事件来分离逻辑。

```javascript
class RoleActiveEvent extends Event {
    constructor(role) {
        super('roleActive');
        this.role = role;
    }
}

function onRIDChange() {
    let roleId = document.getElementById('rid').value;
    let permissionIds = RBAC.PA.filter(v => v[0] === roleId).map(v => v[1]);
    window.dispatchEvent(new RoleActiveEvent({
        id: roleId,
        permissions: permissionIds
    }));
}
```

这个 `roleActive` 事件就由之前带有 `rbac` 类的资源组件来监听了：

```javascript
// add listener
Array.from(document.getElementsByClassName('rbac')).forEach(function (el) {
    el.classList.add('hide');
    window.addEventListener('roleActive', function (event) {
        let pid = el.dataset.pid;
        if (event.role.permissions.includes(pid)) {
            el.classList.remove('hide');
        } else {
            el.classList.add('hide');
        }
    })
})
```
