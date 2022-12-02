---
title: "XCPC Team Registration 的技术文档"
categories: 'DevOps'
toc: true
date: 2021-04-03 16:31:25
tags:
	- 开发
	- Golang
description: ' '
---

## 项目地址

[这里](https://github.com/HeRaNO/xcpc-team-reg)

## 响应结构说明

``` Go
type Respond struct {
	Code  int             `json:"code"`
	Msg   string          `json:"msg"`
	Data  []interface{}{} `json:"data"`
	Total *int            `json:"total",omitempty`
}
```

其中：

- `code`：请求状态码，取值如下：
  - `0`：请求成功
  - `1000`：内部错误
  - `1001`：权限不足
  - `1002`：请求中信息有误
  - `1003`：不在报名时间段内
- `msg`：响应信息，若请求状态码为 `0`，则为 `success`，否则为失败原因。
- `data`：如果响应成功，则包含响应数据段，否则无此字段。
- `total`（可选）：包含响应数据段中数据组数，只有响应成功的情况下可能包含。

以下响应数据结构中只说明

## 接口与接口定义

### `/api/`

#### 方法

`GET`

#### 作用

没有实际作用，可以用作 Ping。

#### 请求数据结构

请求中不需附加任何数据。

#### 响应数据结构

返回一个常量字符串 `xcpc-team-reg`。

#### 权限

任意用户。

### `/api/getIDSchool`

#### 方法

`GET`

#### 作用

获得以 ID 为 key 的学院列表。

#### 请求数据结构

请求中不需附加任何数据。

#### 响应数据结构

返回一个 JSON 对象，格式为：

``` json
{
	"<id_1>": "<school_name_1>",
	"<id_2>": "<school_name_2>"
}
```

其中 `<id>` 为学院 ID，`<school_name>` 为学院名称，ID 与名称一一对应，并且在服务中 ID 是学院的唯一标识。

响应数据中会包含学院总数，为 `total` 字段中的值。

#### 权限

任意用户。

### `/api/getSchoolID`

#### 方法

`GET`

#### 作用

获得以学院为 key 的 ID 列表。

#### 请求数据结构

请求中不需附加任何数据。

#### 响应数据结构

返回一个 JSON 对象，格式为：

``` json
{
	"<school_name_1>": "<id_1>",
	"<school_name_2>": "<id_2>"
}
```

其中 `<id>` 为学院 ID，`<school_name>` 为学院名称，ID 与名称一一对应，并且在服务中 ID 是学院的唯一标识。

响应数据中会包含学院总数，为 `total` 字段中的值。

#### 权限

任意用户。

### `/api/getContestInfo`

#### 方法

`GET`

#### 作用

获得在报名的比赛详细信息。

#### 请求数据结构

请求中不需附加任何数据。

#### 响应数据结构

``` go
type ContestInfo struct {
	Name      string `json:"name"`
	StartTime string `json:"start_time"`
	EndTime   string `json:"end_time"`
	Note      string `json:"note"`
}
```

其中：

- `name`：比赛名称
- `start_time`：报名开始时间
- `end_time`：报名结束时间
- `note`：注意事项

#### 权限

任意用户。

### `/api/verifyEmail`

#### 方法

`POST`

#### 作用

验证用户邮箱。

#### 请求数据结构

``` go
type EmailVerification struct {
	Email string `json:"email"`
	Type  string `json:"type"`
}
```

其中：

- `email`：要验证的邮箱
- `type`：当前操作类型

#### 响应数据结构

返回一个常量字符串 `ok`。

#### 权限

任意用户。

### `/api/register`

#### 方法

`POST`

#### 作用

用户注册。

#### 请求数据结构

``` go
type UserRegister struct {
	Name       string `json:"name"`
	Email      string `json:"email"`
	School     int    `json:"school"`
	StuID      string `json:"stuid"`
	EmailToken string `json:"email_token"`
	PwdToken   string `json:"pwd_token"`
	Action     string `json:"action"`
}
```

其中：

- `name`：用户真实姓名
- `email`：用户邮箱
- `school`：用户学院 ID
- `stuid`：用户学号
- `email_token`：邮箱验证码，用户需要先请求一个邮箱验证码
- `pwd_token`：密码加密后的值
- `action`：当前在进行的操作，应为一常量字符串 `register`

#### 响应数据结构

返回一个常量字符串 `ok`。

#### 权限

任意用户。

### `/api/login`

#### 方法

`POST`

#### 作用

用户登录。

#### 请求数据结构

``` go
type UserLogin struct {
	Email    string `json:"email"`
	PwdToken string `json:"pwd"`
}
```

其中：

- `email`：用户邮箱
- `pwd`：密码加密后的值

#### 响应数据结构

返回一个字符串，为用户的 JWT。以后任何操作都应将 JWT 附属在请求头的 Authorization 字段中，如下形式：

``` plain
Authorization: Bearer <JWT>
```

其中 `<JWT>` 为用户的 JWT。

#### 权限

任意未登录的用户。

### `/api/forgot`

#### 方法

`POST`

#### 作用

用户重置密码。

#### 请求数据结构

``` go
type UserResetPwd struct {
	Email      string `json:"email"`
	EmailToken string `json:"email_token"`
	PwdToken   string `json:"pwd_token"`
	Action     string `json:"action"`
}
```

其中：

- `email`：用户邮箱
- `email_token`：邮箱验证码，用户需要先请求一个邮箱验证码
- `pwd_token`：密码加密后的值
- `action`：当前在进行的操作，应为一常量字符串 `reset`

#### 响应数据结构

返回一个常量字符串 `ok`。

#### 权限

任意未登录的用户。

### `/api/logout`

#### 方法

`POST`

#### 作用

用户登出。

#### 请求数据结构

请求中不需附加任何数据。

#### 响应数据结构

返回一个常量字符串 `ok`。

#### 权限

任意登录的用户。

### `/api/getUserInfo`

#### 方法

`GET`

#### 作用

查看用户信息。

#### 请求数据结构

请求中不需附加任何数据。

#### 响应数据结构

``` go
type UserInfo struct {
	Name       string `json:"name"`
	School     string `json:"school"`
	StuID      string `json:"stuid"`
	BelongTeam int64  `json:"teamid"`
}
```

其中：

- `name`：用户真实姓名
- `school`：用户所属学院
- `stuid`：用户学号
- `teamid`：用户所属队伍编号，`0` 为未加入任何队伍

#### 权限

任意登录的用户。

### `/api/getTeamInfo`

#### 方法

`GET`

#### 作用

查看用户所属队伍信息。

#### 请求数据结构

请求中不需附加任何数据。

#### 响应数据结构

``` go
type TeamInfo struct {
	TeamID       int64      `json:"team_id"`
	TeamName     string     `json:"team_name"`
	TeamAccount  string     `json:"account"`
	TeamPassword string     `json:"password"`
	InviteToken  string     `json:"invite_token"`
	TeamMember   []UserInfo `json:"member"`
	MemberCnt    int        `json:"mem_cnt"`
}
```

其中：

- `team_id`：队伍 ID
- `team_name`：队伍名称
- `account`：队伍账号
- `password`：队伍密码
- `invite_token`：队伍邀请码
- `member`：队伍中所有队员的信息
- `mem_cnt`：队伍中队员总数

#### 权限

任意登录的用户。

### `/api/modifyUserInfo`

#### 方法

`POST`

#### 作用

用户修改个人信息。

#### 请求数据结构

``` go
type UserInfoModify struct {
	Name   string `gorm:"column:user_name" json:"name"`
	School int    `gorm:"column:school" json:"school"`
	StuID  string `gorm:"column:stu_id" json:"stuid"`
}
```

其中：

- `name`：用户真实姓名
- `school`：用户学院
- `stuid`：用户学号

#### 响应数据结构

返回一个常量字符串 `ok`。

#### 权限

任意登录的用户。

### `/api/modifyTeamInfo`

#### 方法

`POST`

#### 作用

用户修改所属队伍信息。

#### 请求数据结构

``` go
type TeamInfoModify struct {
	TeamName string `json:"team_name"`
}
```

其中：

- `team_name`：队伍名称

#### 响应数据结构

返回一个常量字符串 `ok`。

#### 权限

任意登录的用户。

### `/api/createTeam`

#### 方法

`POST`

#### 作用

用户新建一个队伍。

#### 请求数据结构

``` go
type TeamInfoModify struct {
	TeamName string `json:"team_name"`
}
```

其中：

- `team_name`：队伍名称

#### 响应数据结构

返回一个常量字符串 `ok`。

#### 权限

任意登录的用户。

### `/api/joinTeam`

#### 方法

`POST`

#### 作用

用户加入某个队伍。

#### 请求数据结构

``` go
type JoinTeamRequest struct {
	TeamID      int64  `json:"team_id"`
	InviteToken string `json:"invite_token"`
}
```

其中：

- `team_id`：要加入的队伍 ID
- `invite_token`：加入队伍的邀请码

#### 响应数据结构

返回一个常量字符串 `ok`。

#### 权限

任意登录的用户。

### `/api/quitTeam`

#### 方法

`POST`

#### 作用

用户退出自己所属的队伍。

#### 请求数据结构

请求中不需附加任何数据。

#### 响应数据结构

返回一个常量字符串 `ok`。

#### 权限

任意登录的用户。

### `/api/admin/adminHello`

#### 方法

`GET`

#### 作用

没有实际作用，可以用作管理员状态检查。

#### 请求数据结构

请求中不需附加任何数据。

#### 响应数据结构

返回一个常量字符串 `xcpc-team-reg admin`。

#### 权限

登录的管理员用户。

### `/api/admin/export`

#### 方法

`GET`

#### 作用

按 CSV 格式导出所有队伍信息。

#### 请求数据结构

请求中不需附加任何数据。

#### 响应数据结构

返回一个文件，格式为：

- 第一列：队伍 ID
- 第二列：队伍名称
- 接下来的每三列：队员姓名、队员学院，队员学号

#### 权限

登录的管理员用户。

### `/api/admin/import`

#### 方法

`POST`

#### 作用

导入队伍账号密码。

#### 请求数据结构

请求中添加一个文件，字段名为 `team_table`。

#### 响应数据结构

返回一个 JSON 数组，包含所有未成功添加的队伍 ID。`total` 字段为所有未成功添加的队伍总数。

#### 权限

登录的管理员用户。

## 逻辑

### 用户服务逻辑

用户输入正确的邮箱和密码后即可登录成功。

用户只能在非登录状态下登录和重置密码。

用户只能在登录状态下登出。

用户发送注册或找回密码请求前应先请求验证邮箱。

### 队伍管理逻辑

只有未加入队伍的用户可以创建或加入队伍。

用户加入队伍时需要提供队伍 ID 和队伍邀请码。

只有已加入队伍的用户可以退出队伍。

## 可改进点

- 队伍审核后自动生成账号密码（以后再说吧）。
- ~~组队时间控制，在时间之后无法对队伍进行操作。~~
- ~~多场比赛报名，作为 SaaS 提供服务（作为一次性服务运行，不提供了）。~~
