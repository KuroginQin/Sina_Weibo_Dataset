0.Brief Description
    A small-scale data set of Sina Weibo, collected by a simple web crawler program. The data set contains the topology structure and content information of about 2,000 Sina Weibo users.
    For the convenience of conducting Community Detection [1] experiments by using the data set, we also ran the Louvain algorithm [2] to determine the number of communities and the community membership of the network. The details about the data set will be introduced later.
    Due to my limited ability, it’s hard to avoid some errors and deficiencies in the data set. If you find anything that can be improved, you can contact me via [mengqin_az@foxmail.com].

0.简述
一个通过网络爬虫获取的小规模新浪微博数据集，包含2000个用户的拓扑关系和内容信息。
为了方便使用该数据集进行社团发现[1]的实验，我们还运行Louvain算法[2]确定了网络的社团数和社团成员，关于数据集的细节在本文档的后面介绍。
由于个人水平有限，难免有疏漏之处，还望大家批评指正！关于数据集的任何问题，可通过邮件[mengqin_az@foxmail.com]联系，谢谢！

1.The Crawler Program
Under the framework of Multi-thread Crawling and Simulated login implemented by WebCollector [3], we wrote a simple Web Crawler program to crawl data from Mobile Web version of Sina Weibo (weibo.cn), and finally collected such data set.
Different from the PC web version of Sina Weibo (weibo.com), the Mobile web version (weibo.cn) doesn’t have the mechanism of dynamic loading, and it can at most display 10 weibos in a page. Although the PC web version (weibo.com) can display more weibos, the content of new weibos will be loaded only when the cursor moves to the bottom of current page.
The PC web version (weibo.com) may also have limits for users to visit the followee list or the fan list of a certain user. Especially, for the users who have a large number of followees or fans, one can’t visit the corresponding pages of the followee list or the fan list. While the mobile web version (weib0.cn) doesn’t have such limit. One can visit at most 10 pages of the list of followees or fans, and there are at most 10 users displayed on each page of such two lists.
Sina Weibo is a Social Network Platform that the users will be authorized to view the main pages, the followee lists or the fan lists of other users, only when they complete the login. In general, we can operate a process of simulated login, by submitting the cookie of a certain user account.
However, there remain strict limits on web crawler in Sina Weibo. If we use an account to visit related pages too frequently (in order to collect a certain scale of data), the server may “freeze” the account. Under such circumstance, the users need to login on the PC web version (weibo.com), and send SMS according to the prompt to finish the process of “unfreezing”. More importantly, one mobile phone number can only “refreeze” limited number of Sina Weibo accounts.
To overcome the limit discussed above, we registered totally 5 accounts of Sina Weibo. For each account, we first logined on the Chrome browser, and got the corresponding cookie by using the debug function of Chrome. We repeated such process, and finally save the cookies of such 5 different accounts.
The data we collected contains the topological structure and the content information. In general, the topological structure infers the relationship of following and being followed among users. Each user can be seen as a node in the network, and the two types of relationships can be seen as the directed edges. And the content information contains the weibos, the personal information and comments posted by users.
So as to reduce the risk of being “frozen”, while collecting data as much as possible, we used a strategy of simplified forward-link crawling [4]. For each user, we only crawl the first 5 pages of his/her followee list with totally at most 50 user he/her follows, but give up considering his/her fan list. On the other hand, for the content information, we only consider the content of weibo or the reason of retweeting posted by him/her. And we only crawled the content of the first 5 pages of the user’s main page. What’s more, in order to avoid being recognize as Crawling Behavior by the server, everytime the crawler finished the process of crawling a user’s topological structure or content information, we would force current thread to sleep for a random time.
For the whole process of crawling, we used the strategy of BFS (Breadth First Search) according to the users’ relationship of following. First, we set the account whose nickname is “Peking University” as the root user, and then crawled its first 5 pages of weibo content and first 5 pages of followees. We also maintain a FIFO (First in First out) queue for the follow relationship, and pushed the root user’s followees into the queue (according to its followee list). Next, we popped the first user in the queue, and took it as current user. Continuously, we repeated the process introduced above to crawl the topological structure and content information of current user and maintain the queue of follow relationship, until the queue was empty or the totally number of users exceeded the threshold set in advance.
In addition, for the content information, we also implemented related preprocess while crawling the content text. First, we used the word segmentation tools provided by Ansj to segment words form the content text. Then, we deleted the stop word in the result by using a stop word list, and we also used some extra rules to deleted some abnormal words (such as URL, lengthy numbers). Eventually, we generated the keyword representation of content information.

1.爬虫程序 
我们使用一个简单的新浪微博爬虫程序获取数据，该爬虫借助WebCollector提供的多线程爬取和模拟登陆的框架，从新浪微博的手机版网页(weibo.cn)上爬取数据。
与新浪微博PC版网页(weibo.com)不同，手机版网页(wiebo.cn)没有动态加载机制，一页最多能显示10条完整的微博；而PC版一页虽然能够显示更多的微博，但只有光标移动到底部才会加载新的微博。
PC版(weibo.com)对某些用户的关注和粉丝列表的访问做了限制，对于关注数或粉丝数过多的用户，不能查看他们的关注或粉丝列表；而手机网页版(weibo.cn)没有这个限制，能够查看某用户10页的粉丝或关注列表，其中每个页面最多有10个用户。
新浪微博是一种需要完成用户登录才能访问其他用户的主页、关注列表和粉丝列表等的社交网络平台。一般的网络爬虫均采用提交Cookie的方式，达到模拟登陆的目的。
而新浪微博对于网络爬虫也具有比较严格的限制，当某个账号频繁访问页面(为了爬取一定规模的数据)时，服务器将会冻结该账号。在这种情况下，需要手动登录PC版新浪微博 (weibo.com)，按照提示发送手机短信，完成“解冻”操作；而且，一个手机号码在一段时间内只能为有限个账号“解冻”。
为了克服服务器对于爬虫程序的限制，我们特别注册了5个爬取用的新浪微博账号。对于每个账号，我们首先在Chrome浏览器上正常登陆，接着使用浏览器的调试功能，获取账号对应的Cookie字符串；重复这个步骤，我们在本地保存了5个不同账号对应的Cookie。
爬取的数据包括用户的拓扑结构和内容信息。一般情况下，拓扑结构主要指用户之间的关注与被关注的关系；可将用户视为网络中的节点，关注和被关注关系视为网络中的有向边。而内容信息主要包括用户发布的微博内容、个人信息、评论等。
为了减少账号被冻结的风险，并爬取尽可能多的用户数据，这里我们采用了一种简化的前向爬取的策略，即对于每个用户，我们只爬取关注列表的前5页，共50个关注用户；而不考虑该用户的粉丝列表。对于用户的内容信息，我们只考虑用户发布的微博中的内容，或者转发微博时提交的转发理由，并且只爬取用户主页前5页的微博内容。进一步地，为了尽可能避免被服务器捕捉到爬虫欣慰，在每爬取完成一个用户的拓扑结果或内容信息时，我们会强制当前爬取线程随机休眠一段时间。
对于整个爬取过程，我们根据用户的关注关系进行广度优先遍历策略。首先，我们设置“北京大学”的新浪微博(id:)为种子用户，爬取其主页前5页的微博内容，以及关注列表的前5列，同时维护一个“关注”关系的先进先出(FIFO)队列，并将种子的关注关系压入队列中。接着，从队列头部的关注关系处队列，取关注关系中的目标用户作为当前用户，通过同样的方法爬取当前用户的拓扑结果和内容信息，并维护关注关系队列。整个过程不断重复，直到队列为空，或爬取的用户数超过了预先设置的阈值。
此外，对于用户的内容信息，我们在爬取内容文本的同时也执行了相关的预处理操作。首先，我们使用Ansj的分词工具，对内容文本进行分析。然后，我们对照停用词词表去除分词结果中的停用词，并加入一些额外的规则去除异常词(如URL、冗长的数字编号等)。最后生成了内容信息的关键词表示。

2.The data set
    The data set contains the topological structure and the content information of the network, in which the topology is represented in the way of directed edges, and each edge is represented as a tuple (Source Node Index, Target Node Index). And the content is represented in the way of users’ ownership of keywords. Here, we also used a tuple (User Index, Keyword Index) to represent such ownership.
    The details of the data set are as follows:
 *Number of nodes: 2,000
 *Number of edges: 2,527
 *Number of keywords: 124,742
 **Number of communities: 28
 **Maximum Modularity: 0.8018
    The formats of the data file are as follows:
*user_list.txt: the user list. 
The i-th line represents the user with index i-1 (the user index starts form 0), and there is a tab-delimited between each userId and UserNickname.
UserId	UserNickname

*topo.txt: the topology structure. 
There is a tab-delimited between each source user index and target user index.
sourceUserIndex	targetUserIndex

*content.txt: the content information represented as text. 
“@@Content” represents the beginning of the content, and “@@ContentEnd” represents the end of the content.
@@UserIndex:
@@Content:
…
@@ContentEnd

*dict.txt: the dictionary.
The i-th line represents the keyword with index i-1 (the keyword index starts from 0).

*content_index.txt: the content information represented in the index format.
There is a tab-delimited between the userIndex and the wordIndex.
userIndex		wordIndex

*community_membershp.txt: the community membership.
The i-th line represents the community labels of the user with index i-1 (the user index and the community label both start from 0).

2.数据集 The data set
    数据集包含网络的拓扑结构和内容信息，其中拓扑结构通过用户之间的有向边表示，这里我们使用二元组(源节点索引, 目标节点索引)表示一条边；而内容信息通过用户与关键词的所有权关系表示，我们同样使用二元组(用户索引, 键词索引)来表示这种所有权关系。
此外，为了评估社团(Community)结构，我们还针对网络的拓扑结构运行Louvain算法，通过层次聚类的方式，得到了对于网络社团数和模块度的估计，并生成了对应的社团成员关系。
    数据的详细参数如下所示：
*节点总数： 2,000
*边总数： 2,527
*关键词总数： 124,742
**社团总数： 28
**最大模块度： 0.8018

数据文件的格式如下所示：
*user_list.txt: 用户列表，第i行表示索引为(i-1)的用户(用户索引从0 )，用户id与用户昵称之间以Tab分隔符分隔开。
UserId	UserNickname

*topo.txt: 拓扑结构，源用户id与目标用户id之间以Tab分隔符隔开。
sourceUserIndex	targetUserIndex

*content.txt: 以完整文本表示的内容信息，“@@Content”表示内容文本的开始，“@@ContentEnd”表示内容文本的结束。
@@UserIndex:
@@Content:
…
@@ContentEnd

*dict.txt: 字典，第i行表示索引为(i-1)的关键词(关键词索引从0开始)。

*content_index.txt: 以索引表示的内容信息，用户索引与关键词索引之间以Tab分隔符隔开。
userIndex		wordIndex

*community_membershp.txt: 社团成员，第i行表示索引为i-1的用户的社团标签(用户索引和社团标签均从0开始)

5.Reference
[1] M. Girvan, and M. E. J. Newman. Community structure in social and biological networks. Proceedings of the National Academy of Sciences of the United States of America, vol. 99, pp. 7821-6, Dec. 2001.
[2] V.D. Blondel, J.L. Guillaume, R. Lambiotte, and E. Lefebvre. Fast unfolding of communities in large networks. Journal of Statistical Mechanics: Theory and Experiment, 2008(10):P10008, 2008.
[3] WebCollector. http://www.oschina.net/p/webcollector?fromerr=BYiMk3KP
[4] A. Mislove, M. Marcon, K. P. Gummadi, P. Druschel, and B. Bhattacharjee, “Measurement and analysis of online social networks,” in Proceedings of Internet Measurement Conference, 2007.
[5]Ansj. https://github.com/NLPchina/ansj_seg
