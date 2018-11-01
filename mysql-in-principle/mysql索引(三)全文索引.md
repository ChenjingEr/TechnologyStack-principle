## mysql索引(三)： 全文索引
>全文索引是对一个大文本中的内容进行索引，便于查找关键字等信息。

**索引实现:**
全文索引使用倒排(inverted index)索引实现。在辅助表中存储单词与文档/文章的映射关系。通常利用关联数组实现，表现形式：
    
        inverted file index:{单词，文章ID}
        full inverted index : {单词，(文章ID，位置信息)}