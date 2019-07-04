---
title: elasticsearch
date: 2018-01-25
tags: ElasticSearch
categories: ELASTICSEARCH
---
####一些概念
######1. node: 节点，
	节点类型：
		master node:  创建/删除索引，集群管理，分区管理
			node.master: true
			node.data: false
			node.ingest: false
		data node: 存放索引文件，处理数据相关的操作（CRUD, 搜索，聚合）
			node.master: false
			node.data: true
			node.ingest: false
		ingest node：预处理节点，在索引之前预处理文档;在批量请求或索引操作之前，ingest节点拦截请求，并对文档进行处理。
			node.master: false
			node.data: false
			node.ingest: true
		coordinating only node：只负责转发请求到其他节点
			node.master: false
			node.data: false
			node.ingest: false
<!--more-->
######2. index: 索引
######3. type: 从es6.0, 一个index下面只允许有一个type
######4. document： 文档
######5. shards 分区
######6. replica： 副本    ElasticSearch默认设置是5个shards和一个replica
######7. cluster： 集群
######8. tokenizer    分词器，对文本进行分词
######9. filter: token处理器 Token filters accept a stream of tokens from a tokenizer and can modify tokens (eg lowercasing), delete tokens (eg remove stopwords) or add tokens (eg synonyms)
######10. analyzer: 一个analyzer可以包含多个tokenizer和多个filter


####API（kibana console）
######1. 检查集群状态    GET /_cat/health?v
######2. 检查节点    GET /_cat/nodes?v
######3. 获取所有索引     GET /_cat/indices?v
######4. 创建单个文档

	PUT /index/type/document_id/_create
	{
		"name": "ceshi"
	}
	
	成功返回： 201 created
	{
	    "_index":    "website",
	    "_type":     "blog",
	    "_id":       "123",
	    "_version":  1,
	    "created":   true
	 }
	错误返回
	如果文档已存在，返回409 conflict
	{
	    "error": {
	       "root_cause": [
	          {
	             "type": "document_already_exists_exception",
	             "reason": "[blog][123]: document already exists",
	             "shard": "0",
	             "index": "website"
	          }
	       ],
	       "type": "document_already_exists_exception",
	       "reason": "[blog][123]: document already exists",
	       "shard": "0",
	       "index": "website"
	    },
	    "status": 409
	 }

######5. 获取文档
	GET /index/type/document_id?pretty
	成功返回：
	200 ok
	{
	   "_index" :   "website",
	   "_type" :    "blog",
	   "_id" :      "123",
	   "_version" : 1,
	   "found" :    true,
	   "_source" :  {
	       "title": "My first blog entry",
	       "text":  "Just trying this out...",
	       "date":  "2014/01/01"
	   }
	 }
	错误返回：
	404 not found
######6. 更新
	PUT /website/blog/123
	 {
	   "title": "My first blog entry",
	   "text":  "I am starting to get the hang of this...",
	   "date":  "2014/01/02"
	 }
	成功返回：
	{
	   "_index" :   "website",
	   "_type" :    "blog",
	   "_id" :      "123",
	   "_version" : 2,       # 因为文档已存在，这里version更新成2， created为false
	   "created":   false 
	}

######7. 部分更新
	POST /website/blog/1/_update
	{
	   "doc" : {             # 这里为更新的内容，如果字段已存在，则更新，反之则添加
	      "tags" : [ "testing" ],
	      "views": 0
	   }
	}
######8. 查看文档是否存在
	HEAD index/type/document_id
	成功返回：   200 ok 
	错误返回：   404 not found 
######9. 删除文档
	DELETE /website/blog/123
	成功返回：
	200 ok
	错误返回：
	404 not found 
	{
	   "found" :    false,
	   "_index" :   "website",
	   "_type" :    "blog",
	   "_id" :      "123",
	   "_version" : 4
	 }

######10. 按搜索结果删除文档
	POST /index/type/_delete_by_query
	{
	"query": {
	"match": {"name": "chaxun"}
	        }
	}

######11. 批量获取文档
	GET /index/type/_mget?pretty
	{
	   "ids" : [1,2,3]
	}
	-- 200 ok  不管检索结果存在与否，都返回200， 需要通过检查found来确认检索结果

######12. 搜索文档
	GET /index/type/_search?pretty=true
	{
	 "query": {
	    "match":{
	      "name":"小"
	    }
	  }
	}

######13. set alias
	POST /_aliases
	{
	    "actions" : [
	        { "remove" : { "index" : "products_1113", "alias" : "products" } },
	        { "add" : { "index" : "products_1114", "alias" : "products" } }
	    ]
	}

######14. 测试分词器分析结果
	GET /index/_analyze 
	{
	  "analyzer": "ik_syno",
	  "text": ["香氛"]
	}

