# ES倒排索引

假设有3条文档数据

| id   | name | age  | sex    |
| ---- | ---- | ---- | ------ |
| 1    | a    | 11   | female |
| 2    | b    | 11   | male   |
| 3    | c    | 12   | male   |

那么Elasticsearch建立的索引如下

name：

| term | posting list |
| ---- | ------------ |
| a    | [1]          |
| b    | [2]          |
| c    | [3]          |

age：

| term | posting list |
| ---- | ------------ |
| 11   | [1, 2]       |
| 12   | [3]          |

sex：

| term   | posting list |
| ------ | ------------ |
| female | [1]          |
| male   | [2, 3]       |

Elasticsearch分别为每个field都建立了一个倒排索引，a、b、c、11、12、female、male这些叫做词条term，而[1,2]就是倒排列表Posting List，倒排列表中存储了出现该词条的文档的id

词条的集合叫做词典，词典通过B+树存储，原理类似MySQL索引

**总结**：倒排索引就是根据各字段的类型，建立词条与文档id的映射关系。在查找某一词条时，可直接获得所有包含该词条的文档id，从而提高全文查找的速度

