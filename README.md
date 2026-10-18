# OperitForge
Operit publish-only artifact repository for release assets.
# DeepSeek 余额查询 Pro

> 在 Operit 里用一句话查到 DeepSeek 开放平台账户余额。
> Query your DeepSeek platform balance with a single sentence inside Operit.

[![version](https://img.shields.io/badge/version-1.1.1-blue)]() [![license](https://img.shields.io/badge/license-MIT-green)]()

---

## 这是什么

一个 Operit 工具包，提供一个工具 `deepseek_balance_pro:get_balance`，调用 DeepSeek 开放平台的
`GET https://api.deepseek.com/user/balance` 接口，把返回的余额整理成人能直接读的话。

装好之后，对 AI 说「查一下 DeepSeek 余额」，它就当场查给你看。

---

## 披露信息（Disclosure）

### 作者

| 项 | 值 |
|---|---|
| 作者 | DeepSeek娘、猫猫增强 |
| 包 ID | `com.deepseek.whale.balance.pro` |
| 当前版本 | 1.1.1 |
| 许可证 | MIT |

本包基于社区已有的 DeepSeek 余额查询工具思路做了增强实现，包 ID 与原版
`com.deepseek.whale.balance` **完全独立**，两者可以同时安装、互不干扰。

### 环境变量（Environment Variables）

本包**必须**配置 1 个环境变量，已在 `manifest.json` 的 `subpackages[0].env` 中声明：

| 变量名 | 必填 | 说明 |
|---|---|---|
| `DEEPSEEK_API_KEY` | **是** | DeepSeek 开放平台 API Key，以 `sk-` 开头。在 <https://platform.deepseek.com> → API keys 创建 |

- 导入时 Operit 会弹出输入框，直接粘贴即可。
- 也可在 Operit 的「管理环境变量」中手动添加，变量名必须一字不差为 `DEEPSEEK_API_KEY`。
- 本包**不会**读取、上传、记录或缓存这个 Key 之外的信息；Key 仅用于拼装请求头
  `Authorization: Bearer <key>`，不写入任何文件、不写入日志。

> ⚠️ 本仓库**不含任何 API Key**。请勿在 Issue / PR 中粘贴你的 Key。

### 网络行为（Network Behavior）

本包**只访问一个域名**：

| 目标 | 用途 | 方法 |
|---|---|---|
| `https://api.deepseek.com/user/balance` | 查询账户余额 | GET |

- 不访问任何第三方域名、不上报使用数据、不做埋点、无遥测。
- 不包含任何外链广告、跳转或推广。
- 除上述接口外，无其他网络请求。

### 权限（Permissions）

| 能力 | 是否使用 | 说明 |
|---|---|---|
| 网络请求 | 是 | 仅调用上面列出的 DeepSeek 余额接口 |
| 读取环境变量 | 是 | 仅读取 `DEEPSEEK_API_KEY` |
| 文件读写 | 否 | 不读写任何用户文件 |
| 系统命令 / Shell | 否 | 不执行任何系统命令 |
| 其他包的工具 | 否 | 无跨包 `require`，零依赖 |

### 外部服务（External Services）

依赖 DeepSeek 开放平台（<https://platform.deepseek.com>）的官方余额接口。
使用前需自行注册 DeepSeek 账号，本包与 DeepSeek 官方无隶属关系，非官方出品。

### 已知风险（Known Risks）

1. **接口变更风险**：若 DeepSeek 官方调整 `/user/balance` 的返回结构，解析可能失效。
   可传 `raw: true` 拿到未经处理的原始 JSON 用于排查。
2. **Key 保管责任在用户**：Key 存放于 Operit 的环境变量中（Android 应用私有目录）。
   Key 泄露造成的损失由用户自行承担，建议定期在平台轮换。
3. **余额延迟**：余额以接口返回为准，存在极短的账务同步延迟，不保证与平台网页端逐分一致。
4. **不做扣费保证**：本包只读取余额，不参与任何计费、充值或扣款行为。

---

## 安装

1. 下载 `com-deepseek-whale-balance-pro-v1.1.1.toolpkg`
2. Operit → `Manage Packages`（管理包）→ `Import`（导入）→ **从本地文件导入**
3. 选中该文件 → 确认
4. 弹框里填入你的 `DEEPSEEK_API_KEY` → 保存
5. 对 AI 说「查一下 DeepSeek 余额」验证

---

## 用法

### 一句话用法

```
查一下 DeepSeek 余额
我 DeepSeek 还剩多少钱
余额低于 10 块就提醒我
```

### 返回示例

```
✅ DeepSeek 账户可用

【CNY】总余额 ¥6.50
 赠送余额：¥0.00
 充值余额：¥6.50
```

### 参数

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `raw` | boolean | `false` | `true` 返回 API 原始 JSON 字符串，排查接口变更时用 |
| `low_balance_threshold` | number | 不填 | 低余额阈值，**严格小于**才告警 |
| `timeout_ms` | number | `15000` | 单次请求超时（毫秒） |
| `retry` | number | `1` | 失败重试次数，`0` 表示不重试 |
| `format` | string | `text` | `text` 可读文本 / `json` 结构化数据 |

### 返回结构

```jsonc
{
  "success": true,
  "message": "...",
  "data": {
    "is_available": true,
    "api_reported_available": true,
    "has_positive_balance": true,
    "balances": [ { "currency": "CNY", "total_balance": "6.50", "granted_balance": "0.00", "topped_up_balance": "6.50" } ],
    "low_balance_warning": null,
    "message": "...",
    "output_format": "text"
  }
}
```

`raw: true` 时**只返回两个字段**，没有 `success` / `message` / `data` 外壳：

```jsonc
{ "raw": "<API 原始响应文本>", "http_status": 200 }
```

这是刻意设计的：原始模式下不做任何业务语义包装，避免与正常返回的 `success` 混淆。

### 三态可用性判定

这是 Pro 版相对原版的核心区别：

```
api_reported_available = API 返回 is_available 为真？
has_positive_balance   = 总余额 > 0？
is_available           = 两者同时成立
```

原版只看 API 的 `is_available`，即使账户余额已经归零也可能报「可用」；
Pro 版额外校验实际余额，两条件都满足才判定为可用。

---

## 兼容性

| 项 | 要求 |
|---|---|
| 平台 | Operit（Android） |
| 包 API 版本 | `api_version 1.0.0` |
| 依赖 | 无，零外部依赖 |
| 与原版共存 | 可以，包 ID 不同 |

---

## 许可证

MIT License，全文见仓库根目录的 [`LICENSE`](./LICENSE)。

可自由使用、修改、分发，保留版权声明即可。

## 反馈

有问题请在本仓库提 Issue。提交前请确认：

- 已确认 `DEEPSEEK_API_KEY` 拼写正确且已保存
- Issue 内容中**不含**你的 API Key（截图请打码）

---

*本 README 面向包市场发布。日常使用的中文图文版说明见仓库内 `README.md`。*
