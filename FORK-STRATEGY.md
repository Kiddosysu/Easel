# Easel Fork 二开策略

本仓库是 [ZJU-REAL/Easel](https://github.com/ZJU-REAL/Easel) 的团队 fork，用于 **24SITES 社媒代运营**业务的定制二开。本文件是二开协作的约定入口；业务侧上下文见核心舱 `.projects/socialmedia/AGENTS.md`。

> 上游项目：Apache-2.0 · Python 3.10+

## 分支模型

| 分支 | 内容 | 规则 |
|------|------|------|
| `main` | **与上游 `upstream/main` 完全一致**，不含任何自研代码 | 只通过合并 `upstream/main` 前进；**永远不在 main 上直接提交** |
| `custom/meta` | Meta 系（Facebook / Instagram）平台适配等全部二开成果 | 日常工作分支；核心舱 submodule 指向此分支 |

`main` 保持纯净的唯一目的：让「跟上开源迭代」这件事变成零冲突的机械动作。任何直接提交进 `main` 的自研代码，都会在未来每一次 `git merge upstream/main` 时变成反复出现的冲突源。

## remote

| remote | 地址 | 用途 |
|--------|------|------|
| `origin` | `git@github.com:Kiddosysu/Easel.git` | 本 fork，二开成果推送目标、向上游提 PR 的来源 |
| `upstream` | `https://github.com/ZJU-REAL/Easel.git` | 开源上游，**只读**（push URL 已禁用，物理防止误推） |

首次在新机器上配置：

```bash
git remote add upstream https://github.com/ZJU-REAL/Easel.git
git remote set-url --push upstream DISABLED-readonly-forbidden
```

## 同步上游

**用 merge，不要 rebase。** 二开分支一旦推送、被核心舱 submodule 引用或被他人在 PR 中引用，rebase 重写历史会让所有引用失效。

```bash
git checkout main
git fetch upstream
git merge upstream/main        # main 追平上游
git push origin main           # 保持 fork 的 main 与上游一致

git checkout custom/meta
git merge main                 # 把上游更新并入二开分支，冲突在此解决
```

建议在开始一批新二开前先做一次上游同步，使定制建立在较新的基线上，避免定制代码与上游新实现重复。

## 二开约定

**1. 提交前缀。** 二开提交的 message **必须以 `[custom]` 开头**，便于随时审计全部定制内容：

```bash
git log --grep='\[custom\]' --oneline
```

**2. 改动优先级（冲突风险从低到高）。** 优先选择冲突面最小的方式：

1. **新增独立 skill 目录** — 首选。Easel 的发布与创作能力是「一能力一 skill 目录」的纯目录约定（`skills/openclaw/skill-*`），无中心注册表，新增即生效、几乎不会与上游冲突。
2. **新增 `skills/shared/` 下的共享脚本** — 低冲突。
3. **修改上游已有文件** — 尽量避免。必须修改时，单独成一个 `[custom]` 提交，不要与其它改动混在一起，以便跟版时快速定位冲突来源。

**3. Meta 系适配的落点。** 参照上游已有的平台发布 skill（`skill-xhs-publisher`、`skill-douyin-upload`、`skill-bilibili-upload`），Meta 系并列新增：

```
skills/openclaw/skill-facebook-publisher/
skills/openclaw/skill-instagram-publisher/
```

新 skill 遵循 `docs/SKILL-SPEC.md`：目录内含 `SKILL.md`，frontmatter 仅保留 `name` / `description` / `layer` 三个字段，`layer` 取 `publish`；领域知识放 `references/`，可执行脚本放 `scripts/`。SKILL.md 主体控制在 200 行内。

## 红线：凭证与业务数据绝不入库

本 fork 是**公开可读**的。以下内容一律不得提交，等同公开发布：

- 平台登录态：`cookies.json`、`*.jsonl` 会话、各平台 token
- 密钥：`.env`、`.env.*`、`AppSecret`、API key（微信公众号发布器配置文件已在 `.gitignore` 中）
- 客户信息与真实运营数据：账号名称、客户名单、投放数据、内容产物

上游 `.gitignore` 已覆盖 `outputs/**/*`、`assets/**/*`、`profiles/*`、`cookies.json`、`.env*` 等路径，**不要用 `git add -f` 绕过**。业务私有信息应写入核心舱 `.projects/socialmedia/docs/`（核心舱为私有仓库），或仅保留在本地。

## 目录速览

```
easel/                Python CLI（chat / web / skill / doctor / ping）
web/                  FastAPI 后端与 React 工作台
skills/openclaw/      发现 / 策划 / 制作 / 发布 / 归因 能力（一能力一目录）
skills/shared/        跨 skill 共享脚本与配置
docs/                 能力规范、能力地图、架构文档
profiles/             账号画像（仅 _template 入库，实际画像忽略）
outputs/              内容产物（运行期生成，不入库）
openclaw/             隔离 profile、workspace 与同步脚本
```

## 最后更新

2026-09-30

## 审阅人

kiddosysu
