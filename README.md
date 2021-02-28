# Git-Bash 最佳指北手册

每个人都应当遵循对于分支命名、标记和编码的规范。每个组织都有自己的规范或者最佳实践，并且很多建议都可以从网上免费获取，而重要的是尽早选择合适的规范并在团队中遵循。

## 概述

- DEV 环境 用于开发者调试使用。

- UAT 环境 用户验收测试环境，用于生产环境下的软件测试者测试使用。

- FAT 环境 功能验收测试环境，用于测试环境下的软件测试者测试使用。

- PRO 环境 生产环境。

* 项目域名为：http://www.abc.com，相关环境的域名可这样配置：

      DEV 环境：本地配置虚拟域名即可
      FAT 环境：http://fat.abc.com
      UAT 环境：http://uat.abc.com
      PRO 环境：http://www.abc.com

## 分支

| 分支    | 名称         | 环境 | 可访问 |
| ------- | ------------ | ---- | ------ |
| master  | 主分支       | PRO  | 是     |
| release | 预上线分支   | UAT  | 是     |
| hotfix  | 紧急修复分支 | DEV  | 否     |
| develop | 测试分支     | FAT  | 是     |
| feature | 需求开发分支 | DEV  | 否     |

### master 分支

- master 为主分支，用于部署到正式环境（PRO），一般由 release 或 hotfix 分支合并，任何情况下不允许直接在 master 分支上修改代码。

### release 分支

- release 为预上线分支，用于部署到预上线环境（UAT），始终保持与 master 分支一致，一般由 develop 或 hotfix 分支合并，不建议直接在 release 分支上直接修改代码。

- 如果在 release 分支测试出问题，需要回归验证 develop 分支看否存在此问题。

### hotfix 分支

- hotfix 为紧急修复分支，命名规则为 hotfix- 开头。
  当线上出现紧急问题需要马上修复时，需要基于 release 或 master 分支创建 hotfix 分支，修复完成后，再合并到 release 或 develop 分支，一旦修复上线，便将其删除。

### develop 分支

- develop 为测试分支，用于部署到测试环境（FAT），始终保持最新完成以及 bug 修复后的代码，可根据需求大小程度确定是由 feature 分支合并，还是直接在上面开发。

一定是满足测试的代码才能往上面合并或提交。

### feature 分支

- feature 为需求开发分支，命名规则为 feature- 开头，一旦该需求上线，便将其删除。

这么说可能有点不容易理解，接下来举几个开发场景。

## 开发场景

### 新需求加入

有一个订单管理的新需求需要开发，首先要创建一个 feature-order 分支，问题来了，该分支是基于哪个分支创建？
如果 存在 未测试完毕的需求，就基于 master 创建。
如果 不存在 未测试完毕的需求，就基于 develop 创建。

需求在 feature-order 分支开发完毕，准备提测时，要先确定 develop 不存在未测试完毕的需求，这时研发人员才能将将代码合并到 develop （测试环境）供测试人员测试。

- 测试人员在 develop （测试环境） 测试通过后，研发人员再将代码发布到 release （预上线环境）供测试人员测试。

- 测试人员在 release （预上线环境）测试通过后，研发人员再将代码发布到 master （正式环境）供测试人员测试。

- 测试人员在 master (正式环境) 测试通过后，研发人员需要删除 feature-order 分支。

### 普通迭代

有一个订单管理的迭代需求，如果开发工时 < 1 天，直接在 develop 开发，如果开发工时 > 1 天，那就需要创建分支，在分支上开发。

开发后的提测上线流程 与 新需求加入的流程一致。

### 修复测试环境 Bug

在 develop 测试出现了 Bug，如果修复工时 < 2h，直接在 develop 修复，如果修复工时 > 2h，需要在分支上修复。

修复后的提测上线流程 与 新需求加入的流程一致。

### 修改预上线环境 Bug

在 release 测试出现了 Bug，首先要回归下 develop 分支是否同样存在这个问题。

如果存在，修复流程 与 修复测试环境 Bug 流程一致。

如果不存在，这种可能性比较少，大部分是数据兼容问题，环境配置问题等。

### 修改正式环境 Bug

在 master 测试出现了 Bug，首先要回归下 release 和 develop 分支是否同样存在这个问题。

如果存在，修复流程 与 修复测试环境 Bug 流程一致。

如果不存在，这种可能性也比较少，大部分是数据兼容问题，环境配置问题等。

### 紧急修复正式环境 Bug

需求在测试环节未测试出 Bug，上线运行一段时候后出现了 Bug，需要紧急修复的。
我个人理解紧急修复的意思是没时间验证测试环境了，但还是建议验证下预上线环境。

- 如果 release 分支存在未测试完毕的需求，就基于 master 创建 hotfix-xxx 分支，修复完毕后发布到 master 验证，验证完毕后，将 master 代码合并到 release 和 develop 分支，同时删掉 hotfix-xxx 分支。

- 如果 release 分支不存在未测试完毕的需求，但 develop 分支存在未测试完毕的需求，就基于 release 创建 hotfix-xxx 分支，修复完毕后发布到 release 验证，后面流程与上线流程一致，验证完毕后，将 master 代码合并到 develop 分支，同时删掉 hotfix-xxx 分支。

- 如果 release 和 develop 分支都不存在未测试完毕的需求， 就直接在 develop 分支上修复完毕后，发布到 release 验证，后面流程与上线流程一致。

## 提交代码的姿势

### 检出远程仓库


- git clone https://github.com/Mr-GaoYu/Git-Bash.git

可以检出 origin/master 分支到本地，这是 GitHub 创建仓库时默认的 主机名/分支名。使用 git branch -vv 查看本地分支状态

<img src="/1.png">