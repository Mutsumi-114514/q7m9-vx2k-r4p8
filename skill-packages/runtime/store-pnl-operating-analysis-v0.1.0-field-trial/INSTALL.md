# 安装说明 — store-pnl-operating-analysis

这是一个按 Agent Skills 开放格式组织的可安装 Skill 包。

## 目录结构

```text
store-pnl-operating-analysis/
├─ SKILL.md
├─ INSTALL.md
└─ references/
   ├─ production-system-contract.md
   ├─ store-pnl-data-contract.md
   └─ query-scope-and-population-assembly.md
```

## 安装时下载什么

**只下载 / 打包整个 `store-pnl-operating-analysis` 文件夹。**

不要只拿 `SKILL.md`，因为本 Skill 的运行合同已经以相对路径打包在 `references/` 中。

安装后的 Skill 应保持上述目录结构不变。

## ZIP 安装

如果目标平台支持上传自定义 Skill ZIP：

1. 将整个 `store-pnl-operating-analysis/` 文件夹压缩为 `store-pnl-operating-analysis.zip`；
2. ZIP 解压后的顶层应直接得到 `store-pnl-operating-analysis/SKILL.md`；
3. 上传该 ZIP；
4. 安装后用一句真实经营问题做冒烟测试，例如：

```text
请分析南京 202608 综合毛利为什么同比下降，先展示数字水印，再进行正式分析。
```

## 文件系统安装

对于支持 Agent Skills 目录发现的工具，将整个 `store-pnl-operating-analysis/` 文件夹放入其 Skill 目录即可。

例如某些客户端使用：

```text
<skills-root>/store-pnl-operating-analysis/SKILL.md
```

具体 skills-root 位置由目标客户端决定。

## 运行前最低能力

目标运行环境至少需要：

- 能读取用户提供的 Excel / CSV / 表格数据；
- 能做确定性聚合、Join、公式计算；
- 能读取本 Skill 的 `references/` 文件；
- 能保留并输出结构化中间结果；
- 在 Hard Invariant 失败时能够停止而不是继续猜测。

## 本包不包含什么

当前 V0.1 **没有独立 Python / SQL Production Engine**。

也就是说，当前仍属于：

```text
SKILL instructions
+ bundled contracts
+ Agent / runtime calculation capability
```

而不是：

```text
SKILL
+ deterministic executable engine
```

这正是第一轮 Field Trial 要验证的内容之一：哪些确定性能力必须在后续版本下沉成代码。

## 版本纪律

`references/` 是安装包内的 Runtime Snapshot。以后 Canonical Contract 更新时，应从主仓同步更新本 Skill 包，而不是在安装包内单独演化一套平行方法论。
