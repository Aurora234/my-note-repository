# 协同过滤



# 1994.GroupLens系统

[1] Resnick P , Iacovou N , Suchak M ,et al.GroupLens: An Open Architecture for Collaborative Filtering of Netnews[C]//Acm Conference on Computer Supported Cooperative Work.MIT Center for Coordination Science, 1994.DOI:10.1145/192844.192905.

## 背景知识

### Usenet系统

Usenet（User's Network，用户网络）是一个分布式的互联网讨论系统，起源于1979年，早于万维网（World Wide Web）和现代论坛的出现。它最初由杜克大学的研究生Tom Truscott和Jim Ellis设计，目的是在Unix系统之间通过UUCP（Unix-to-Unix Copy Protocol）交换消息和新闻。

#### 一、基本原理

Usenet的核心是**新闻组**（Newsgroups），类似于今天的在线论坛或子版块（subreddits）。每个新闻组专注于一个特定主题，例如科学、政治、娱乐、计算机等。用户可以在这些新闻组中发布消息（称为“帖子”或“文章”），也可以回复他人发布的消息，形成讨论串。

与集中式论坛不同，Usenet采用**分布式架构**：没有单一的服务器存储所有内容。相反，消息通过多个**新闻服务器**（news servers）之间相互复制（称为“转发”或“传播”），最终同步到全球网络中的其他服务器上。这种机制使得信息具有较强的抗审查性和容错能力。

#### 二、新闻组命名体系

Usenet使用层级式命名法组织新闻组，例如：

- `comp.*`：计算机相关话题（如 `comp.os.linux`）
- `sci.*`：科学话题（如 `sci.physics`）
- `rec.*`：休闲娱乐（如 `rec.music`）
- `soc.*`：社会文化（如 `soc.history`）
- `talk.*`：辩论性话题（常具争议性）
- `alt.*`：非正式、不受严格管理的另类新闻组（如 `alt.fan.pratchett`）

这些被称为“Big 8”层级（加上 `alt.*` 共九大类），构成了Usenet的主要分类体系。这些名字都是逻辑上的“主题路径”。

#### 三、技术协议

Usenet主要使用 **NNTP**（Network News Transfer Protocol，网络新闻传输协议）进行通信，该协议定义了客户端如何从新闻服务器获取文章、如何发布新文章，以及服务器之间如何同步内容。NNTP通常运行在TCP端口119（未加密）或563（SSL/TLS加密）上。

早期Usenet依赖UUCP通过电话线点对点传输数据，后来随着互联网普及，全面转向基于TCP/IP的NNTP。

| 特性       | 说明                                           |
| ---------- | ---------------------------------------------- |
| 分布式     | 无中心服务器，靠服务器间同步                   |
| 异步通信   | 用户不必同时在线即可参与讨论                   |
| 匿名性强   | 早期几乎无需注册，身份验证弱                   |
| 内容持久性 | 高保留期服务器可保存多年数据                   |
| 协议标准   | 基于NNTP，支持客户端（如Thunderbird、Newsbin） |