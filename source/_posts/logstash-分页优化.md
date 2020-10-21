---
title: logstash input-jdbc 分页性能优化问题
date: 2019-08-14 17:23:29
updated: 2019-08-16 10:10:29
category: code
tags: 
- elk
- logstash
toc: true
---
![](https://i.loli.net/2020/01/08/khoDBwpMKAb4nJO.jpg)
<!-- more -->

# 原因
logstash 自带分页查询,竟然是查处所有数据再计数的。

```
SELECT count(*) AS `count` FROM (SELECT * FROM ip_bundle_data_20190813) AS `t1` LIMIT 1
```



# 解决办法：修改源码

## 文件位置   

```
logstash-7.2.0\vendor\bundle\jruby\2.5.0\gems\logstash-input-jdbc-4.3.13\lib\logstash\plugin_mixins\jdbc\jdbc.rb
```

## 添加参数

```

// line:100 => config
# 开启子查询分页 @boole 2018-03-30
config :subquery_paging_enabled, :validate => :boolean, :default => false
# 总数sql，结果集列名为sum。for example `select count(*) as sum from goods` @boole 2018-03-30
config :sum_statement, :validate => :string

```

## 修改方法

```

// line:254 => perform_query
private
    def perform_query(query)
		#subquery paging @boole 2018-03-30
		if @subquery_paging_enabled
		  @logger.info("################### subquery paging optimization ################")
		  data_sum = @database[@sum_statement].get(:sum)
		  @logger.info("data_sum=#{data_sum}")
		  data_offset = 0
			while data_offset < data_sum do
				@logger.info("data_offset=#{data_offset}")
				sub_page_query = @database[@statement, symbolized_params({"data_offset" => data_offset, "jdbc_page_size" => @jdbc_page_size})]
				sub_page_query.each do |row|
				yield row
			end
			data_offset += @jdbc_page_size
		end
		elsif @jdbc_paging_enabled
        query.each_page(@jdbc_page_size) do |paged_dataset|
          paged_dataset.each do |row|
            yield row
          end
        end
      else
        query.each do |row|
          yield row
        end
      end
    end

```

## 配置logstash.conf	

sql中使用`:data_offset`和`:jdbc_page_size`进行分页

```
statement => "SELECT * FROM ip_bundle_data_20190813 limit :data_offset,:jdbc_page_size"

subquery_paging_enabled => "true"

sum_statement => "select count(*) as `count` from ip_bundle_data_20190813"

```