
# Addax GaussDB使用指南

## Addax介绍
Addax 是一个异构数据源离线同步工具，致力于实现包括关系型数据库(GaussDB、MySQL、Oracle 等)、HDFS、Hive、HBase、FTP 等各种异构数据源之间稳定高效的数据同步功能。  
GaussDBReader插件实现了从GaussDB读取数据。GaussDBWriter插件实现了写入数据到GaussDB。

官方参考文档: [https://wgzhao.github.io/Addax/latest/quickstart/](https://wgzhao.github.io/Addax/latest/quickstart/) 

## Addax 安装

可以直接使用 Docker 镜像，只需要执行下面的命令即可

```shell
docker run -it --rm quay.io/wgzhao/addax:latest /opt/addax/bin/addax.sh /opt/addax/job/job.json
```

## 案例分享

本案例实现场景为 从GaussDB的一张源表抽取数据写入到另外一张不同库的GaussDB目标表。          

* 配置一个自定义SQL的数据库同步任务的作业：  

```json
{
    "job": {
        "setting": {
            "speed": 1048576
        },
        "content": [
            {
                "reader": {
                    "name": "gaussdbreader",
                    "parameter": {
                        "username": "xx",
                        "password": "xxxx",
                        "where": "",
                        "connection": [
                            {
                                "querySql": [
                                    "select id,name,comment from schema.table_name where id < 10;"
                                ],
                                "jdbcUrl": [
                                    "jdbc:gaussdb://host:port/database"
                                ]
                            }
                        ]
                    }
                },
				"writer": {
					"name": "gaussdbwriter",
					"parameter": {
						"username": "xx",
						"password": "xxxx",
						"preSql": ["truncate table schema.table_name"],						
						"connection": [
							{
								"jdbcUrl": "jdbc:gaussdb://host:port/database",
								"table": [
									"schema.table_name"
								]
							}
						],
						"column": ["id","name","comment"],
						"postSql": []
					}
				}
            }
        ]
    }
}
```

* 执行作业      

```shell
python bin/addax.py job/gaussdb2gaussdb.json
```

作业执行完成，可以查看目标表是否正确写入数据。 
