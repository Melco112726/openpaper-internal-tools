# 许可证与第三方声明

本仓库是封闭内测发布仓库，包含两个独立的 Windows 二进制发布包和配套说明。不同组件的许可证边界如下。

## OpenPaper

OpenPaper 源项目声明使用 MIT License：

> MIT License
>
> Copyright (c) 2026 OpenPaper contributors

完整 MIT 许可文本随 OpenPaper 发布包保留，并可在源项目的 `LICENSE` 文件中查看。OpenPaper 打包的第三方 Python/JavaScript 组件继续受各自许可证约束。

## DoiRescue 与 InstSci Workflow

DoiRescue 使用修改后的 **InstSci Workflow 0.2.0a2**，上游地址：

<https://github.com/deathcats4/instsci-workflow>

InstSci Workflow 基于 MIT 许可的 InstSci 项目。InstSci 原作者和贡献者保留其版权；MIT 许可及上游署名/修改版本说明适用于随包 Worker 中对应的第三方组件。

DoiRescue 当前没有单独对外发布的源码许可证。本仓库中的二进制、说明文字和测试信息不应被理解为对 DoiRescue 原创代码额外授予开源许可。若未来公开源码，应在公开前单独确定许可证和第三方组件清单。

## 隐私与研究资料

许可证不涵盖用户自己的研究数据、论文全文、学校账户、浏览器会话或 API 凭据。发布包已检查并排除：

- 机构账号、密码、Cookie、Token 和授权请求头；
- 浏览器 Profile、登录历史和个人设置；
- 用户下载批次、项目快照和论文全文；
- 个人邮箱、学校信息和本机路径。

使用者仍有责任遵守数据库、出版社、学校/图书馆及论文内容本身的许可条款。
