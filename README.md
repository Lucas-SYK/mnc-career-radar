# Campus Radar — 外企校招雷达

为中国大陆自动化、控制科学与工程、工业软件、机器视觉方向设计的官方招聘导航站 MVP。

> **真实功能边界**：当前版本默认包含 8 家**核实存在的企业官方招聘入口**以及 1 条**经人工核实的 2027 校招项目**。每日任务会刷新站点时间和检查官方入口；Greenhouse、Lever、JobPosting 适配器已经实现，但**未配置真实公开 ATS 公司标识时，不会自动发现其所有新岗位**。不能把官网入口可访问写成岗位仍开放。项目不自动投递简历。

## 立即运行

需要 Python 3.10+，建议 3.12。

```bash
python -m pip install -r requirements.txt
python -m unittest discover -s tests -v
python scripts/update_jobs.py --offline
python -m http.server 8000 -d docs
```

然后访问 `http://localhost:8000`。`--offline` 仅用于离线演示，不进行网络核验；去掉此参数才会检查官方网页和已配置的公开招聘接口。

## 不开电脑的 GitHub 部署

1. 在 GitHub 创建公开仓库，例如 `foreign-campus-radar`，默认分支命名 `main`。
2. 解压本项目，**将里面的所有文件夹和文件上传到仓库根目录**（不要把压缩包本身当作代码上传）。确保 `.github/workflows/daily-update.yml` 文件存在。
3. 进入仓库 `Settings → Pages → Build and deployment → Source`，选择 **GitHub Actions**，而不是 `Deploy from a branch`。
4. 进入 `Actions → Update & Deploy Campus Radar → Run workflow`，手动运行一次。等待成功后到 `Settings → Pages` 查看网站地址，通常形如 `https://你的GitHub用户名.github.io/foreign-campus-radar/`。
5. 工作流中 `cron: '0 16 * * *'` 是 UTC 16:00，即**中国北京时间次日 00:00**。GitHub Actions 有排队和延迟，**不能保证精确到秒**；公开仓库长期无活动时，GitHub 还可能暂停 schedule。要求严格整点触发应改为有 SLA 的外部调度器。

GitHub Pages 发布的站点是**公开可访问的**。不要把真实简历、身份证号、电话、密码、API 密钥或登录 Cookie 推送到此仓库。网站当前的收藏和已投递记录使用浏览器 localStorage，仅保存于该浏览器。

### 2027届不是你的真实毕业届别怎么办？

`config/jobs_curated.json` 里的 `cohort` 来自招聘公告；它不是对使用者毕业年份的假设。页面可按届别筛选。需要自定义匹配分数时改 `docs/app.js` 中 `score()`，当前分数只代表规则式关键词相关度，不代表企业对你的录用概率。

## 增加企业与真实岗位

- 在 `config/companies.json` 添加已核实的外资企业中国招聘官网 URL、名称与方向。
- 如果发现**真实、公开、属于该企业**的 Greenhouse board token，在 `config/integrations.json` 的 `greenhouse` 中添加 `{"company_id":"企业id","board_token":"核实过的token"}`。
- 对 Lever，用 `"lever":[{"company_id":"企业id","site":"核实过的site"}]`。
- 对带有标准 `schema.org/JobPosting` 的官方职位详情页面，用 `"jsonld_job_pages":[{"company_id":"企业id","url":"https://官方职位详情页"}]`。列表页不一定包含结构化岗位信息。
- 没有公开接口的岗位，由人核实后追加到 `config/jobs_curated.json`。不要捏造 API 标识或招聘状态；不要绕过验证码、登录、反爬或招聘网站使用限制。
- 工作流每天运行、去重、输出到 `docs/data` 并部署。若任何源无法访问，不会把失败解读为岗位关闭。

### 自动采集规则

脚本只保留同时匹配 **中国大陆地点**、**校园/初级岗位信号**、**专业相关关键词** 的岗位。保守策略可能漏掉真实岗位，尤其是中文或非标准分类。自动结果始终写明“需官网核对”。实际生产化应该为各家公司建立经过验证的适配器、招聘年级匹配、截止日期核对、数据留痕、官方 URL 保活和灰度测试。

## 一键投递的正确实现路线

- V1：点击卡片进入**企业官方网站**，由求职者确认后投递；网页本地记录收藏/已投。
- V2：本地或本人可信的登录环境使用 Playwright **辅助填写表单**，个人信息保存在本机或加密私有存储，**提交前必须人工确认**。
- V3：只有在招聘网站明确允许且本人获授权使用的 API 存在时，才能讨论自动提交；企业账号、验证码、短信验证、求职确认和合规条件不能被绕过。GitHub Pages 是静态托管平台，不能在服务端安全保存个人简历。

不要直接用公开的“LinkedIn 机器人”套件套国内求职流程，域名、验证码、学历年级与中文字段差异很大。

## 项目结构

```text
.github/workflows/daily-update.yml  # 北京时间0点的计划任务+部署
config/companies.json               # 真实官方外企招聘渠道
config/jobs_curated.json            # 人工核实校招项目
config/integrations.json            # 可配置 Greenhouse/Lever/JobPosting
scripts/update_jobs.py              # 更新、过滤、去重、官方来源检查
docs/index.html                      # 纯静态网站
docs/styles.css                      # 自适应桌面/手机界面
docs/app.js                          # 筛选、匹配、收藏、投递跟踪、CSV导出
docs/data/                         # 公开快照，不含个人简历
tests/test_update.py                # 后端过滤/适配器单元测试
AGENTS.md                           # Codex 云端工程约束
```

## Codex Cloud

在 `https://chatgpt.com/codex` 连接刚建的 GitHub 仓库，创建 Cloud Environment，Python 3.12。Setup script 可以设置为：

```bash
python -m pip install -r requirements.txt
```

如果让 Codex 云端亲自访问招聘网站进行测试，需要在云端环境启用限制性的联网，仅开启需要的官方域名并限制为 GET/HEAD 等只读操作。**Codex 云端是协作开发工具，不是每天 0 点保持常驻的服务。** 日常定时运行由 GitHub Actions 完成。

详见 `AGENTS.md` 的任务、验收标准和安全约束。
