# 本文档记录了搭建一个网页系统业务对话式agent的搭建记录:  
## 阶段1：
1.一开始跟AI聊了需求，AI的建议是先用本地代码做对话，即通过匹配关键字，使用计分制判断本地资料库与这个问题匹配度，从而回答这个问题。因为是AI的推荐，为了搞清他推荐得原因和原理，于是让他生成了个初版看看效果。  
      结果：  
      经常性答非所问，或者过度回答。 比如我问关键词你们公司防火墙项目主要是做什么的，回答：我们防火墙业务是xxx（正确回答），之后接了个无关意义的关系，例：防火墙业务是用xxxx。分布式框架的业务是用来xxxx。
      调查了下思维逻辑发现，记分制是通过正则匹配关键词计分，但是有时候从自然语言提取出来的关键词过于宽泛，比如上面的例子，知识库中举了几个公司已有的项目，但是计分时候将公司作为关键词提了出来，而每个项目在知识库都有
      提及是本公司的项目，于是乎很多不相关的知识点也因为这种宽泛关键词导致的加分而作为了回答返回给了前端。后续加了一些限制，如二次分析结果关联性，增加业务关键词的分数，抹除通用宽泛词的分数，之后效果基本达到，但是
      不符合我的需求，且过于局限性，遂放弃。

2.第二次换了个项目从0开始，这次定义希望支持简单的查，尽量本地分析生成以节省token，若本地无法处理即与远端大模型交互并获取方案，其中关于查询的SQL允许本地把需求和简单的数据库模型发给远端LLM，再生成SQL后本地做二次验证在执行。 
  到此主要是想把基础架子搭起来，做到本地已知的直接本地处理减少额外开支，本地无法处理的，比如自然语言过于模糊，语意不清等情况，发给远端，为了数据安全发出去的仅包含需求和表结构，以允许远端LLM生成对应SQL， 
  为了安全本地还是会做二次验证防止修改类语句破坏数据库（insert/delete/drop...）  
      结果：  
      开始时对说的清楚的问题回答准确率基本100%，但是实际上看LLM API请求基本为0，检查了log和代码结构发现，AI生成了大量正则表达式和关键词，现有的问法的确都cover住了，但是就是觉得怪怪的。。甚至在怀疑这是否算是一种AI Agent
      <img width="1488" height="445" alt="c61013a6a7a090d924670585ed241d39" src="https://github.com/user-attachments/assets/2077964a-64c4-4597-ab40-5f483216e4f4" />  
      Ai给出的回答是：  
      按咱们这份方案：业务CRUD的主路径是 规则意图 + 本地解析 + 固定写库确认，LLM 只是兜底，所以这一块更像：「带 AI 能力的业务助手 / Copilot」，而不是「端到端自主 Agent」。
      不知道是我表达有问题还是怎么，但是由于这一块足够稳健，打算先放在这，摸索出好的方案之后再修改这一块。
      
 3.因为有了解langcharin和langGraph，langCharin偏向线性单次任务，Graph则支持循环多次思考给出结果，现在我打算针对上面这个业务允许用自然语言生成增删查的选项，于是尝试引入langcharin试试单次线性任务，考虑的是现在的需求来看，
   暂时没有需要Graph的场景，没必要特意去使用。 项目本身是Java的，有一个LangCharin4J框架可以直接使用，但是查询后发现这个算是个私人项目，虽然获得langcharin官方认可，但是考虑稳定性，并且因为有一点python能力，还是选择原生
   的比较好，遂打算使用Lnagcharin+fastapi的组合，即本地java还是做基本判断与处理，如果遇到无法处理需要调用LLM时候，通过API调用langcharin，从而调用LLM API发送问题获取结果。  
       结果：  
       由于没有特殊要求，系统又写了一大堆关键词和正则（参考上面图片），只有上面关键词匹配不上在调用llm，还是固定语义可用，看起来非常的奇怪。  
       3.1 由于上面的结果，已经对智能助手的本意产生疑问了，于是先研究下是方向问题还是表达问题，于是我按照一个请求调用的方法顺序从头理了下。  
           首先是一个语言进来后于一大堆正则和关键词做匹配，研究后发现这部分实际上做了分流+拦截的操作。快速找到对应的业务方法或者决定是否调用Charin和LLM。这其中还涉及到正常agent模型中缓存机制，关键词实际上就是第一层缓存机制，
           这部分之后会着重研究下，但是证明方向和表达没问题，AI是根据最佳实践在处理。  
           其次是看了代码发现代码在本地建了很多MD作为知识库，这部分到时没什么问题，业务尽量放在本地，结合上面分流，快速定位这个自然语言想要做的业务方向。但是这其中还是使用了关联记分制，这个因为上面有一次不好的体验于是
           着重查了下原因。首先资料显示现在主流得方案有：关键词打分（我称之为计分制），向量数据库，全文检索，结构化路由和知识图谱。 其中向量数据库我是理解的，现阶段数据量很少，而且向量数据库需要额外的文件，没有选择合情合理
           可以忽略。 其他的几个中，**全文检索**更像是关键词检索的加强版，本质还是在做关键词匹配并打分，区别是面对很大量的文档，他结合了索引的概念，先做了个索引，之后做关键词匹配打分。现在的做法看起来就像是轻量的全文索引.
           **知识图谱**，这个更像是ER图的思想，我把关系先梳理出来，之后通过关键词找到每一个关系，从而推断出这个问题可能的答案，适合做推力和校验。不过暂时不适合本项目，因为本项目是已有的数据基础上，做确切的回答和计算，不需要
           太多的推理思考，比如我需要查具体的，修改什么。这些都不是我们通过碎片信息推理猜测的。当然要做也是可以的，但是你需要维护另一套模型成本，这这个关系实际上数据库也已经有了，所以并没有太大必要。最后一个是**结构化路由**，
           这个比较有意思的是项目实际上也用到了，一个项目可以用多个方法我的确没想到，毕竟上面几个方法看起来要不是包含要不是互斥。说回这个他适合固定流程性的工作，你可以理解为很多经常性使用的功能，比如作为一个仓储管理系统，用的
           最多的无非是查数量，统计数量等，这些常用的方法用ifelse拼装起来，就是结构化路由的大概意思。    
      行吧，这一顿研究后发现AI也没忽悠我，那至少现阶段方向看起来没问题。不过我突然意识到每一个业务都这么写，实际上很麻烦，虽然完全可行，但是作为一个人，用AI就是因为懒。程序员的目标不就是为了懒而创造工具让自己或帮助别人更懒，
      而且像大模型语言他也不可能每个业务都细致研究，但是他也能回答我问题，那么就说明应该有一个办法可以做通用的分析和计算，而不是完全依赖知识库。在这个基础上于是新建了个branch，打算打开新世界大门。  

  4.再亲切友好带一些强迫性质的交流后，AI给了我一个解决方案，已有的部分先不动，再此基础上增加一个通用业务Agent，不过这其中因为我想要强化LLM的使用并且想看看一天token使用量，我让他新增了个变量来确保每个问题是否都强制走LLM：  
    1.增强本地业务知识库    
    2.强化现有sql查询路径，尝试优先使用LLM转义生成SQL。  
    3.通过Mutation框架强化变更路径  
    4.强化意图路由  
    什么乱七八糟的，我的仔细研究研究这些新的名词，不过我远端服务器跳闸断电了，管理员（伪）觉得断电了不适合上班吧服务器启动就跑路了，殊不知虚拟机没设置上电自启，于是乎剩下的明天再看吧。


## 阶段2：
那天晚上思来想去还是觉得这个Agent并不是我期待的智能化，如果什么都通过知识库，只有生成SQL等才去远端那跟本地一堆ifelse有撒谎区别呢，于是跟群里大佬探讨了下， 跟[chaleaoch](https://chaleaoch.com)和wiloon一顿激情探讨后，最后给我share了一个link：  
https://www.anthropic.com/engineering/building-effective-agents  
里面有句话我觉得到时点醒了我:  

> What are agents?
"Agent" can be defined in several ways. Some customers define agents as fully autonomous systems that operate independently over extended periods, using various tools to accomplish complex tasks. Others use the term to describe more prescriptive implementations that follow predefined workflows. At Anthropic, we categorize all these variations as agentic systems, but draw an important architectural distinction between workflows and agents:  
**Workflows** are systems where LLMs and tools are orchestrated through predefined code paths.  
**Agents**, on the other hand, are systems where LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks.  
Below, we will explore both types of agentic systems in detail. In Appendix 1 (“Agents in Practice”), we describe two domains where customers have found particular value in using these kinds of systems.  

我的需求是不想死板的定义一堆ifelse，而是给一些关键词，AI根据关键词做自行判断，这样本地hardcode部分少，虽然可能会引起一定的幻觉，但是这个可以之后慢慢限制词处理。但是这样做之前做的就都应该定义为workflows，即代码引导到不同分支，
这并不是我一开始想做的，我怀疑是一开始给的限制条件如希望减少token消耗，数据本地不出环境导致的。不过已有的部分我打算先保留，之后下一部分业务试试纯Agent开发方向。即通过LLM动态的分析自然语言，找到目标业务，再继续往下执行,一次一个任务
，自己思考，调用，思考。  
方向定好后，那么应该是逃不掉langGraph了，架构就变成了：  
LangGraph 负责调度  
  - chatbot 节点：用 LangChain 调 LLM，让模型自己选工具  
  - tools 节点：用 LangChain 的 ToolNode 执行模型选中的工具  
  - 再回到 chatbot …  

由此又带出来一个新的关键词：ReAct /tool-calling Agent，他俩到是我之前想要的那种Agent的具体体现:  
ReAct,即Reason(think how) +Action(use toole/do sth) 它是一种思维方式  
Tool-calling Agent则是把这个思维方式落地的方法：   
用户消息  
  - LLM（已 bind_tools，知道有哪些工具）  
  - 若返回 tool_calls：执行工具，把结果写成 ToolMessage  
  - 再喂给 LLM  
  - 直到 LLM 只返回普通文本（不再要工具）→ 最终答案  

挺好，这么下来至少是串起来了，而且符合我的预期，使用的技术和设计方案也是我预期的，看一下效果：  

<img width="519" height="901" alt="image" src="https://github.com/user-attachments/assets/50a45a31-e9b8-45d4-94b3-82e99eeac2d7" />

这样下来为了做这个Agent整个项目的改动有：  
新的python项目（Agent文件夹）:  
  - FastAPI,提供restapi接口允许Java调用LLM等  
  - LangChain:  
    - tools.py 定义了agent的可以获得哪种能力，我觉得就是大家一直说的agent有哪些skills，比如我之前强调了不允许agent直接读数据库或者写库，所以目录下有一个定义就是当需要查询时候，调用XXX方法，我看了下方法就是调用了spring的api
    来查询数据库，从而规避一些问题,使得对数据库的操作都受控。  
    - 消息结构：SystemMessage、HumanMessage、AIMessage、ToolMessage, 主要记录一次 Agent 运行中的对话状态，最后组装成一起发给大模型,包括系统规则，用户输入，模型输入（包括答案和tool call），工具执行结果  
    - 统一大模型封装, langchain_openai.ChatOpenAI, 这没啥好说的，统一接口  
    - Tool Calling / Function Calling， 给AI注册有哪些工具（skill），这里bind_tools就是上面tools.py里面定义的功能  
  - LangGraph： 没有用太多高技术含量的东西,主要是状态机任务编排，循环（即是否需要下一步，是否需要调用tool，还是答案以足够结束循环）  
    -  StateGraph 定义 Agent 执行流程  
    -  ToolNode 执行模型发起的工具调用  
    -  START/END 控制流程开始结束  
    -  条件边判断模型是否还需要继续调用工具  
    -  collect 节点收集工具 evidence  


- 旧的Spring项目（前后端混合）,总结一下主要是负责安全和提供接口，在加上前端页面:  
  -  新增一个agent页面  
  -  代理服务，调用python的agent的接口,传递用户的信息  
  -  暴露了一些API，比如读写DB，目的是允许agent根据判断执行一些功能，又担心agent权限过高，于是用接口的方式使得这部分可控  
  -  鉴权，禁止用户直接使用Agent API，通过每次请求带上特定key来允许调用。  
  -  Agent权限配置等  
  -  增加了一些业务的信息和MD，因为一开始agent做的是工作流模式，他依赖本地文档，这部分保留因为针对某个业务，MD文件足够丰富，足够驱动一些问题答案，如果不够还有LLM兜底，变相节省token。但是本地信息也容易出现误导导致的错误直接返回
  结果而不通过LLM二次判断。这个需要考虑下怎么处理。  
  - api-registry.yml注册表 集中统一记录了当需要增啥改查时候需要的信息，内部定义了每一个业务的名称，该名称下有关键词create/update/delete，这样代码只需要确定业务和操作，直接读取对应关键词下的内容操作即可，否则需要写大量的IF/else
  来驱动不同业务的增删改该怎么操作。示例如图:  
  <img width="203" height="520" alt="image" src="https://github.com/user-attachments/assets/52c97717-6fc9-49f9-8391-e87278c3980a" />

  
另：
  LangChain有几个我知道比较著名但是没用上的功能：  
  1.LangChain Memory 上下文记忆，这里AI自己写了个小的记忆体，给出的解释是业务过于简单，没必要上这么重的上下文监控,不需要复杂的上下文记忆。 这个的确是我限制的，因为业务主要是快速分析生成报表结论什么的，不需要记昨天甚至上周的内容。  
  2.本地向量数据库，同上，内容和业务没有那么复杂， 简单的MD足以，硬上向量除了多一个服务没啥优势。  

  

