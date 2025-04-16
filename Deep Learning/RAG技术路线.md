```bash
#启动Neo4j容器并安装apoc依赖
docker run -it --name neo4j --publish=7474:7474 --publish=7687:7687 --env NEO4J_AUTH=none --env NEO4J_PLUGINS='["apoc"]' --volume=I:/neo4j/plugins:/plugins  neo4j:5.20.0
```
```bash
#rag启动指令
docker compose -f docker-compose.yml up -d
```
