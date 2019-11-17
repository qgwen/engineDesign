#### B+tree 索引
B+tre索引在数据库中的实现具有高扇出性，一般在数据库中B+Tree的高度为2-4层，在InnoDB存储引擎中，分为聚集索引和辅助索引，这两种索引内部都是B+tree的数据结构  

#### 聚集索引
1、每张表只存在一棵聚集索引  
2、根据表的主键构造的  
3、叶子节点是数据页，数据页中存放的完整的行数据  
4、非叶子节点是索引页，存放的主键的数据  
5、每个数据页都是通过双向链表链接的

#### 辅助索引  
1、每张表存在多棵辅助索引  
2、根据创建索引时的字段构造的  
3、叶子节点是数据页，存放的是构造索引的字段和对应的主键的值  
4、非叶子节点是索引页，存放的是字段的值  
5、索引的构造是按照创建索引时指定的顺序  

#### 全文索引（full text index）
全文索引基于倒排索引来实现的，mysql5.7之前不支持中文的全文索引，mysql5.7内置了ngram全文检索插件，用来支持中文检索。  
例如：  
CREATE TABLE articles (  
id INTUNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,  
title VARCHAR(200),  
body TEXT,  
FULLTEXT (title,body) WITH PARSER ngram  
) ENGINE=InnoDB CHARACTER SET utf8mb4;  


