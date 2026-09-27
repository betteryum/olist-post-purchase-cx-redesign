# 文件 1：GitHub_VSCode_每日执行流程.md

# Olist CX Project GitHub + VS Code Daily Workflow

## Project Goal

项目目标：

使用 Olist Brazilian E-Commerce Dataset 分析 marketplace post-purchase CX failure。

核心路径：

delivery failure

↓

review impact

↓

seller/category/region risk

↓

review pain points

↓

customer journey breakdown

↓

service blueprint

↓

service recovery system

↓

KPI validation plan


---

# 一、每天开始 Before Work


## Step 1：打开 GitHub Repository

打开：

https://github.com/betteryum/olist-post-purchase-cx-redesign


进入：

Issues

↓

Projects

↓

Olist CX Redesign Progress


检查：

In Progress


规则：

同时只能有 1 个 Issue 在 In Progress。


如果没有：

从 Todo 选择一个：

Todo

↓

In Progress


---

## Step 2：打开当前 Issue


阅读：

- Objective
- Tasks
- Definition of Done
- Output Files


回答：

今天结束之前：

我可以完成哪个 checkbox？


不要问：

“今天研究什么？”


只问：

“今天我要关闭哪个任务？”


---

## Step 3：打开 VS Code


打开：

olist-post-purchase-cx-redesign


Terminal:

```bash
conda activate olist-cx
```


检查：

```bash
git status
```


确认：

没有未保存的重要修改。


---

# 二、开始工作 Start Session


在当前 Issue 留评论：


```markdown
## Daily Start

Date:

Current Issue:

Today's Goal:

- 

Expected Output:

Files:
```


例如：


```markdown
## Daily Start

Date:
2026-09-27

Current Issue:

Build master order table


Today's Goal:

- inspect order tables
- create first master table


Expected Output:

data/processed/master_order_table.csv
```


---

# 三、工作规则 During Work


## Rule 1

只做当前 Issue。


不要同时：

- 改 dashboard
- 做 ML
- 写 report
- 做 service blueprint


新的想法：

创建新的 Issue。


例如：

Title:

Future: Build delivery prediction model


Label:

nice-to-have


然后回到当前任务。


---

## Rule 2

按照项目顺序工作


不要跳跃。


顺序：


## Phase 1 Data

01_data_understanding_and_master_table.ipynb


↓

02_late_delivery_baseline.ipynb


↓

03_review_impact_analysis.ipynb


↓

seller/category/region risk


↓

review text coding


↓

repeat purchase analysis


↓

KPI baseline


---

## Phase 2 Service Design


pain point taxonomy


↓

journey map


↓

service blueprint


↓

opportunity matrix


↓

service recovery concept


---

## Phase 3 Final Package


report


↓

presentation


↓

GitHub cleanup



---

# 四、代码修改流程


## 修改前


运行：

```bash
git status
```


---

## 完成一个小任务后


例如：

完成：

data loading


执行：

```bash
git add .
```


commit:


```bash
git commit -m "Complete data loading audit"
```


push:


```bash
git push
```


---

# Commit Message Rules


使用：

```
Build master order table

Add data audit analysis

Analyze late delivery baseline

Add review impact analysis

Create seller risk table

Add service blueprint
```


不要使用：

```
update

change

test

fix
```


---

# 五、每天结束 End Session


## Step 1：更新 Issue


评论：


```markdown
## Daily Update

Completed:

- [ ]

Files Changed:

-

Problems:

-

Next Step:

-
```


---

## Step 2：更新 Checklist


完成：

```
- [ ]
```


改成：

```
- [x]
```


---

## Step 3：检查 Git


运行：

```bash
git status
```


如果有修改：

```bash
git add .

git commit -m "Daily progress"

git push
```


---

## Step 4：更新 Project Board


如果完成：

In Progress

↓

Done


如果没有：

保持：

In Progress



---

# 六、每周 Review


打开：

GitHub Project Board


检查：


## 1. Done 是否增加


如果一周没有 Done：

说明任务太大。


拆 Issue。


---

## 2. Blocked 检查


如果一个 Issue 卡超过 3 天：

创建：

Decision Needed Issue


---

## 3. Scope Check


项目主线：


delivery failure

↓

customer dissatisfaction

↓

risk segmentation

↓

pain points

↓

journey

↓

blueprint

↓

recovery system

↓

validation



---

# 七、项目优先级


## Must Have


必须完成：


- master order table
- late delivery analysis
- review impact analysis
- seller risk
- category risk
- region risk
- review text coding
- pain taxonomy
- journey map
- service blueprint
- recovery concept
- KPI plan
- final report
- presentation


---

## Nice To Have


最后再做：


- ML prediction
- advanced NLP
- dashboard polish
- geospatial analysis
- extra research



---

# 八、Issue 完成标准


一个 Issue 只有满足：

## Code

代码完成


## Output

文件生成


## Evidence

结果可以解释


## Documentation

README / Issue 更新


才可以关闭。


---

# 九、最终完成标准


项目完成：


☐ Data analysis completed

☐ Delivery failure diagnosed

☐ Review impact explained

☐ Risk segments identified

☐ Review pain points coded

☐ Journey map completed

☐ Service blueprint completed

☐ Recovery concept designed

☐ KPI validation planned

☐ Report completed

☐ Presentation completed

☐ GitHub cleaned
```



# 文件 2：每日提醒.md

```markdown
# Olist CX Daily Reminder


## Start


☐ 打开 GitHub

☐ 看 Project Board

☐ 只选择 1 个 Issue

☐ 阅读 Definition of Done

☐ Move to In Progress


☐ 打开 VS Code

☐ conda activate olist-cx

☐ git status



---

## Work


今天只完成当前 Issue。


不要：

☒ 改方向

☒ 做额外功能

☒ 无限研究


记录：

- 做了什么
- 遇到什么问题
- 下一步是什么


完成小任务：

git add

↓

git commit

↓

git push



---

## End


☐ 更新 Issue comment

☐ 勾选完成 checklist

☐ git status

☐ commit

☐ push

☐ 更新 Project Board


---

## 每天问自己


1.

今天关闭哪个任务？


2.

这个分析支持哪个 business question？


3.

这个结果以后会用于：

Data evidence?

Service design?

Recommendation?


---

## 项目主线


Delivery failure

↓

Customer dissatisfaction

↓

Risk identification

↓

Pain points

↓

Journey breakdown

↓

Service blueprint

↓

Recovery system

↓

KPI validation



---

## 当前原则


不要追求每天学很多。


每天提交一个小作业。


Issue = Assignment

Milestone = Deadline

Project Board = Canvas

Commit = Submission

Done = Completed
```


