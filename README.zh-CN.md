# uniwebpay Skill

**语言：** [English](README.md) | 中文

uniweb 支付平台的 AI 集成助手。你不用自己读文档，让 coding agent 帮你写集成代码。

## Quick Start

四步，从零到收款。全程用大白话对 Claude 说需求，skill 自动帮你执行。

### 1. 安装 Skill

```bash
npx skills add uniwebpay/skills
```

把 `uniweb` skill 装进 Claude Code。装完 Claude 就能识别“支付 / 收款 / 结账”这类需求并自动调用它。

> 想装到全局所有项目：`npx skills add uniwebpay/skills -g`

### 2. 创建账户

对 Claude 说：

> 帮我创建一个 uniweb 收款账户

Claude 自动执行 `uniweb wallet create`，生成钱包并把密钥安全存到本地（`~/.uniweb/config.json`）：

```text
✓ Wallet created
Wallet ID:  wal_xxxxxxxxxxxx
Status:     active
Currency:   SGD
```

> 只需做一次，之后所有操作自动复用这个账户。

### 3. 生成支付链接

对 Claude 说：

> 生成一个收 10 块钱的支付链接

Claude 自动执行 `uniweb link create 1000 -c SGD`，返回一个永久可付款的链接：

```text
✓ Payment link created
URL:    https://skill.uniwebpay.com/p/plink_xxxxxxxxxxxx
Amount: 10.00 SGD
```

把它贴到网页、邮件或二维码里即可开始收款，**无需任何后端代码**：

```html
<a href="https://skill.uniwebpay.com/p/plink_xxxxxxxxxxxx">立即支付 S$10</a>
```

### 4. 认领后台

对 Claude 说：

> 给我的账户绑定 dashboard

Claude 自动执行 `uniweb wallet claim`，返回一个认领链接：

```text
Claim URL:  https://skill.uniwebpay.com/claim/wal_xxxx?token=claim_xxxx
Expires At: 2026-07-10 14:23
```

在浏览器打开该链接、登录账号，即可把命令行创建的钱包绑定到网页后台，之后可查看余额、订单、支付链接、Webhook 与提现等。

> 认领链接有有效期，过期后重新运行 `uniweb wallet claim` 即可生成新链接。

### 小结

| 步骤 | 你说的话 | Skill 执行的命令 |
| --- | --- | --- |
| 1. 安装 | - | `npx skills add uniwebpay/skills` |
| 2. 创建账户 | “创建 uniweb 收款账户” | `uniweb wallet create` |
| 3. 生成收款 | “生成收 10 块的支付链接” | `uniweb link create 1000` |
| 4. 认领后台 | “绑定 dashboard” | `uniweb wallet claim` |

**装一次 skill，Claude 就永久学会了接支付。** 之后你只需用人话说需求。

## Install (Codex)

```bash
git clone https://github.com/uniwebpay/skills.git uniwebpay-skills
mkdir -p ~/.codex/skills
cp -R uniwebpay-skills/skills/uniweb ~/.codex/skills/uniweb
```

然后显式调用这个 skill：

```text
Use $uniweb to add a checkout button to my Express app.
```

## 能做什么

- 为 Express、Next.js 和其他 Node.js 框架生成 checkout session 集成代码
- 设置带签名校验的 webhook handler
- 配置带试用期和催收逻辑的订阅计费
- 创建无需代码的支付链接
- 辅助使用 CLI 命令和 REST API
- 排查支付问题

## Links

- [uniweb Docs](https://skill.uniwebpay.com/docs)
- [API Reference](https://skill.uniwebpay.com/docs/api-reference)
- [Dashboard](https://skill.uniwebpay.com/dashboard)
