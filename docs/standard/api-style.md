# 前后端接口规范 - RESTful 版

本规范的三个目标：**简洁**、**统一**、**开放**。

关于如何设计良好风格的 RESTful API，Github 有一份[满分答案](https://docs.github.com/en/rest/reference/actions)，熟读三遍，其义自现。本规范将在其基础之上使用尽可能简单的表述方式从以下几个常见部分作出详细约定：

- [基础约定](#1-基础约定)

# 规范内容

## 1. 基础约定

#### 1.1 (必要)接口路径以 `/api` 或 `/[version]/api` 开头

正确：`/api/task` 或 `/v2/api/tasks`

错误：`/biz/tasks` 或 `/biz/api/tasks`

注意：一个产品无论后端有多少个服务组成也应该只有一个 API 入口

#### 1.2 (必要)接口路径以驼峰式 `api/taskGroups` 方式命名

正确：`/api/taskGroups`

错误：`/api/task-groups`

#### 1.3 (推荐)接口路径使用资源名词而非动词，动作应由 HTTP Method 体现（在 Method 受限（只能使用 get/post）的场合除外），资源组可以进行逻辑嵌套

正确：POST `/api/tasks` 或 `/api/taskGroups/1/tasks` 表示在 id 为 1 的任务组下创建任务

错误：POST `/api/create-task`

#### 1.4 (推荐)接口路径中的资源使用复数而非单数

正确：`/api/tasks`

错误：`/api/task`

#### 1.5 (推荐)接口设计面向开放接口，而非单纯前端业务

要求我们在给接口路径命名时要面向通用业务而非单纯前端业务，以获取筛选表单中的任务字段下拉选项为例

正确：`/api/tasks`

错误：`/api/taskSelectOptions`

虽然这个接口暂时只用在表单的下拉选择中，但是需要考虑的是在未来可能会被用在任意场景，因此应以更通用方式提供接口交由客户端自由组合

#### 1.6 (推荐)规范使用 HTTP 方法（可以只使用 get/post 请求，接口路径使用动词）

| 方法         | 场景         | 例如                                                     |
| ------------ | ------------ | -------------------------------------------------------- |
| GET（推荐）  | 获取数据     | 获取单个：GET `/api/tasks/1`、获取列表：GET `/api/tasks` |
| POST（推荐） | 创建数据     | 创建单个：POST `/api/tasks`                              |
| PATCH        | 差量修改数据 | 修改单个：PATCH `/api/tasks/1`                           |
| PUT          | 全量修改数据 | 修改单个：PUT `/api/tasks/1`                             |
| DELETE       | 删除数据     | 删除单个：DELETE `/api/tasks/1`                          |

#### 1.7 (必要)规范使用 HTTP 状态码

| 状态码 | 场景                                                                 |
| ------ | -------------------------------------------------------------------- |
| 200    | 创建成功，通常用在同步操作时                                         |
| 202    | 创建成功，通常用在异步操作时，表示请求已接受，但是还没有处理完成     |
| 400    | 参数错误，通常用在表单参数错误                                       |
| 401    | 授权错误，通常用在 Token 缺失或失效，注意 401 会触发前端跳转到登录页 |
| 403    | 操作被拒绝，通常发生在权限不足时，注意此时务必带上详细错误信息       |
| 404    | 没有找到对象，通常发生在使用错误的 id 查询详情                       |
| 500    | 服务器错误                                                           |

其它更多响应状态码请查阅 [MDN Web Docs](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status)

#### 1.8 (必要)基础外层数据结构

- 不分页数据

  ```javascript
  {
      code: 200,
      status: 200,
      message: "请求成功",
      data: {
          id: 1,
          name: '任务 1'
      }
  }
  ```

- 分页数据

  ```javascript
  {
      code: 200,
      status: 200,
      message: "请求成功",
      // 内容自定义
      data: {
          items: [{
              id: 1,
              name: '任务 1'
          }, {
              id: 2,
              name: '任务 2'
          }],
          total: 2,
          pageSize:1,
          pageNumber:2,
      }
  }
  ```

注意：其中 `code` 表示业务编码，`status` 表示 HTTP 响应状态码，如此设计的原因是部分场景下前后端之间存在不可控的网关或代理（比如某些网关有一些流量控制策略会导致直接返回 403 响应状态码，此时客户端无法分辨 403 是网关的还是业务方），类似这类情况下为了能够让客户端正确分辨业务方的真实处理结果则需要在响应体加上 `status`，而 `code` 表示的业务编码是为了帮助工程师更容易定位问题，它并不是必须的，这取决你们团队风格。

> 尽管在响应体中体现了响应状态码，但这并不代表所有 HTTP 就可以全部返回 200 了，无论如何请在条件允许范围内尽可能使用标准的 HTTP 响应状态码

#### 1.9 (必要)code 码(业务编码)定义样例

内置 `code` 码取值区间为 0-999，1000(含)以上为自定义`code`码。

```
    // 返回成功 为 0 其他为异常情况
    int SUCCESS = 0;

    // 自定义code码举例

    //请求参数错误
    int ERR_INVALID_REQ_PARAM = 110000;

    //请求方式不正确
    int ERR_METHOD_NOT_SUPPORTED = 110001;

    //账户已绑定
    int ERR_ACCOUNT_BIND = 110100;

    //密码验证失败
    int ERR_PASSWORD_CHECK_FAILED = 110101;

    //账户不存在
    int ERR_ACCOUNT_NOT_EXIST = 110102;

    //账户未绑定
    int ERR_ACCOUNT_NOT_BIND = 110103;

    //账户未实名认证
    int ERR_ACCOUNT_NOT_AUTHORIZED = 110104;

    //权限不足
    int ERR_LIMITED_AUTHORITY = 110105;

    //验证码错误
    int ERR_CATPCHA = 110106;

    //token过期
    int AUTH_TOKEN_ERROR = 110107;

    //验证码错误
    int ERR_PASSWORD = 110108;

    //认证错误
    int AUTH_ERR = 110109;

    //新增错误
    int INSERT_ERR = 110110;

    //更新错误
    int UPDATE_ERR = 110110;

```

#### 1.10 (必要)请求和响应字段采用 `aaBbCc` 驼峰方式命名

```javascript
// 正确
{
   roleIds: [11, 12, 35],
}
// 错误
{
  role_ids: [11,12,35],
  RoleIds: [11, 12, 35],
  ROLE_IDS: [11, 12, 35]
}
```

#### 1.11 (推荐)常见业务字段约定

唯一标识：id

名称：name

创建时间：createdAt

更新时间：updatedAt

#### 1.12 (必要)空数组使用 []，而不是 null

```javascript
// 正确
{
    code: 200,
    status: 200,
    message: "请求成功",
    data: {
        id: 1,
        roleIds: [],
    }
}
// 错误
{
    code: 200,
    status: 200,
    message: "请求成功",
    data: {
        id: 1,
        roleIds: null
    }
}
```

#### 1.13 (必要)前后端传输过程以标准 JSON 格式，避免反复正反序列化

```javascript
// 正确
{
    code: 200,
    status: 200,
    message: "请求成功",
    data: {
        roles: [{
            id: 1,
            name: '角色 1'
        }, {
            id: 2,
            name: '角色 2'
        }]
    }
}
// 错误
{
    code: 200,
    status: 200,
    message: "请求成功",
    data: {
        roles: '[{"id":1,"name":"角色 1"},{"id":2,"name":"角色 2"}]'
    }
}
```
