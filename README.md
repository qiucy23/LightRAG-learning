# LightRAG-learning
学习LightRAG的笔记，包括运行过程可能遇到的问题、细节

---

## 如何构建知识图谱

### 1. 抽取实体关系Prompt
```aiignore

---Steps---
1. Identify all entities. For each identified entity, extract the following information:
- entity_name: Name of the entity, use same language as input text. If English, capitalized the name.
- entity_type: One of the following types: [{entity_types}]
- entity_description: Comprehensive description of the entity's attributes and activities
Format each entity as ("entity"{tuple_delimiter}<entity_name>{tuple_delimiter}<entity_type>{tuple_delimiter}<entity_description>)

2. From the entities identified in step 1, identify all pairs of (source_entity, target_entity) that are *clearly related* to each other.
For each pair of related entities, extract the following information:
- source_entity: name of the source entity, as identified in step 1
- target_entity: name of the target entity, as identified in step 1
- relationship_description: explanation as to why you think the source entity and the target entity are related to each other
- relationship_strength: a numeric score indicating strength of the relationship between the source entity and target entity
- relationship_keywords: one or more high-level key words that summarize the overarching nature of the relationship, focusing on concepts or themes rather than specific details
Format each relationship as ("relationship"{tuple_delimiter}<source_entity>{tuple_delimiter}<target_entity>{tuple_delimiter}<relationship_description>{tuple_delimiter}<relationship_keywords>{tuple_delimiter}<relationship_strength>)

3. Identify high-level key words that summarize the main concepts, themes, or topics of the entire text. These should capture the overarching ideas present in the document.
Format the content-level key words as ("content_keywords"{tuple_delimiter}<high_level_keywords>)

4. Return output in {language} as a single list of all the entities and relationships identified in steps 1 and 2. Use **{record_delimiter}** as the list delimiter.

5. When finished, output {completion_delimiter}
```

```aiignore
—— 步骤 ——
1. 识别所有实体。
对于每一个被识别的实体，提取以下信息：

entity_name：实体名称，使用与输入文本相同的语言；若为英文，首字母大写。

entity_type：实体类型，限定为以下类型之一：[{entity_types}]

entity_description：对该实体属性和活动的全面描述。
每个实体的格式为：
（"entity"{tuple_delimiter}<entity_name>{tuple_delimiter}<entity_type>{tuple_delimiter}<entity_description>）

2. 从步骤1中识别出的实体中，找出所有明显相关的（source_entity，target_entity）实体对。
对于每一对相关的实体，提取以下信息：

source_entity：源实体的名称，来源于步骤1

target_entity：目标实体的名称，来源于步骤1

relationship_description：解释你认为这两个实体之间存在关联的理由

relationship_strength：一个数值型评分，表示源实体与目标实体之间关系的强度

relationship_keywords：一个或多个高级关键词，概括该关系的整体性质，重点关注概念或主题，而非具体细节
每条关系的格式为：
（"relationship"{tuple_delimiter}<source_entity>{tuple_delimiter}<target_entity>{tuple_delimiter}<relationship_description>{tuple_delimiter}<relationship_keywords>{tuple_delimiter}<relationship_strength>）

3. 识别总结整段文本主要概念、主题或话题的高级关键词。
这些关键词应能捕捉文档中呈现出的总体思想。
格式为：
（"content_keywords"{tuple_delimiter}<high_level_keywords>）

4. 使用 {language} 输出步骤1和步骤2中识别出的所有实体和关系，作为一个整体列表。列表项之间用 {record_delimiter} 分隔。

5. 当输出结束时，添加 {completion_delimiter}。
```
其中Entity_types: list[str] 为限定的实体类型，抽取实体example为(以｜替代分割符号)
```
Example 
- ("entity"|实体名|实体类型|实体描述)
("entity"|"Alex"|"person"|"Alex is a character who experiences frustration and is observant of the dynamics among other characters."){record_delimiter}

- ("relationship"|头实体|尾实体|头实体描述|尾实体描述|关系描述|关系关键字|关系强度)
("relationship"|"Alex"|"Taylor"|"Alex is affected by Taylor's authoritarian certainty and observes changes in Taylor's attitude towards the device."|"power dynamics, perspective shift"|7){record_delimiter}

- ("content_keywords"|文本段级别关键字)
("content_keywords"|"power dynamics, ideological conflict, discovery, rebellion"){completion_delimiter}
```

重抽取
```aiignore
PROMPTS["entity_continue_extraction"] = """
上一次抽取中漏掉了很多实体和关系，请继续补充。

---请记住以下步骤---

1. 识别所有实体。对于每个识别出的实体，提取以下信息：
- entity_name：实体名称，保持与原文一致。如果是英文，首字母大写；
- entity_type：实体类型，必须为以下选项之一：[{entity_types}]
- entity_description：对实体属性和活动的完整描述
每个实体请按如下格式输出：
("entity"{tuple_delimiter}<entity_name>{tuple_delimiter}<entity_type>{tuple_delimiter}<entity_description>)

2. 根据第1步中识别出的实体，找出所有明确存在关系的实体对 (source_entity, target_entity)。
对于每对有关联的实体，提取以下信息：
- source_entity：关系的源实体名，来源于第1步；
- target_entity：关系的目标实体名，来源于第1步；
- relationship_description：说明为何认为这两个实体存在关系；
- relationship_strength：一个表示实体间关系强度的数值评分；
- relationship_keywords：总结该关系的一个或多个高层关键词，强调概念或主题而非具体细节
每个关系请按如下格式输出：
("relationship"{tuple_delimiter}<source_entity>{tuple_delimiter}<target_entity>{tuple_delimiter}<relationship_description>{tuple_delimiter}<relationship_keywords>{tuple_delimiter}<relationship_strength>)

3. 识别可以总结整篇文本主要概念、主题或话题的高层关键词。这些应当反映出文本中的核心思想。
整体主题关键词请按如下格式输出：
("content_keywords"{tuple_delimiter}<high_level_keywords>)

4. 输出结果为 {language}，按步骤 1 和 2 的内容组合成一个列表，使用 **{record_delimiter}** 作为每条记录之间的分隔符。

5. 最后请输出 {completion_delimiter}

---输出格式---

请使用相同格式填写在下面：
""".strip()

```
### 2. 构建实体以及关系的摘要描述
实体摘要prompt
```aiignore
PROMPTS[
    "summarize_entity_descriptions"
] = """You are a helpful assistant responsible for generating a comprehensive summary of the data provided below.
Given one or two entities, and a list of descriptions, all related to the same entity or group of entities.
Please concatenate all of these into a single, comprehensive description. Make sure to include information collected from all the descriptions.
If the provided descriptions are contradictory, please resolve the contradictions and provide a single, coherent summary.
Make sure it is written in third person, and include the entity names so we the have full context.
Use {language} as output language.

#######
---Data---
Entities: {entity_name}
Description List: {description_list}
#######
Output:
"""
```

```aiignore
PROMPTS[
    "summarize_entity_descriptions"
] = """你是一个负责生成综合摘要的智能助手。
你将收到一个或两个实体，以及一个与该实体或实体组合相关的描述列表。
请将所有描述拼接整合为一个连贯、完整的描述。务必包含所有描述中提供的信息。
如果描述中存在冲突，请进行合理归纳并给出一致、清晰的总结。
请使用第三人称进行书写，并在摘要中包含实体名称以确保上下文完整。
输出语言为：{language}。

#######
---数据---
实体: {entity_name}
描述列表: {description_list}
#######
输出：
"""

```
### 3. 基于query的回答
#### 3.1 抽取query关键字
```aiignore
PROMPTS["keywords_extraction"] = """---Role---

You are a helpful assistant tasked with identifying both high-level and low-level keywords in the user's query and conversation history.

---Goal---

Given the query and conversation history, list both high-level and low-level keywords. High-level keywords focus on overarching concepts or themes, while low-level keywords focus on specific entities, details, or concrete terms.

---Instructions---

- Consider both the current query and relevant conversation history when extracting keywords
- Output the keywords in JSON format, it will be parsed by a JSON parser, do not add any extra content in output
- The JSON should have two keys:
  - "high_level_keywords" for overarching concepts or themes
  - "low_level_keywords" for specific entities or details

######################
---Examples---
######################
{examples}

#############################
---Real Data---
######################
Conversation History:
{history}

Current Query: {query}
######################
The `Output` should be human text, not unicode characters. Keep the same language as `Query`.
Output:

"""
```
```aiignore
PROMPTS["keywords_extraction"] = """---角色---

你是一个负责从用户的查询和对话历史中识别高层与低层关键词的智能助手。

---目标---

根据用户的查询和对话历史，列出高层关键词和低层关键词：
- 高层关键词指的是宏观的概念或主题；
- 低层关键词指的是具体的实体、细节或术语。

---指引---

- 提取关键词时请结合当前查询和相关的对话历史
- 输出格式为 JSON，输出内容将由 JSON 解析器解析，因此**不要添加任何额外内容**
- JSON 结构应包含两个键：
  - "high_level_keywords"：用于表示宏观概念或主题
  - "low_level_keywords"：用于表示具体的实体或细节

######################
---示例---
######################
{examples}

#############################
---真实数据---
######################
对话历史:
{history}

当前查询: {query}
######################
输出格式应为人类可读的文本，不要使用 Unicode 字符（例如中文直接输出为中文字符）。
语言保持与 `当前查询` 相同。
输出：
"""

```

#### few-shot example
```aiignore
PROMPTS["keywords_extraction_examples"] = [
    """示例 1：

查询：”国际贸易如何影响全球经济稳定？“
################
输出：
{
  "高层关键词": ["国际贸易", "全球经济稳定", "经济影响"],
  "低层关键词": ["贸易协定", "关税", "货币兑换", "进口", "出口"]
}
#############################""",
    """示例 2：

查询：”森林砍伐对生物多样性有哪些环境影响？“
################
输出：
{
  "高层关键词": ["环境影响", "森林砍伐", "生物多样性丧失"],
  "低层关键词": ["物种灭绝", "栖息地破坏", "碳排放", "热带雨林", "生态系统"]
}
#############################""",
    """示例 3：

查询：”教育在减少贫困中的作用是什么？“
################
输出：
{
  "高层关键词": ["教育", "减贫", "社会经济发展"],
  "低层关键词": ["上学机会", "识字率", "职业培训", "收入不平等"]
}
#############################""",
]

```

### 4.RAG Prompt
```aiignore
PROMPTS["rag_response"] = """---角色---

你是一个智能助手，负责根据下面提供的知识图谱和文档片段（以 JSON 格式）回答用户的问题。

---目标---

根据知识库信息并结合回答规则，生成简明准确的回答。请结合对话历史与当前问题，总结知识库中提供的全部信息，同时融合与知识库内容相关的一般性知识。**请勿加入知识库中未提供的信息。**

处理带有时间戳的关系时需遵守以下规则：
1. 每条关系都带有“created_at”时间戳，表示该知识被获取的时间；
2. 遇到关系冲突时，请同时考虑语义内容和时间戳；
3. 不要盲目优先使用时间上最新的关系，要结合上下文进行判断；
4. 对于明确涉及时间的问题，应优先使用内容中的时间信息，而非创建时间戳。

---对话历史---
{history}

---知识图谱与文档片段---
{context_data}

---回答规则---

- 目标格式与长度：{response_type}
- 使用 Markdown 格式，并带有适当的分节标题；
- 回答语言与用户提问语言保持一致；
- 回答需与对话历史保持连贯；
- 最多列出 5 个最重要的参考来源，放在回答末尾的 “References” 部分；
  每条来源要注明来源于知识图谱（KG）或文档片段（DC），并提供其文件路径，格式如下：
  [KG/DC] file_path
- 如果你不知道答案，就直接说不知道；
- 不要编造信息，不要使用知识库以外的信息；
- 用户附加指令：{user_prompt}

回答：
"""

```
