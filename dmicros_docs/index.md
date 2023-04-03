# 用户指南-(Dmicros-v0.8.0)

欢迎阅读！此份用户指南对应公开版人群分类可视化服务。

API的地址是[api-sic.iamhd.top](https://api-sic.iamhd.top).

文档修改自早期demo版，可能有错漏之处，请及时反馈或新建pull request进行修订。

**文档并未覆盖全部API用法，我们可以根据集成需求进行相应的增补。** 公开版APP是基于API开发的前后端分离应用，API功能应足以支持集成开发。

## 前言

Dmicros API是提供数据服务的工具API，为用户提供可用于可视化的数据和在线模型计算。

API遵循RESTFul标准，使用[eve-0.9](https://github.com/pyeve/eve)软件包开发，本份指南中将只包括Dmicros独有的设置，其他未尽事项（如错误返回格式等）请您参考[pyeve文档](http://python-eve.org)。

如果您有任何问题或遇到bug，都可以在本项目中进行反馈。为了保证通用性，以下均使用`curl`命令来示例请求。但出于测试方便，推荐您使用[httpie](https://httpie.org)或[postman](https://www.getpostman.com).

示例中的具体数据不是真实数据，主要供参考格式。

## 重要提示

* **API正在进行大版本迭代，因此对于API设计不尽合理的地方，我们尚不能及时修订和部署**
* **数据交换默认使用json格式，如果使用其他的方式传递参数可能会引发没有被单元测试覆盖的错误。请保持json格式的输入**
* **本份文档保密，出于安全考虑请勿外发**

## 获取API令牌（token）

API令牌:

```shell
<主令牌另行发送>
```

API的每项服务都需要使用令牌才能访问，每项服务均有自己的权限要求。

以上令牌已经配置了token管理以外的必要权限，如果还会遇到401等权限错误，则可能是API或初始化脚本的bug，请随时反馈。


## 1. API 服务状态

该API返回可用的服务项目。

**HTTP Request:**

`GET https://api-sic.iamhd.top`

**示例**

```shell
curl https://api-sic.iamhd.top  -H "Authorization:<your access token>"
```

**返回：**

```json
{
  "_links": {
    "child": [
      {
        "href": "vdatas",
        "title": "vdatas"
      },
      {
        "href": "clusteds",
        "title": "clusteds"
      },
      {
        "href": "clusteds/edit",
        "title": "clusteds/edit"
      },
      {
        "href": "clustedProjects",
        "title": "clustedProjects"
      },
      {
        "href": "clustedProjects/reload-clusteds",
        "title": "clustedProjects/reload-clusteds"
      },
      {
        "href": "clustedProjects/append-clusteds",
        "title": "clustedProjects/append-clusteds"
      },
      {
        "href": "tokens",
        "title": "tokens"
      },
      {
        "href": "quotas",
        "title": "quotas"
      }
    ]
  }
}
```

## 2. vdatas API : 可视化数据

本部分与集成无关，已省略，请暂时跳过本节。

该API返回可用于可视化的设置和数据，相应的数据可以应用到[Echarts](http://www.echartsjs.com)上进行可视化。

## 3. quotas API : 配额

该API将返回服务资源配额及使用记录。会消费配额的服务有：

* **创建`clusteds`**
* **创建`clustedProjects`**

**HTTP Request:**

`GET https://api-sic.iamhd.top/quotas`

**示例**

```shell
curl https://api-sic.iamhd.top/quotas  -H "Authorization:<your access token>"
```

**返回：**

```json
{
  "_items": [
    {
      "_id": "5a862f11c3666e1e5654fce9",
      "token": "<your access token>",
      "resource": "clusteds",
      "quota": 4933,
      "quota_acc": 5000,
      "_updated": "Fri, 16 Feb 2018 01:08:33 GMT",
      "_created": "Fri, 16 Feb 2018 01:08:33 GMT",
      "_etag": "09e12f466cb3ac1d25251a460126412688c3f174",
      "_links": {
        "self": {
          "title": "Quota",
          "href": "quotas/5a862f11c3666e1e5654fce9"
        }
      }
    }
  ],
  "_links": {
    "parent": {
      "title": "home",
      "href": "/"
    },
    "self": {
      "title": "quotas",
      "href": "quotas"
    }
  },
  "_meta": {
    "page": 1,
    "max_results": 25,
    "total": 1
  }
}
```

其中：

* `"resource": "clusteds"` : 对应的服务资源
* `"quota": 4933` : 配额余额
* `"quota_acc": 5000` : 配额总量

## 4. clusteds API : 人群分类结果

该API返回人群分类结果：创建人群分类样本，并存储到服务器上，即时返回人群分类结果
 

其中创建的过程需要**消费该项资源的配额**，如果您没有足够的配额，服务器将会返回402错误。

### 4.1 POST : 创建人群分类结果

该API将创建人群分类结果，输入的数据格式为json：

```json
{
  "raw": "654,英菲尼迪Q50L,4,3,6,6,1,2,5,7,35,0,1,2,4,5,4,6,5,6,6,4,5,5,5,5,6,6,5,6,6,5,6,5,5,5,5,5,4,4,5,5,6,5,5,6,4,5,5,5,6,6,5,5,5",
  "v": 2,
}
```

字段`raw`的值即人群分类语句模块问卷测试记录，每一问题得分间用半角逗号`,`隔开。

`raw`的拼装：

* 和手动处理的数据一致，`raw`由48个字段值以半角逗号隔开拼装，如果某个值为空用`''`即可（见示例）。前两位起标识作用
* 虽然md5 hash理论上不保证输入的唯一性，但由于`raw`的实际样本空间不大，目前仍可利用它简易避免重复输入

该API也支持批量创建，输入格式为json array：

```json
[
    {
    "raw": "1,RAV4,5,,6,7,14,1,7,3,5,7,5,6,5,6,6,5,5,3,4,6,6,5,2,6,4,5,6,5,6,5,5,7,7,6,2,7,6,5,5,7,5,7,5,6,6,3,6"
    ...
    },
	...
]
```

---

* `v`字段用于切换不同版本的模型，取值为1或2，类型为Int，1对应老版本模型（2012版），2对应新版本模型（2018版）
* **当前版本请务必使用v=2**
* v=3对应2022版模型，尚未在公版的当前版本实装
* 创建人群分类结果的同时可以传入更多的数据，并会自动添加`user`字段存储创建结果的用户名
* `raw`被计算为`hashed`之后，API会要求`hashed`是**唯一的**，重复输入会导致API返回422错误。批量输入中出现重复，也会导致创建失败
* 当前的hash算法使用md5，有极低的几率（1.47*10^-29）发生碰撞

---

**HTTP Request:**

`POST https://api-sic.iamhd.top/clusteds`

**示例**

*请注意传入的数据保持UTF-8编码，不要传输形如`\u30123`的数据。*

```shell
curl -XPOST https://api-sic.iamhd.top/clusteds  -H "Authorization:<your access token>" -H "Content-Type:application/json" -d '{"raw":"654,英菲尼迪Q50L,4,3,6,6,1,2,5,7,35,0,1,2,4,5,4,6,5,6,6,4,5,5,5,5,6,6,5,6,6,5,6,5,5,5,5,5,4,4,5,5,6,5,5,6,4,5,5,5,6,6,5,5,5","v": 2}'
```

**返回：**

返回中的`summary`包括计算结果，你可以使用`"href": "clusteds/5b16607ec78ef360a703b4f4"`中的地址取回结果。如果进行了批量插入，`sumarry`将包括一个由结果构成的列表。结果包括人群分类的相关编码，请参见本部分最后一小节4.A。

```json
{
  "_updated": "Tue, 05 Jun 2018 10:05:50 GMT",
  "_created": "Tue, 05 Jun 2018 10:05:50 GMT",
  "_etag": "6dc1988e24635490061ad09bc963fb63f3355088",
  "_id": "5b16607ec78ef360a703b4f4",
  "_links": {
    "self": {
      "title": "Clusted",
      "href": "clusteds/5b16607ec78ef360a703b4f4"
    }
  },
  "_status": "OK",
  "summary": "654,英菲尼迪Q50L,9,4,4,3"
}
```

### 4.2 GET : 读取已创建的人群分类结果

该API将读取已经创建的人群分类结果，结果在键值`summary`中。

该API支持通过`_id`和`hashed`码两种方式获取：

* `_id`和包括_id的`href`在创建时的返回结果中
* `hashed`码是输入`raw`的md5 hexdigest（不加盐），因此可以直接在客户端用同样的方法计算得出

**HTTP Request:**

用`hashed`码获取结果

`GET https://api-sic.iamhd.top/clusteds/3e69a5afc5083b3553fa533d8c411bdf`

或

用`_id`获取结果

`GET https://api-sic.iamhd.top/clusteds/5a90f132c78ef35c4730d0df`

**示例**

```shell
curl https://api-sic.iamhd.top/clusteds/3e69a5afc5083b3553fa533d8c411bdf  -H "Authorization:<your access token>"
```

或
```shell
curl https://api-sic.iamhd.top/clusteds/5a90f132c78ef35c4730d0df  -H "Authorization:<your access token>"
```


**返回：**

计算结果如`"summary": "1,RAV4,6,3,3,1"`。

API会自动添加创建结果的用户和结果有效状态：` "user": "dmicros"`和`"is_valid": true`。

```json
{
  "_id": "5b16607ec78ef360a703b4f4",
  "raw": "1,RAV4,5,,6,7,14,1,7,3,5,7,5,6,5,6,6,5,5,3,4,6,6,5,2,6,4,5,6,5,6,5,5,7,7,6,2,7,6,5,5,7,5,7,5,6,6,3,6",
  "summary": "1,RAV4,6,3,3,1",
  "hashed": "3e69a5afc5083b3553fa533d8c411bdf",
  "user": "dmicros",
  "is_valid": true,
  "_updated": "Tue, 05 Jun 2018 10:05:50 GMT",
  "_created": "Tue, 05 Jun 2018 10:05:50 GMT",
  "_etag": "6dc1988e24635490061ad09bc963fb63f3355088",
  "_links": {
    "parent": {
      "title": "home",
      "href": "/"
    },
    "self": {
      "title": "Clusted",
      "href": "clusteds/5b16607ec78ef360a703b4f4"
    },
    "collection": {
      "title": "clusteds",
      "href": "clusteds"
    }
  }
}
```

其中`is_valid`字段默认为`true`，标示样本有效。

### 4.3 PATCH clusteds/edit: 修改已创建的人群分类结果

如需修改人群分类结果，可以使用ENDPOINT `clusted/edit`。它支持通过`PATCH`方法访问结尾是`_id`的地址来修改人群分类结果。（结尾是`hashed`码的地址不支持修改操作）

需要注意的是为了保证并行修改记录不造成冲突，请求头需要加上`If-Match`。

因此，典型的修改流程是先获取目标记录的`_id`和`_etag`，然后再进行实际的修改。

支持进行修改的字段包括(字母序）：

* age
* area_tiers
* birth_year
* brand_intention
* car_body
* car_intention
* car_owned
* city
* clusted_project：可设为空，或修改为已有的clustedProject ID
* date
* date_survey
* education
* gender
* has_car
* income
* is_valid
* price_range
* remote_id
* segment



**HTTP Request:**

通过`id`地址修改人群分类结果：

`PATCH https://api-sic.iamhd.top/clustedsEdit/5b0ec4558690a90011ce2ebf`

**示例**

获得人群分类结果记录中的`_id`和`_etag`

```shell
curl https://api-sic.iamhd.top/clustedsEdit/3e69a5afc5083b3553fa533d8c411bdf -H Authorization:<your access token>
```

得到

```json
{
  "_id": "5b0ec4558690a90011ce2ebf",
  "hashed": "3e69a5afc5083b3553fa533d8c411bdf",
  "is_valid": false,
  "_updated": "Thu, 31 May 2018 11:56:52 GMT",
  "_created": "Wed, 30 May 2018 15:33:41 GMT",
  "_etag": "6381e22d2cc690589fdf3870b9025ecfb7c9b905",
  "_links": {
    "parent": {
      "title": "home",
      "href": "/"
    },
    "self": {
      "title": "Clustedsedit",
      "href": "clustedsEdit/5b0ec4558690a90011ce2ebf"
    },
    "collection": {
      "title": "clustedsEdit",
      "href": "clustedsEdit"
    }
  }
}
```

获取`"_id": "5b0ec4558690a90011ce2ebf"`和`"_etag": "6381e22d2cc690589fdf3870b9025ecfb7c9b905"`然后进行实际的修改

```shell
curl -g -XPATCH https://api-sic.iamhd.top/clustedsEdit/5b0ec4558690a90011ce2ebf -H Authorization:<your access token> -H If-Match:6381e22d2cc690589fdf3870b9025ecfb7c9b905 -H Content-Type:application/json -d '{"is_valid": 0}'
```


**返回：**

修改成功：

```json
{
  "_id": "5b0ec4558690a90011ce2ebf",
  "_updated": "Thu, 31 May 2018 12:19:36 GMT",
  "_created": "Wed, 30 May 2018 15:33:41 GMT",
  "_etag": "96eb9d9aa682f8298f34cb408f2d263e746c08b1",
  "_links": {
    "self": {
      "title": "Clustedsedit",
      "href": "clustedsEdit/5b0ec4558690a90011ce2ebf"
    }
  },
  "_status": "OK"
}
```

### 4.A 人群分类结果对照

人群分类结构中`"summary": "1,RAV4,6,3,3,1"`，对应`"1,RAV4,稳健彰显族,驾驭,中层,体验需求"`，分别是

1. ID
1. 车型名称
1. 人群分类编码
1. 价值观编码
1. 阶层编码
1. 汽车观编码

人群分类编码详情如下：

```
1 = "安稳务实族"
2 = "顾家进取族"
3 = "稳进内敛族"
4 = "富有精英族"
5 = "经济体面族"
6 = "稳健彰显族"
7 =  "经济享乐族"
8 = "现代乐活族"
9 = "知性内涵族"
10 = "实力任性族"
0 = "其他族"
```

价值观编码：

```
1 = "安逸"
2 = "进取"
3 = "驾驭"
4 = "引领"
5 = "刺激"
0 = "其他"
```

阶层编码：

```
1 = "基层"
2 = "中下层"
3 = "中层"
4 = "中上层"
5 = "上层"
0 = "其他"
```

汽车观编码：

```
1 = "体验需求"
2 = "彰显身份"
3 = "彰显个性"
4 = "基本需求"
0 = "其他"
```

## 5. clustedProjects API : 人群分类结果项目

该API返回人群分类结果项目，一般来说您需要分两步来使用：

 1. 先创建人群分类结果项目，服务会批量存储人群分类结果和该项目的相关信息到服务器上
 2. 读取服务器上的结果

其中创建的过程需要**消费`clusteds`资源的配额**，如果您没有足够的配额，服务器将会返回402错误。

---
Change Log:

* 0.3.0 - 改名为`clustedProjects`，并增加`append_clusteds`和`reload_clusteds`用于整理内嵌的`clusteds`文档。

---

### 5.1 POST : 创建人群分类结果项目

该API将创建人群分类结果项目，输入的数据格式为json：

```json
{
  "name": "testing",
  "project_id": 61395,
  "description": "This is testing project",
  "extras": {
    "status": "testing"
  },
  "clusteds": [
    {
      "raw": "1,RAV4,5,,6,7,14,1,7,3,5,7,5,6,5,6,6,5,5,3,4,6,6,5,2,6,4,5,6,5,6,5,5,7,7,6,2,7,6,5,5,7,5,7,5,6,6,3,6",
      ...
    },
    {
      "raw": "2,指南者,5,,4,7,16,1,2,7,6,6,5,6,5,3,5,4,2,4,4,5,5,4,4,4,5,5,6,7,3,5,7,2,5,6,4,5,5,7,3,5,4,5,7,6,6,1,6",
      ...
    },
  ]
}
```

其中：

* `"name": "test"` (可选) : 该名称
* `"project_id": 612349` (可选) : 该指定ID，仅是描述性质，API不要求唯一性，具体用法由用户自行决定
* `"description": "just for testing"` (可选) : 该项目描述
* `"allowed_users": ["vw", "sgm"]` (可选) : 该项目允许哪些用户查看，创建时服务会自动将当前用户列入
* `"extras": {"status": "testing"}` (可选，自定义) : 该项目有哪些其他属性，键和值均可自定义
* `"clusteds"` **(必选)** : 该项目有哪些输入，格式和`clusteds`服务输入格式相同，创建一个项目至少要提供2个及以上的输入

**HTTP Request:**

`POST https://api-sic.iamhd.top/clustedProjects`

**示例**

```shell
curl -XPOST https://api-sic.iamhd.top/clustedProjects  -H "Authorization:<your access token>" -H "Content-Type:application/json" -d '{"name": "testing", "project_id": 61395, "description": "This is testing project", "extras": {"status": "testing"}, "clusteds": [...]}'
```

考虑到输入内容较多，如果将输入储存在`cproject.json`文件中的话：

```shell
curl -XPOST https://api-sic.iamhd.top/clustedProjects  -H "Authorization:<your access token>" -H "Content-Type:application/json" --data-binary "@cproject.json"
```

**返回：**


```json
{
  "_id": "5b173e2ac78ef360a703b4ff",
  "_updated": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_created": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_etag": "b29cdc43e32406170eb929f3cc5174b906dd6e89",
  "_links": {
    "self": {
      "title": "Clustedproject",
      "href": "clustedProjects/5b173e2ac78ef360a703b4ff"
    }
  },
  "_status": "OK"
}
```

### 5.2 GET 列表: 读取已创建项目人群分类结果列表

该API将返回有权查看的全部`clustedProjects`

**HTTP Request:**

`GET https://api-sic.iamhd.top/clustedProjects`

**示例**

```shell
curl https://api-sic.iamhd.top/clustedProjects  -H "Authorization:<your access token>"
```

**返回：**

```json
{
  "_items": [
    {
      "_id": "5b173e2ac78ef360a703b4ff",
      "name": "testing",
      "project_id": 61395,
      "description": "This is testing project",
      "extras": {
        "status": "testing"
      },
      "clusteds": [
        "5b173e2ac78ef360a703b500",
        "5b173e2ac78ef360a703b501",
        "5b173e2ac78ef360a703b502"
      ],
      "allowed_users": [
        "dmicros"
      ],
      "_updated": "Wed, 06 Jun 2018 01:51:38 GMT",
      "_created": "Wed, 06 Jun 2018 01:51:38 GMT",
      "_etag": "b29cdc43e32406170eb929f3cc5174b906dd6e89",
      "_links": {
        "self": {
          "title": "Clustedproject",
          "href": "clustedProjects/5b173e2ac78ef360a703b4ff"
        }
      }
    }
  ],
  "_links": {
    "parent": {
      "title": "home",
      "href": "/"
    },
    "self": {
      "title": "clustedProjects",
      "href": "clustedProjects"
    }
  },
  "_meta": {
    "page": 1,
    "max_results": 25,
    "total": 1
  }
}
```

### 5.3 GET 单项详情: 读取已创建项目人群分类结果列表

该API将返回单项`clustedProjects`资源的详情，需提供`?embedded={"clusteds":1}`参数，这样返回的结果里`clusteds`字段中包括嵌入文档。

**HTTP Request:**

`GET https://api-sic.iamhd.top/clustedProjects/5b173e2ac78ef360a703b4ff?embedded={"clusteds":1}`

**示例**

```shell
curl -g 'https://api-sic.iamhd.top/clustedProjects/5b173e2ac78ef360a703b4ff?embedded={"clusteds":1}'  -H "Authorization:<your access token>"
```

Tips:由于参数中包括`{}`，可能需要在请求发出时对地址进行一定处理，如`-g`选项。

**返回：**

```json
{
  "_id": "5b173e2ac78ef360a703b4ff",
  "name": "testing",
  "project_id": 61395,
  "description": "This is testing project",
  "extras": {
    "status": "testing"
  },
  "clusteds": [
    ...
  ],
  "allowed_users": [
    "dmicros"
  ],
  "_updated": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_created": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_etag": "b29cdc43e32406170eb929f3cc5174b906dd6e89",
  "_links": {
    "parent": {
      "title": "home",
      "href": "/"
    },
    "self": {
      "title": "Clustedproject",
      "href": "clustedProjects/5b173e2ac78ef360a703b4ff"
    },
    "collection": {
      "title": "clustedProjects",
      "href": "clustedProjects"
    }
  }
}
```

### 5.4 GET 单项概要: 读取已创建项目人群分类结果列表

该API将返回单项`clustedProjects`资源概要，`clusteds`字段仅包括嵌入文档的`_id`，不包括详情。

**HTTP Request:**

`GET https://api-sic.iamhd.top/clustedProjects`

**示例**

```shell
curl https://api-sic.iamhd.top/clustedProjects/5a910541c78ef35c4730d0e5  -H "Authorization:<your access token>"
```

**返回：**

```json
{
  "_id": "5b173e2ac78ef360a703b4ff",
  "name": "testing",
  "project_id": 61395,
  "description": "This is testing project",
  "extras": {
    "status": "testing"
  },
  "clusteds": [
    "5b173e2ac78ef360a703b500",
    "5b173e2ac78ef360a703b501",
    "5b173e2ac78ef360a703b502"
  ],
  "allowed_users": [
    "dmicros"
  ],
  "_updated": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_created": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_etag": "b29cdc43e32406170eb929f3cc5174b906dd6e89",
  "_links": {
    "parent": {
      "title": "home",
      "href": "/"
    },
    "self": {
      "title": "Clustedproject",
      "href": "clustedProjects/5b173e2ac78ef360a703b4ff"
    },
    "collection": {
      "title": "clustedProjects",
      "href": "clustedProjects"
    }
  }
}
```

### 5.5 PATCH clustedProjects/reload-clusteds 重载内嵌文档

该API将根据`clusteds`的`clusted_project`字段，重载对应的`clustedProject`应该包括的内嵌文档。即API会找出`clusteds`中所有属于该`clustedProject`的文档，然后更新内嵌字段中的ID列表。

例如：我们把一个内嵌文档的`clusted_project`修改为空值，再重载，那它原来对应的`clustedProjects`就不会再继续包含它了。

**HTTP Request:**

`PATCH https://api-sic.iamhd.top/clustedProjects/reload-clusteds`

**示例**

我们要修改的内嵌文档`clustedProjects/5b173e2ac78ef360a703b4ff`，它包括`clusteds/5b173e2ac78ef360a703b501`。首先查看一下项目概要。

```shell
curl 'https://api-sic.iamhd.top/clustedProjects/5b173e2ac78ef360a703b4ff' -H "Authorization:<your access token>"
```

**返回：**

```json
{
  "_id": "5b173e2ac78ef360a703b4ff",
  "name": "testing",
  "project_id": 61395,
  "description": "This is testing project",
  "extras": {
    "status": "testing"
  },
  "clusteds": [
    "5b173e2ac78ef360a703b500",
    "5b173e2ac78ef360a703b501",
    "5b173e2ac78ef360a703b502"
  ],
  "allowed_users": [
    "dmicros"
  ],
  "_updated": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_created": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_etag": "b29cdc43e32406170eb929f3cc5174b906dd6e89",
  "_links": {
    "parent": {
      "title": "home",
      "href": "/"
    },
    "self": {
      "title": "Clustedproject",
      "href": "clustedProjects/5b173e2ac78ef360a703b4ff"
    },
    "collection": {
      "title": "clustedProjects",
      "href": "clustedProjects"
    }
  }
}
```

然后，将`clustedProjects/5b173e2ac78ef360a703b4ff`的一个内嵌文档的`clusted_project`修改为空。

```shell
curl -XPATCH 'https://api-sic.iamhd.top/clusteds/edit/5b173e2ac78ef360a703b501' -H "Authorization:<your access token>" -H "If-Match:c6f710e0de2373e936e82cb4222feb929fc2bfa6" -H "Content-Type:application/json" -d '{"clusted_project":null}'
```

**返回：**

```json
{
  "_id": "5b173e2ac78ef360a703b501",
  "_updated": "Wed, 06 Jun 2018 02:26:54 GMT",
  "_created": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_etag": "011e9e39ac1ccb1d14fc718c2e7bcad4ad5ff2b9",
  "_links": {
    "self": {
      "title": "Clustedsedit",
      "href": "clusteds/edit/5b173e2ac78ef360a703b501"
    }
  },
  "_status": "OK"
}
```

接着，再调用`clustedProjects/reload_clusteds/5b173e2ac78ef360a703b4ff`。

*注意：虽然该API不能修改其他字段，也需要传递一个空数据。*

```shell
curl -XPATCH 'https://api-sic.iamhd.top/clustedProjects/reload-clusteds/5b173e2ac78ef360a703b4ff' -H "Authorization:<your access token>" -H "If-Match:b29cdc43e32406170eb929f3cc5174b906dd6e89" -H "Content-Type:application/json" -d '{}'
```

**返回：**

```json
{
  "_id": "5b173e2ac78ef360a703b4ff",
  "_updated": "Wed, 06 Jun 2018 02:39:14 GMT",
  "_created": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_etag": "b6385003e0c4143b4a3533c12086e83f64cbdf1f",
  "_links": {
    "self": {
      "title": "Clustedprojectsreload",
      "href": "clustedProjects/reload-clusteds/5b173e2ac78ef360a703b4ff"
    }
  },
  "_status": "OK"
}
```

再查看`clustedProjects/5b173e2ac78ef360a703b4ff`，发现已不再包括`clusteds/5b173e2ac78ef360a703b501`。

```shell
curl 'https://api-sic.iamhd.top/clustedProjects/5b173e2ac78ef360a703b4ff' -H "Authorization:<your access token>"
```

**返回：**

```json
{
  "_id": "5b173e2ac78ef360a703b4ff",
  "name": "testing",
  "project_id": 61395,
  "description": "This is testing project",
  "extras": {
    "status": "testing"
  },
  "clusteds": [
    "5b173e2ac78ef360a703b500",
    "5b173e2ac78ef360a703b502"
  ],
  "allowed_users": [
    "dmicros"
  ],
  "_updated": "Wed, 06 Jun 2018 02:39:14 GMT",
  "_created": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_etag": "b6385003e0c4143b4a3533c12086e83f64cbdf1f",
  "_links": {
    "parent": {
      "title": "home",
      "href": "/"
    },
    "self": {
      "title": "Clustedproject",
      "href": "clustedProjects/5b173e2ac78ef360a703b4ff"
    },
    "collection": {
      "title": "clustedProjects",
      "href": "clustedProjects"
    }
  }
}
```

### 5.6 PATCH clustedProjects/append-clusteds 新增内嵌文档

该API将新增`clustedProjects`包括的内嵌文档。

*注意：这里的新增是指新加入数据库的`clusteds`。如果想要内嵌已经在数据库里的`clusteds`，请使用`clustedProjects/reload-clusteds`。*

**HTTP Request:**

`PATCH https://api-sic.iamhd.top/clustedProjects/append-clusteds`

**示例**

`PATCH`的参数是`{"clusteds": [<新的cluteds输入>, ...]}`

```shell
curl -XPATCH 'https://api-sic.iamhd.top/clustedProjects/append-clusteds/5b173e2ac78ef360a703b4ff' -H "Authorization:<your access token>" -H "If-Match:b6385003e0c4143b4a3533c12086e83f64cbdf1f" -H "Content-Type:application/json" -d '{"clusteds":[...]}'
```

**返回：**

```json
{
  "_id": "5b173e2ac78ef360a703b4ff",
  "_updated": "Wed, 06 Jun 2018 03:13:55 GMT",
  "_created": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_etag": "ca0232b6da555fa234ba87c61484d5fa57211a24",
  "_links": {
    "self": {
      "title": "Clustedprojectsappend",
      "href": "clustedProjects/append-clusteds/5b173e2ac78ef360a703b4ff"
    }
  },
  "_status": "OK"
}
```

查看新增后的`clustedProjects`：

```shell
curl -g 'https://api-sic.iamhd.top/clustedProjects/5b173e2ac78ef360a703b4ff?embedded={"clusteds":1}' -H "Authorization:<your access token>"
```

```json
{
  "_id": "5b173e2ac78ef360a703b4ff",
  "name": "testing",
  "project_id": 61395,
  "description": "This is testing project",
  "extras": {
    "status": "testing"
  },
  "clusteds": [
   ...
  ],
  "allowed_users": [
    "dmicros"
  ],
  "_updated": "Wed, 06 Jun 2018 03:13:55 GMT",
  "_created": "Wed, 06 Jun 2018 01:51:38 GMT",
  "_etag": "ca0232b6da555fa234ba87c61484d5fa57211a24",
  "_links": {
    "parent": {
      "title": "home",
      "href": "/"
    },
    "self": {
      "title": "Clustedproject",
      "href": "clustedProjects/5b173e2ac78ef360a703b4ff"
    },
    "collection": {
      "title": "clustedProjects",
      "href": "clustedProjects"
    }
  }
}
```

## API限制

API数据传输限制是16M：

* 请求大小的硬性限制为16M，是mongodb文档格式bson的大小限制
* 如果出现数据小于16M但不能正常传输的情况，应是BUG，需要联系管理员排查解决

## 文档生成

文档源码可以使用`mkdocs`生成静态网页。

下载镜像（包括了必要的依赖）：

```shell
docker pull dhuan/mkdocs_plantuml:material
```

在`mkdocs.yaml`所在的文件夹运行镜像，即可通过http:\\127.0.0.1:8000 查看编译好的文档

```shell
docker run -it --rm --name docs -v `pwd`:/docs -p 8000:8000 dhuan/mkdocs_plantuml:material
```
