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