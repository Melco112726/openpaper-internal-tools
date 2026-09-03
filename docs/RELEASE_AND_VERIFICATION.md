# 发布与完整性验证

## 1. 本次发布物

GitHub Release 标签建议为：

```text
v0.2.0-internal.2-bundle
```

Release 包含：

| 文件 | 内容 | 大小（约） |
|---|---|---:|
| `OpenPaper-v0.2.0-internal.2-Windows-x64-Portable.zip` | OpenPaper Windows x64 便携内测版 | 72.26 MiB |
| `DoiRescue-v0.1.0-Windows-x64.zip` | DoiRescue GUI + 同级 instsci-worker | 139.70 MiB |
| `SHA256SUMS.txt` | 两个 ZIP 的 SHA-256 | 很小 |

大文件作为 Release assets 发布，不直接提交进 Git 历史。

## 2. SHA-256

```text
915aa1909dd8b75fe6cba8b1ab57a9c1abe8a351b235ccdc22e88c81c3664e43  OpenPaper-v0.2.0-internal.2-Windows-x64-Portable.zip
ba5fcf46dbf4be2b9b8220c7213efb95c2016d37673674a5f5a95dfd6e8a8bfb  DoiRescue-v0.1.0-Windows-x64.zip
```

Windows PowerShell 校验：

```powershell
Get-FileHash -Algorithm SHA256 .\OpenPaper-v0.2.0-internal.2-Windows-x64-Portable.zip
Get-FileHash -Algorithm SHA256 .\DoiRescue-v0.1.0-Windows-x64.zip
```

输出 Hash 应与上方一致。文件名和 Hash 都匹配后再解压运行。

## 3. OpenPaper 发布证据

源发布清单记录：

- 产品版本：`0.2.0-internal.2`；
- 发布通道：`closed-internal-testing`；
- 构建时间：`2026-08-26T17:03:13Z`；
- Python：`3.12.6`；
- PyInstaller：`6.22.2`；
- Playwright：`1.62.0`；
- 150 项 Python 单元测试通过；
- `static/app.js` 前端语法检查通过；
- 最终 ZIP 新鲜解压健康检查通过；
- 项目 CRUD 和双通道增量发布契约通过；
- 服务只监听 `127.0.0.1`；
- 新安装 Sci-Hub 兼容设置为关闭；
- 安全扫描通过；
- Authenticode 未签名。

## 4. DoiRescue 发布证据

本次打包前后确认：

- 当前版本：`0.1.0`；
- 54 项测试全部通过；
- ZIP 含 `DoiRescue/DoiRescue.exe`；
- ZIP 含 `instsci-worker/instsci-worker.exe`；
- Worker `--help` 可启动并列出命令；
- 压缩包未发现 browser-profile、runtime-data、Cookie、Login Data、用户 config/settings、下载批次和个人 PDF 路径；
- 两个 EXE 目录保持同级；
- Worker 固定使用 InstSci Workflow `0.2.0a2`。

## 5. 解压后快速验收

### OpenPaper

1. 完整解压。
2. 双击 `OpenPaper.exe`。
3. 确认浏览器访问 `http://127.0.0.1:8787/`。
4. 确认页面显示版本 `0.2.0-internal.2`。
5. 确认首次设置没有他人的 workspace、邮箱、Key 或历史项目。

### DoiRescue

1. 完整解压。
2. 确认 `DoiRescue` 和 `instsci-worker` 同级。
3. 双击 `DoiRescue\DoiRescue.exe`。
4. 确认界面显示 InstSci 已就绪。
5. 使用少量测试 DOI 验证解析和元数据；不要一开始提交大批机构任务。

## 6. 为什么不用 Git 提交 ZIP

GitHub 普通 Git 文件有单文件大小限制，DoiRescue ZIP 也不适合进入永久 Git 历史。Release assets 更适合二进制发布：

- 文档可以正常版本化；
- ZIP 可独立下载；
- 每个版本有明确标签和说明；
- 更新版本不会让仓库 Git 历史急剧膨胀；
- 可以附带独立校验值。

## 7. 发布前隐私清单

任何新版本发布前都必须确认没有：

- OpenPaper `runtime-data` 或 `%LOCALAPPDATA%\OpenPaper\data`；
- DoiRescue 下载批次或 `%LOCALAPPDATA%\DoiRescue\browser-profile`；
- `.instsci\config.json`、`.doi-rescue\settings.json`；
- Cookie、Login Data、Local State、History；
- API Key、OAuth Token、Authorization Header；
- 完整邮箱、个人 workspace、本机用户名或绝对路径；
- 用户下载的论文 PDF；
- 未脱敏日志和截图。

## 8. 版本更新建议

- OpenPaper 与 DoiRescue 分别维护自己的版本号；
- Bundle 标签说明两个具体版本，不暗示两者必须同步升级；
- 新 ZIP 解压到新目录，不覆盖旧程序；
- 更新前备份本机数据目录；
- 发布后重新下载资产并复算 SHA-256，确认 GitHub 文件与本地一致。
