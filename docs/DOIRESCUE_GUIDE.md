# DoiRescue 完整使用指南

适用版本：`0.1.0`，Windows 10/11 x64。

## 1. 它为什么存在

OpenPaper 第三阶段已经覆盖默认 OA 与部分授权来源，但现实中仍会留下：

- 出版社已识别、需要学校或图书馆登录的 DOI；
- 当前 Provider 暂不支持的出版社；
- 网络超时、WAF、验证码或页面结构变化造成的失败；
- 需要用户从官方页面手动下载并关联的 PDF。

DoiRescue 把这些情况从一个模糊的“失败”拆成可以继续执行的队列。它是独立 EXE，不写回 OpenPaper 项目。

## 2. 包内程序

```text
DoiRescue-v0.1.0-Windows-x64\
├─ DoiRescue\
│  ├─ DoiRescue.exe          # 用户打开的桌面主程序
│  └─ _internal\            # GUI 运行依赖
└─ instsci-worker\
   ├─ instsci-worker.exe     # 后台 Provider Worker
   └─ _internal\            # InstSci、浏览器与 PDF 依赖
```

两个目录必须保持同级。`instsci-worker.exe` 不需要用户单独双击。

## 3. 从 OpenPaper 交接

1. 在 OpenPaper 第三阶段完成本轮全文任务。
2. 保存或找到 `*-missing-dois.txt`。
3. 打开 DoiRescue。
4. 把文件拖到左上方导入区，也可点击“导入文件”。
5. 如果没有清单，可直接粘贴中英文参考文献并点击“解析文本”。
6. 确认表格中的 DOI 数量和去重结果。

程序读取输入文件，不会写回或删除 OpenPaper 文件。

## 4. DOI 解析与元数据

DoiRescue 会：

- 去除 `doi:`、`https://doi.org/` 等常见前缀；
- 统一转为小写，清理尾部标点并去重；
- 从一段参考文献文本中提取 DOI；
- 按需从 Crossref 获取题名、作者、年份和期刊；
- 根据题名与声明信息判断中文、英文或未知；
- 为每个 DOI 生成 `https://doi.org/{normalized_doi}`。

元数据识别失败不会删除 DOI，用户仍可进入后续流程。

## 5. 首次设置

点击“设置”，可配置：

- Unpaywall 联系邮箱；
- 机构中文名称；
- 机构英文名称；
- 是否启用机构访问；
- 机构登录等待上限；
- 是否复用专用浏览器会话；
- Profile 状态、位置和最后使用时间。

设置文件：

```text
%USERPROFILE%\.instsci\config.json
%USERPROFILE%\.doi-rescue\settings.json
```

浏览器会话：

```text
%LOCALAPPDATA%\DoiRescue\browser-profile
```

程序不提供学校密码输入框。账号密码、验证码和 2FA 不进入 DoiRescue 配置；自动填充由浏览器或 Windows 密码管理机制负责。

邮箱为空时会跳过 Unpaywall，不会发送 `?email=` 空参数。开始任务时用户也可以明确选择跳过 Unpaywall。

## 6. Provider 分组

开始下载前，Worker 使用 InstSci 的 publisher profile 推断每个 DOI 的出版社，并生成 `provider-plan.json`。

支持的记录按出版社分组；无法推断的记录进入 `unknown`。这样可以避免把 Wiley、Elsevier、Springer 等混合列表整体交给 `--publisher auto` 后一次性失败。

Provider plan 是计划，不代表已经下载，也不代表 unknown DOI 不存在。

## 7. OA 阶段

点击“开始下载”后：

1. 仅运行 OA-only 流程。
2. 不启动机构浏览器。
3. 不在隐藏控制台等待机构名称或用户输入。
4. 尝试缓存、合法开放来源、arXiv 或出版社开放 PDF。
5. 校验下载候选是否为 PDF。
6. 按结果更新状态。

状态映射：

- OA 成功：`downloaded`；
- 出版社已支持但 OA 未成功：`auth_required`；
- 出版社未知：`manual_required` / `unsupported`；
- 网络或程序异常：`failed`；
- 经过有效查询确认没有记录：`not_found`。

`unsupported_publisher` 不能被映射为 `not_found`。

## 8. 机构访问队列

当存在 `auth_required` 记录时，“使用机构权限”按钮可用。

机构阶段只处理：

- 已识别出版社；
- OA 阶段未成功；
- 用户仍选择处理的记录。

每篇任务都显式传入 publisher 和 institution。浏览器保持可见，由用户手动完成登录、SSO、验证码、二维码或 2FA。

### 会话复用

启用“复用浏览器登录状态”后，后续任务使用同一个 DoiRescue 专用 Profile。已有 Cookie/localStorage 可能减少重复登录，但能否继续使用由机构和出版社决定。

“清除当前登录会话”只允许删除 DoiRescue 默认专用 Profile，并在执行前要求确认。程序拒绝清除用户日常 Chrome/Edge Profile。

### 为什么机构并发固定为 1

多个并行出版社窗口共享同一 Profile 时，容易造成：

- 登录和验证码状态互相覆盖；
- 下载文件归属不清；
- CloakBrowser 单会话限制；
- 用户来不及判断每个页面；
- 一个关闭事件误伤其他任务。

因此当前版本选择“一个交互式任务 + 可跳过/重试/停止”，而不是一次弹出多个网站。

### 队列控制

- **跳过当前篇**：结束当前 Worker 树，记录跳过，继续下一篇。
- **重试当前篇**：保留已有结果，重新运行该 DOI。
- **停止全部**：停止当前 Worker 及其子进程，保留已完成 PDF 和部分 manifest。
- **关闭浏览器**：只影响当前 DOI；程序应在超时或浏览器退出后回收任务，不永久停留在处理中。

程序按精确 PID 管理 Worker 及其 Playwright 子进程，不按进程名结束用户自己的 Chrome 或 Node。

## 9. 人工处理队列

进入条件包括：

- `manual_required`；
- `unsupported`；
- 机构访问后仍未下载且需要人工确认；
- WAF、页面验证或 PDF 候选冲突。

每条记录可以：

1. 打开标准 DOI 官方解析页；
2. 如果重定向成功且域名可信，打开最终出版社页；
3. 选择人工处理原因；
4. 标记正在处理、完成或跳过；
5. 选择本地 PDF 并与记录关联；
6. 使用上一篇/下一篇继续队列。

一次默认只打开一篇，不自动弹出大量网页。打开页面不等于下载成功。

## 10. 人工关联 PDF

选择本地 PDF 后会执行：

1. 检查文件存在且大小合理；
2. 检查 `%PDF-` 文件头；
3. 计算 SHA-256；
4. 尽可能在可读内容中搜索目标 DOI；
5. 若无 DOI，则比对题名关键词；
6. 给出 `high`、`medium`、`low` 或 `unknown` 匹配置信度；
7. 低置信度时先警告，由用户决定是否继续；
8. 复制到批次 `pdf` 目录，不删除原文件；
9. 使用不会覆盖现有文件的稳定文件名；
10. 更新 manifest 与各类清单。

无效 PDF 会进入 `quarantine`。扫描件可能没有可搜索文本，仍需人工核对。

## 11. 状态说明

| 机器状态 | 界面含义 | 是否已有有效 PDF |
|---|---|---|
| `pending` / `metadata_ready` | 等待或已获取元数据 | 否 |
| `downloading` / `queued` / `running` | 任务处理中 | 尚未确认 |
| `downloaded` | 已保存并通过基础校验 | 是 |
| `auth_required` | 需要机构访问 | 否 |
| `manual_required` / `unsupported` | 需人工处理 | 否 |
| `manual_in_progress` | 用户正在处理 | 否 |
| `manual_completed` | 已记录人工动作但仍无 PDF | 否 |
| `skipped` | 用户跳过 | 否 |
| `not_found` | 有效查询后未找到记录 | 否 |
| `failed` | 网络、请求或程序错误 | 否 |
| `cancelled` | 任务被取消 | 否 |

只有程序确认有效 PDF，或用户选择本地 PDF 并确认关联后，状态才能成为 `downloaded`。

## 12. 批次输出

```text
<时间戳>_doi-rescue\
├─ pdf\
├─ quarantine\
├─ provider\
├─ manifest.json
├─ success.csv
├─ still-missing-dois.txt
├─ institution-required-dois.txt
├─ manual-required-dois.txt
├─ manual-required-links.html
├─ manual-required-links.csv
├─ skipped-dois.txt
├─ failed-dois.txt
├─ institution-queue.json
├─ provider-plan.json
└─ run.log
```

`manifest.json` 是主审计记录，包含 publisher、provider_status、failure_stage、failure_reason、next_action、oa_checked、institution_attempted、官方链接、人工处理和 PDF 校验字段。

## 13. 恢复批次

软件重启后点击“恢复批次”，选择含 `manifest.json` 的旧批次目录：

- 已下载记录保持成功，不重复覆盖 PDF；
- 中断时仍为 running 的记录恢复为可重试状态；
- 机构和人工队列继续显示；
- 已保存的手动处理进度继续保留。

## 14. 使用边界

- 只处理 DOI，不处理 CNKI/万方 ID 或 CAJ。
- 不保证所有机构都订阅目标文献。
- 不绕过验证码、2FA、访问控制或付费墙。
- 不保存学校密码，不自动提交登录表单。
- 不集成 Sci-Hub。
- PDF 基础校验不能替代最终学术核对。
