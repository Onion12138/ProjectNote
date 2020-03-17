## Neo4j

### Docker下安装部署

```shell
 docker run -di --name neo4j -p 7474:7474 -p 7687:7687 -v ~/notehub/neo4j/data:/var/lib/neo4j/data -v ~/notehub/neo4j/conf:/var/lib/neo4j/conf -v ~/notehub/neo4j/logs:/var/lib/neo4j/logs neo4j:3.5
```

### SpringBoot整合

```yml
spring:
  data:
    neo4j:
      uri: bolt://120.77.176.55:7687
      username: neo4j
      password: 969023014
```



```java
@Query("match (note1:note)-[r:relate]-(note2:note)" +
            "where note1.noteId = {0} " +
            "return note2 " +
            "order by r.relatedTimes " +
            "limit 20")
    List<NoteInfo> findNearestRelatedNotes(String noteId);

    @Query("match (note1:note)-[r:relate*2..3]-(note2:note)" +
            "where note1.noteId = {0} " +
            "return note2 " +
            "limit 10")
    List<NoteInfo> findNearbyRelatedNotes(String noteId);

    @Query("MATCH (n1:note),(n2:note) " +
            "WHERE n1.noteId = {0} AND n2.noteId = {1} " +
            "MERGE (n1)-[r:relate]-(n2) " +
            "ON CREATE SET r.relateTimes = 1 " +
            "ON MATCH SET r.relateTimes = r.relateTimes + 1")
    void addRelation(String NoteId1, String noteId2);
```

![avatar](http://ecnuonion.club/969023014%40qq.com07da7496e0ff4cc0869f5a5c9d0781a6?e=1614415930&token=SStPJbNpriAFEzb0LvB1ooO7X__CB5xpwt8cE8UE:jslCeSEFDXRvPvY1DUstFKUkhDA=)

