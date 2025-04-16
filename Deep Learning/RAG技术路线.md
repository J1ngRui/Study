```bash
#启动Neo4j容器并安装apoc依赖
docker run -itd --name neo4j -p 7474:7474 -p 7687:7687 -e NEO4J_dbms_memory_transaction_total_max=4G -e NEO4JLABS_PLUGINS='["apoc"]' -e NEO4J_apoc_import_file_enabled=true -e NEO4J_AUTH=neo4j/123qweasd! neo4j:latest
```
