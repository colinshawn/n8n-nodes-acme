# n8n-nodes-acme

[English](README.md) | [简体中文](README.zh-CN.md)

用于在 n8n 工作流中通过 ACME 申请 TLS 证书、生成和解析证书材料，以及查询 DNS TXT 记录的社区节点插件。

本项目提供三类节点，可以组合成证书签发工作流。ACME 节点提供 Let's Encrypt 测试环境（Staging）和生产环境（Production），支持 DNS-01 和 HTTP-01 域名验证方式，默认选择测试环境。

## 支持的操作

| 节点        | 操作                                                                     |
| ----------- | ------------------------------------------------------------------------ |
| ACME        | 创建账户、创建订单、查询订单、获取验证参数、完成验证、提交签发、下载证书 |
| Certificate | 生成私钥、生成 CSR（证书签名请求）、解析 CSR、解析证书                   |
| DNS         | 查询 DNS TXT 记录、等待 DNS TXT 记录生效                                 |

DNS 节点负责查询记录和等待解析生效。创建和删除 DNS 记录、提供 HTTP 验证响应、定时续期和部署证书，需要在工作流中连接其他节点或服务。

## 安装

包名为 `n8n-nodes-acme`。在 n8n 实例中安装时，请参考 [n8n 社区节点安装指南](https://docs.n8n.io/integrations/community-nodes/installation/)。

如果需要从本仓库进行本地开发，请参阅[本地开发](#本地开发)。

## 账户密钥和数据

本项目目前没有自定义的 n8n 凭证类型。ACME 操作通过 **Account Key** 参数接收 PEM 格式的账户私钥文本。同一账户和目录端点的后续操作应复用相同的账户密钥。

账户密钥和证书私钥的用途不同：账户密钥用于标识 ACME 账户；证书私钥用于生成 CSR，签发后需要与证书一起保留并用于部署。

- **Generate Private Key（生成私钥）** 和 **Generate CSR（生成 CSR）** 将结果输出到指定的二进制附件字段。传入要求字符串的参数前，需要先将附件转换为 PEM 文本。
- 订单、验证参数、解析结果、DNS 查询结果，以及下载的证书 PEM 文本，会写入指定的 JSON 输出字段。
- **Complete Challenge（完成验证）**、**Finalize Order（提交签发）** 和 **Wait for DNS Text Record（等待 DNS TXT 记录生效）** 在操作成功后将输入数据继续传递给后续节点。

请勿将真实账户密钥、证书私钥或包含敏感信息的工作流导出文件提交到版本库。仓库的 `.gitignore` 已覆盖常见的密钥和证书文件扩展名，以及 n8n 本地数据目录。

## DNS-01 工作流示例

1. 生成并保存账户私钥，以及另一份独立的证书私钥。
2. 使用证书私钥和需要申请证书的域名生成 CSR。
3. 创建 ACME 账户，再使用账户密钥和 CSR 创建证书订单。
4. 选择 **DNS-01**，获取订单的域名验证参数。在工作流中分别处理每个返回的域名验证项。
5. 使用 DNS 服务商的节点或 API，将每个验证项的 `authorization` 值写入对应的 `_acme-challenge` TXT 记录。
6. 对每条记录执行 **Wait for DNS Text Record**。只有所有配置的 DNS 服务器连续指定次数查询到预期值，等待才会完成。
7. 执行 **Complete Challenge**，再使用相同的 CSR 执行 **Finalize Order**。
8. 执行 **Download Certificate（下载证书）**，将证书与对应私钥一起部署，并通过 DNS 服务商删除临时 TXT 记录。

开发和调试工作流时使用测试环境端点。准备申请生产证书时，再切换到生产环境端点。

## 本地开发

开发环境使用 Node.js 24，版本记录在 `.nvmrc` 中；使用 npm 和仓库中提交的 `package-lock.json` 管理依赖。

```bash
nvm use
npm ci
npm run build
npm run dev
```

如果不使用 nvm，可以通过其他版本管理工具选择 Node.js 24。

| 命令                   | 用途                                              |
| ---------------------- | ------------------------------------------------- |
| `npm run dev`          | 启动 n8n 节点开发环境，并在文件变化后自动重新构建 |
| `npm run build`        | 编译 TypeScript，并将节点资源复制到 `dist/`       |
| `npm run build:watch`  | 仅监听并编译 TypeScript                           |
| `npm run lint`         | 执行现有的 n8n 节点代码规范检查                   |
| `npm run lint:fix`     | 自动修复可修复的代码规范问题                      |
| `npm run format:check` | 使用 Prettier 检查格式                            |
| `npm run format`       | 使用 Prettier 格式化支持的项目文件                |

项目结构和验证流程请参阅 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 兼容性

本项目声明使用 n8n Nodes API 版本 1。目前尚未声明最低 n8n 版本，也没有提供经过测试的 n8n 运行时版本列表。在用于证书自动化之前，请先验证目标 n8n 运行环境。

## 开源协议

[MIT](LICENSE)。
