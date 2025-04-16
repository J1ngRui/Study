```bash
#启动Neo4j容器
docker run -itd --name neo4j -p 7474:7474 -p 7687:7687  -v /opt/neo4j/plugins:/plugins  -v /opt/neo4j/data:/data  -v /opt/neo4j/logs:/logs  -e NEO4J_AUTH=neo4j/12345678 -e NEO4J_dbms_security_procedures_unrestricted=apoc.* neo4j:5.26.0
#装入APOC依赖
docker cp I:\neo4j\plugins\apoc-5.26.0-extended.jar neo4j:/plugins/
```
