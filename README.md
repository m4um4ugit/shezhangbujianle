借助世界上最大的单体中文NLP大模型(浪潮 源1.0 https://air.inspur.com/home ），我们做出了一个可以跟人类玩“剧本杀”的AI……
# 无图无真相
我们为本项目特别改编了一个微型线上剧本杀剧本，体验者将参加一场五人的线上剧本杀，但这里面的一人其实是我们的AI。

为了尽量避免引起玩家怀疑，我们不想让用户额外下载APP或者登录一个网页。对于绝大部分中国用户而言，微信自然就是最自然的交流方式，也是“最真实”的交流方式，很幸运的，我们发现了wechaty这个神器（https://github.com/wechaty ），借助它，我们打造了一个“微信虚拟人”。

（从账号昵称到头像，都是“编导组”成员精心策划的，我们甚至还细心的为她准备了三天的朋友圈内容）

整体剧情并不复杂，讲的是某高校社团中，五个骨干成员因为一件事情牵涉到各自利益而产生的种种勾心斗角。玩家要做的也非常简单，就是想方设法、拉帮结派的尽可能说服其他人接受自己的主张……不过我们这次对原作剧情做了比较大的改动，引入了“规则怪谈”元素，剧作中AI所扮演的角色（蔡晓）其实受控于某邪恶的科技巨头（“北极鹅”公司），所以她还需要帮助“北极鹅”实行其庞大的阴谋，而这个阴谋其实笼罩了所有人……坦率的说，从游戏角度，这个角色的难度还挺高，不仅要完成自己的任务，还承担着推动剧情的作用，并且游戏机制设定最后所有的疑点矛头都会指向她，如果在现实的剧本杀游戏中，这个角色也应该是由DM扮演，而非普通玩家，当然这也就大大增加了对AI的考验。

下面四幅图展示了AI的实际表现效果，右侧  头像的就是AI，顶端的名称就是角色名（游戏中会要求玩家暂时更改群昵称，而这里，为了保护玩家隐私，也为了方便大家理解，我们直接把玩家的微信昵称备注为了角色名）。

### 谭明VS蔡晓（AI）
剧情中谭明策划了一个阴谋，并计划私下与蔡晓达成联盟，然而蔡晓其实背着他还有一个更大的阴谋……我们为AI制定的针对谭明的策略就是可劲儿的忽悠他并想方设法利用他。实际表现中，AI很好的贯彻了这个思路，甚至发挥想象力的使用了色诱绝技……坦率的讲，这招也极大的超出了我们的预料……
![img](https://github.com/WukongZeming/shezhangbujianle/blob/main/assets/%E5%B1%8F%E5%B9%95%E5%BD%95%E5%88%B62022-03-13%2014.48.44.gif)

### 孔墨VS蔡晓（AI）
### 李超VS蔡晓（AI）
### 孙若VS蔡晓（AI）
### 蔡晓（AI）在公聊（房间）

# 核心功能——“目的性对话”端到端生成方案
“剧本杀”本身就是一种用户与用户之间充分博弈的游戏，玩家之间的对话可能更是无穷无尽，这种情况下使得传统的规则式或“词槽式”“目的域对话”方案均行不通，同时因为玩家之间的对话都是带有目的性的，尤其是AI所扮演的这个角色还承担着部分推动剧情的DM功能，所以“开放域”对话也不适合。

本项目所使用的NLP大模型浪潮源1.0是一种生成式预训练模型（GPT），其使用的模型结构是Language Model（LM），类似于openAI的GPT-3，但是与GPT-3不同，源1.0更加擅长的是零样本（Zero-Shot）和小样本（Few-Shot）学习，而非目前更多模型所擅长的微调试学习（finetune）。从实际应用效果来看也确实如此，在2~3个，甚至1个合适example的示范下，模型可以很好的理解我们希望实现的“对话策略”，仿佛具有“举一反三”的能力，但是如果没有example的话，那么模型的生成则非常不靠谱，甚至会出现答非所问的情况。因此，本项目的关键就在于如何针对用户的提问选择适当的example供给模型。

最终我们采取的方案是：建立example语料库，然后针对每次提问从语料库中选择最贴近的三个example作为模型本次生成的few-shots。

实际实现中，因为除AI扮演的角色外游戏还有四个角色，而每个角色针对AI角色的提问又有两种可能：1、私聊中提问，2、公聊（群）里面的@提问，所以语料库被分装成8个TXT文件，程序会根据提问者以及问题来源（私聊还是公聊）去对应选择语料来源。这个机制的思路很简单，但是执行起来马上遇到的一个问题就是，如何从对应语料中抽取与当前提问最为相似的example？因为在实际游戏中，玩家可能的提问会很多，为了照顾到各种情况，我们准备的语料库也会涵盖很多问题（截止本项目发布时，8个语料库文件最少的问题量也达到了  个），显然不可能把这些问题都作为example在每次生成时输给模型。在这里我们用到了百度飞桨PaddlePaddle发布的预训练模型——simnet_bow，它能够自动计算短语相似度（https://www.paddlepaddle.org.cn/hubdetail?name=simnet_bow&en_category=SemanticModel ），基于百度海量搜索数据预训练，实际应用下来效果非常不错
