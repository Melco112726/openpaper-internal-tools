# 设计决策与项目演进

这份文档解释为什么项目最终形成“OpenPaper 主工作台 + DoiRescue 独立补漏器 + InstSci Worker”的结构。

## 1. 最初的问题

OpenPaper 已经具备：

- Undermind、Crossref 和 OpenAlex 文献发现；
- Unpaywall、Europe PMC 等 OA 线索；
- DOI 批量全文流程；
- 中文/知网通道；
- 项目快照与历史续作；
- 文献关系图谱；
- 未获取全文清单。

真正的缺口不是“再造一个检索器”，而是第三阶段结束后仍有 DOI 需要继续处理：

1. OA 来源未成功；
2. 需要已有机构权限；
3. 出版社工具暂不支持；
4. 需要用户从官方页面人工下载；
5. 成功、需登录、不支持、未找到和程序错误被混为一类。

## 2. 为什么不把多个 Skill 全部加入 OpenPaper

候选仓库中，多个项目重复提供 CNKI 搜索、浏览器操作或全文获取。全部复制进 OpenPaper 会带来：

- 功能重复，Agent 不知道同一任务应该选哪个工具；
- Playwright、Chrome DevTools、Camofox、CloakBrowser 等会话与端口冲突；
- 不同 Profile、Cookie 和验证码状态互相干扰；
- `instsci` 与 `instsci-workflow` 使用相同包名和命令名，不能装在同一 Python 环境；
- 主项目依赖和发布体积持续膨胀；
- 某个外部网站变化会拖累整个 OpenPaper。

因此采用原则：**每类能力只保留一个主实现，其他能力作为可替换 Provider 或外部交接。**

## 3. 为什么选择独立桌面补漏器

DoiRescue 被设计成单独 EXE，而不是 OpenPaper 新页面，原因是：

- OpenPaper 的研究项目和选择状态保持稳定；
- 补漏器可以独立崩溃、停止、更新或卸载；
- 机构浏览器和 Provider 依赖不会进入 OpenPaper 环境；
- 文件交接容易检查，也方便用户理解数据边界；
- 未获取清单可以来自 OpenPaper，也可以来自其他文献管理流程。

第一版采用 `missing-dois.txt` 文件交接，不直接写 OpenPaper 的 `projects`、`runs`、`downloads` 或 `graph_cache`。

## 4. 为什么使用 instsci-workflow

在 `instsci` 和 `instsci-workflow` 之间，本项目选择后者作为固定 Provider，因为它更接近“OA 优先—机构访问—出版社批量—结果 manifest”的完整工作流。

但 Provider 被包在独立 Worker 中：

- DoiRescue GUI 不直接依赖 Provider 的内部页面实现；
- Worker 可以有自己的命令、依赖和浏览器工具；
- GUI 只消费结构化计划、状态和 manifest；
- 将来更换 Provider 时，影响集中在适配层。

当前固定版本为 InstSci Workflow `0.2.0a2`，来源提交：

```text
3c3bb4f4da5c73d23b198a791ce1068c5112b92d
```

## 5. 从失败批次得到的第一组修复

最初的混合 DOI 批次暴露了四个核心问题：

1. Unpaywall 邮箱为空仍发送 `?email=`，导致请求错误；
2. 多个出版社被整体传入 `--publisher auto`，自动推断无法代表整批；
3. `unsupported_publisher` 被错误显示为 `not_found`；
4. 机构按钮只看错误状态，用户无法继续合法机构访问。

因此加入：

- 邮箱设置与“跳过 Unpaywall”；
- 逐 DOI 出版社推断与 `provider-plan.json`；
- OA、机构和人工三阶段分离；
- 更细的状态语义；
- 机构清单、人工清单和下一步动作。

## 6. 从浏览器卡死得到的第二组修复

机构访问要求用户处理登录、Cookie、验证码和学校选择。用户关闭某篇浏览器后，隐藏 Worker 曾可能继续等待，导致整批停在“处理中”。

修复方向是：

- 每篇 DOI 是独立队列项；
- Worker 有硬超时和浏览器存活监测；
- 可以跳过、重试或停止当前篇；
- 停止使用精确进程树，不误杀用户自己的浏览器；
- 部分 manifest 和已验证 PDF 立即保留；
- 关闭外部浏览器不能永久阻塞整个队列。

## 7. 为什么不并发打开多个机构网站

虽然批量并发看似更快，但当前浏览器实现和人工登录流程共享 Profile。并发会增加登录态覆盖、验证码冲突、下载归属混乱和误停止风险。

最终选择：

- OA 与普通元数据任务可以有限并发；
- 交互式机构任务固定并发数 1；
- 用“跳过当前篇 / 重试当前篇 / 停止全部”提高可控性；
- 先做好稳定的队列，再考虑未来是否存在安全的多 Profile 并发方案。

## 8. 为什么复用会话但不保存密码

用户重复访问同一机构或出版社时，完全重新登录成本很高。项目选择复用 DoiRescue 专用浏览器 Profile 中的有效会话，但不由 DoiRescue 保存明文账号密码。

这一区分很重要：

- 浏览器 Profile 可以保存站点 Cookie/localStorage；
- 密码自动填充交给 Chrome/Edge 或 Windows 安全机制；
- DoiRescue 不读取、导出或记录这些值；
- 验证码、二维码和 2FA 仍由用户完成；
- 清除会话只允许删除 DoiRescue 自己的专用 Profile。

## 9. 为什么需要人工处理队列

自动工具无法覆盖所有出版社、WAF 和页面流程。人工队列不是“失败垃圾桶”，而是正式工作阶段：

- 始终保留规范化 DOI 官方链接；
- 可信时记录最终出版社地址；
- 每次只打开一篇，避免网页风暴；
- 用户可记录原因和进度；
- 手动 PDF 经校验后才进入成功状态；
- 软件重启后从 manifest 恢复。

## 10. 为什么保留详细 manifest

单一“成功/失败”无法回答：

- 是否真正查询过？
- OA 是否检查过？
- 出版社是否被识别？
- 是否尝试机构访问？
- 是网络错误、权限不足还是工具暂不支持？
- 用户下一步应该做什么？

因此结果保留 `publisher`、`provider_status`、`failure_stage`、`failure_reason`、`next_action`、`oa_checked`、`institution_attempted`、官方链接、人工处理和 PDF 校验字段。

## 11. 最终职责划分

| 能力 | 唯一主实现 |
|---|---|
| 研究问题、候选、选择状态和项目续作 | OpenPaper |
| 英文/中文发现 | OpenPaper |
| 国际关系图谱 | OpenPaper Graph |
| 默认 OA 获取 | OpenPaper 第三阶段 |
| 第三阶段之后的 DOI 补漏 | DoiRescue |
| 出版社/机构执行 | DoiRescue 内的 InstSci Worker |
| 人工 PDF 关联与补漏审计 | DoiRescue |
| 密码、验证码和 2FA | 用户与浏览器/Windows 安全机制 |

这个结构的目标不是让功能最多，而是让边界清楚、失败可恢复、状态可解释。
