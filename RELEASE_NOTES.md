# v0.2.0-internal.2-bundle

这是 OpenPaper 与 DoiRescue 的首次联合封闭内测发布。两个程序保持独立安装和独立运行，通过 DOI 缺失清单完成接力。

## 发布资产

- `OpenPaper-v0.2.0-internal.2-Windows-x64-Portable.zip`
- `DoiRescue-v0.1.0-Windows-x64.zip`
- `SHA256SUMS.txt`

## OpenPaper v0.2.0-internal.2

OpenPaper 是本地优先的文献研究工作台，本版包含：

- 英文本地检索、可选 Undermind 深度检索、中文/知网通道与双通道合并；
- 人工复核、来源保留、身份归一与项目快照；
- 可核验引用关系和本地语义关系组成的关系图谱；
- Crossref、OpenAlex、Unpaywall、Europe PMC 等合法来源组成的默认全文流程；
- 下载成功清单和 `*-missing-dois.txt`，用于与 DoiRescue 交接；
- Windows x64 Portable 打包、回环地址服务、DPAPI 保护和隐私扫描。

## DoiRescue v0.1.0

DoiRescue 是 OpenPaper 第三阶段之后的独立 DOI 文献补漏器，本版包含：

- 文件拖入、参考文献粘贴、DOI 规范化、去重和元数据识别；
- 先按出版社建立 Provider 计划，再逐组执行，避免混合出版社整批失败；
- OA、机构访问和人工处理三个边界清晰的阶段；
- 可见机构浏览器、专用持久 Profile、任务跳过/重试/停止和超时保护；
- 人工处理队列、DOI 官方链接、可信出版社链接及本地 PDF 关联；
- PDF 文件头、大小、基本可读性和元数据匹配检查；
- 可恢复 manifest，以及成功、待机构、待人工、跳过和失败清单。

`instsci-worker.exe` 是随 DoiRescue 打包的后台 Provider 执行器，由主程序按需调用，不是用户单独启动的第二个应用。

## 安装与升级

1. 分别下载所需 ZIP 和 `SHA256SUMS.txt`。
2. 核对 SHA-256。
3. 将 ZIP 完整解压到普通本地目录，不要直接在压缩包内运行。
4. OpenPaper 运行根目录中的 `OpenPaper.exe`。
5. DoiRescue 运行 `DoiRescue\DoiRescue.exe`，并保留相邻的 `instsci-worker` 目录。
6. 不要用新版程序目录覆盖旧目录；先备份运行数据，再按文档迁移。

## 发布验证

- OpenPaper：150 项 Python 测试通过，前端语法检查、打包健康检查和安全扫描通过。
- DoiRescue：54 项测试通过，冻结版 GUI 和 Worker 可启动，压缩包结构和敏感路径检查通过。
- 两个发布 ZIP 均不包含个人浏览器 Profile、Cookie、账号密码、下载批次或个人论文全文。

## 内测限制

- 仅提供 Windows 10/11 x64 构建，EXE 尚未进行 Authenticode 签名。
- 外部数据库、出版社、Undermind 和机构认证的可用性不由本项目保证。
- 机构访问仅适用于使用者真实拥有权限的学校/图书馆订阅。
- DoiRescue 不处理 CNKI/万方 ID 或 CAJ；没有 DOI 的中文题录应在 OpenPaper 中文通道中处理。
- PDF 自动校验不能替代研究者对题名、作者、年份、版本和页码的最终核对。

完整操作方法、状态解释、安全边界和排障方式见仓库首页及 `docs/` 目录。
