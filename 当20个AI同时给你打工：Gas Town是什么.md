> Steve Yegge花了半年时间，做了4个版本的编排系统，最终在2026年元旦发布了Gas Town

## 

这是个什么东西

2026年1月1日，Steve Yegge在Medium发了篇长文，介绍他做的新项目Gas Town。简单说，这是一个让你同时跑20-30个Claude Code实例的系统。

Steve Yegge是谁？在Amazon干了7年（1998-2005），在Google干了13年（2005-2018），后来去了东南亚的Grab，2022年加入Sourcegraph当工程主管。他最出名的是写技术博客，2011年还因为不小心把Google内部memo发到Google+上了闹出过大新闻。

Gas Town不是他第一次尝试做这种多agent编排系统。按他自己说的，这是他的第四个版本——前三个都失败了。其中第二个版本的失败产物变成了Beads项目。当前版本用Go写的，12月15日开始写第一行代码。

## 

问题：一个Claude Code不够用

用Claude Code干大活会碰到几个问题：

上下文会满。跑几个小时，对话窗口就满了，会话自动结束。你得重开一个，然后手动告诉新会话之前干了啥。

多开会打架。你想开15个窗口并行干活，结果5个AI同时改同一个文件，Git冲突炸了，有人的工作白干。

你记不住谁在干啥。窗口一多，哪个负责登录、哪个负责加密，全乱了。

Steve的解决思路：让AI来管AI。

## 

Gas Town的结构

Steve用了个建筑工地的比喻来设计这套系统：

组件 干啥的 Mayor 接收你的需求，分配任务，完成后汇报 Polecats 临时工，干完就退，可以同时派很多个 Refinery 处理代码合并，排队一个一个来，避免冲突 Witness 每隔几分钟巡视一圈，发现卡住的推一把 Crew 正式员工，用来讨论架构这种需要长期对话的事 Deacon 每2分钟检查系统健康 Dogs 杂活，清理旧分支什么的

这些名字来自《疯狂的麦克斯》和其他影视作品。Gas Town本身就是《疯狂的麦克斯：狂暴之路》里那个炼油堡垒的名字。

## 

GUPP：怎么让AI不停下来

最大的问题是：AI会话结束后，新会话不知道要干啥。

Steve的解决方案叫GUPP（Gastown Universal Propulsion Principle），核心思想就一句话：如果你的Hook上有活，你就必须干。

Hook是什么？每个AI工人有个专属的任务钩子，存在Git里。会话结束了，Hook还在。新会话启动时自动检查Hook，发现有活就接着干。

这解决了"身份持久化"的问题。传统方式下，会话结束等于一切消失。Gas Town把AI身份和会话分开——身份是永久的工号，会话只是临时的执行者。

## 

MEOW：任务怎么组织

MEOW是Molecular Expression Of Work的缩写，就是任务组织的层级结构：

Bead：最小任务单元，一行JSON存在Git里

Epic：带子任务的Bead，子任务默认并行

Molecule：工作流，有执行顺序，一个接一个干

Formula：用TOML配置文件描述的工作流模板

其中Gate是个有意思的设计。很多工作有等待期，比如等CI跑完（可能要2小时）。传统做法是AI一直等着，浪费上下文。Gate的做法是：AI执行到等待步骤时主动休眠，进度保存到Git，等条件满足了系统再唤醒新AI接着干。

## 

实际运行情况

根据DoltHub的Tim Sehn的测试（他和Steve在Amazon共事过），用Gas Town跑了60分钟，花了大约100美元的Claude tokens，是普通Claude Code单位时间成本的10倍。

截至1月14日的数据：Gas Town从12月15日第一次commit到现在，已经有2684次commits，189k行Go代码。其中100多个PR来自近50个贡献者，加了44k行代码——Steve说这些代码没有人类看过。

Steve和Gene Kim合写了一本书叫《Vibe Coding》，已经出版。书里详细讲了这种"氛围编程"的方法论。

## 

使用门槛

Steve在文章里明确警告：大多数人还没准备好。

他把程序员的AI使用阶段分成8级：

- Stage 1-3：在IDE里用AI助手
    

- Stage 4-6：开始用CLI，多实例
    

- Stage 7：10+个agent，手动管理
    

- Stage 8：构建自己的编排系统
    

他说如果你不在Stage 7，别碰Gas Town。

其他注意事项：

- 烧钱。Steve自己用爆了两个Claude Code账号的额度
    

- 需要学tmux（终端复用器）
    

- 完全依赖Beads项目
    

- 工作方式会变得"流动"——有些工作会丢失，有些bug要修2-3次
    

## 

和Kubernetes的对比

Steve说Gas Town意外地很像Kubernetes：

方面 Kubernetes Gas Town 目标 让服务保持运行 让任务完成 工作单元 Pod Polecat 结束状态 维持运行 完成后销毁

两者都有控制平面、执行节点、本地代理、状态存储。核心区别是K8s让东西一直跑，Gas Town让工作干完。

## 

Steve的态度

他在博客结尾写：

"我不会回去上班，除非找到'懂'的公司。我厌倦了到处告诉人们未来，把它晃在他们面前，却没人相信。我宁愿坐在家里，亲手创造未来。我家有6种竹子，我已经是那只熊猫了。"

## 

核心概念速查

概念 一句话解释 Gas Town 多agent编排系统 GUPP Hook上有活就干 Hook 存在Git里的任务列表 Bead 最小任务单元 Molecule 有序工作流 Gate 智能等待机制 Polecat 临时工AI Refinery 合并排队员 Mayor 你的主接口

## 

链接

- Gas Town仓库：
    
    [https://github.com/steveyegge/gastown](https://github.com/steveyegge/gastown)
    

- Beads仓库：
    
    [https://github.com/steveyegge/beads](https://github.com/steveyegge/beads)
    

- Steve的Medium：
    
    [steve-yegge.medium.com](https://steve-yegge.medium.com/)
    

- 《Vibe Coding》：Gene Kim和Steve Yegge合著，IT Revolution出版
    

本文基于Steve Yegge的"Welcome to Gas Town"等博客文章整理