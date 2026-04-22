目录结构就按“来源类型分目录”来定，这个最稳，后续再通过标签做主题视图，不会把物理存储结构绑死在一套分类法上。

  下一个高影响点是 X 的抓取方式，因为这会直接决定稳定性、合法性风险和维护成本。

• Questions 1/1 answered
  • 第一版里，X 账号内容你希望怎么获取？
    answer: RSS/第三方镜像优先 (Recommended)

• RSS/第三方镜像优先 适合第一版，先把主链路跑通，再决定是否为少数高价值账号补登录态抓取。

  GitHub 也需要收窄一下，不然“有更新”这个定义会过宽。

• Questions 1/1 answered
  • 第一版里，GitHub 仓库哪些变化算值得入库？
    answer: Release + 默认分支新提交 (Recommended)

• GitHub 就按 Release + 默认分支新提交 收，信息密度比较合适。网页源也要定一个边界，不然课程页、博客页、导航页会混在一起。

• Questions 1/1 answered
  • 通用网页在第一版里，什么页面最值得支持？
    answer: 文章页/课程目录页 (Recommended)

• 网页范围就收敛到“文章页/课程目录页”，这能避免第一版陷入通用爬虫产品化。

  还要锁一个核心偏好：摘要与分类是完全规则化，还是允许调用 LLM 来生成高质量 summary。

• Questions 1/1 answered
  • 第一版的 summary 和标签分类，你接受调用 LLM 吗？
    answer: 允许调用 LLM (Recommended)

• 那就按 LLM 生成高质量 summary + 标签 设计，原始内容仍然保留，避免笔记只剩模型转述。

  Discord 这边还差一个时效性决策：是每天跑完立刻发，还是只发每日总览。

• Questions 1/1 answered
  • 每天抓取完成后，Discord 通知你希望怎么发？
    answer: 每日汇总一条 (Recommended)

• 现在需求已经足够稳定，可以收敛成几种实现路线。我建议先不再加新需求，先把架构方向定掉。

• 基于你刚才锁定的偏好，我建议走这 3 种方案里的第 1 种。

  1. 文件驱动的本机采集器，推荐
     一个本机定时任务每天运行一次，分别拉取 X / GitHub / 通用网页，做增量检测，把新增内容写入 Obsidian Markdown，再调用 LLM 生成标题摘要和标签，最后发一条 Discord 每日
     汇总。优点是最贴合你的使用方式，系统边界清晰，维护成本最低。
  2. 数据库存储 + Markdown 导出
     先把原始抓取和变更记录存进 SQLite，再导出为 Obsidian Markdown。优点是后续检索、去重、回溯更强；缺点是第一版会明显更重，而你当前核心价值还是“沉淀到 Obsidian”。
  3. 自动化平台拼装
     用现成服务拼一个版本，比如抓取 + summarization + Discord + 文件写入。优点是启动快；缺点是源类型一复杂就会很快失控，长期可维护性差。

  我推荐方案 1，原因很直接：你的核心资产是 Obsidian 里的学习资料，不是后台系统本身。第一版应该让“每天有新增，自动沉淀成可读笔记，并提醒你”这件事先稳定发生。

  先给你第一段设计，确认整体骨架是否对：

  - 运行方式：本机 cron/launchd 每天执行一次
  - 数据源适配器：
    X 走 RSS/镜像源
    GitHub 走 release + default branch commits
    Web 走文章页/课程目录页
  - 增量检测：
    统一抽象成“source item”，用稳定 ID 去重，只识别新增，不追踪细粒度编辑
  - 内容处理：
    保留原始标题、链接、时间、正文/摘要片段
    再调用 LLM 生成中文核心 summary、建议标题和标签
  - Obsidian 存储：
    按来源类型分目录，如 AI-Feeds/X/、AI-Feeds/GitHub/、AI-Feeds/Web/
    每条新增内容生成一个 Markdown 文件
  - Discord：
    每天任务结束后发 1 条 digest，包含当天新增条目摘要和对应 Obsidian 笔记路径/链接说明

  这段骨架如果你认可，我下一条会继续把“Markdown 结构、去重规则、失败重试和最小配置格式”补到可实现级别。