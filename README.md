# scripts-benchmarking-bdgs
Abaixo estão todos os scripts utilizados na produção do artigo "Benchmarking de Bancos de Dados em Grafos: Neo4j e ArangoDB".

## 1 - Queries em Cypher

### 1.1 - Query 1
``` Cypher
MATCH (a:Usuario {id: 1})-[:SEGUE]->(b:Usuario)-[:SEGUE]->(c:Usuario)
RETURN c.id, c.nome
```

### 1.2 - Query 2
``` Cypher
MATCH (u:Usuario {id: 100})-[i]->(:Post)-[:POSSUI]->(t:Tag)
WITH u, t, COUNT(i) AS total
RETURN t.nome
ORDER BY total DESC
LIMIT 1
```

### 1.3 - Query 3
``` Cypher
MATCH (p:Post)-[:PUBLICADO_POR]->(u:Usuario)
WITH u.cidade AS cidade,
     SUM(p.qtd_curtidas + p.qtd_comentarios + p.qtd_compartilhamentos) AS total_engajamento
RETURN cidade, total_engajamento
ORDER BY total_engajamento DESC
LIMIT 10
```

## 2 - Queries em AQL

### 2.1 - Query 1
``` AQL
FOR a IN Usuario
  FILTER a._key == "1"
  FOR b, edgeB IN OUTBOUND a segue
    FOR c, edgeC IN OUTBOUND b segue
      RETURN { id: c._key, nome: c.nome }
```

### 2.2 - Query 2
``` AQL
WITH Usuario, Post, Tag

FOR u IN Usuario
  FILTER u._key == "100"
  FOR p, edgeP IN OUTBOUND u curtiu, compartilhou, comentou
    FOR t, edgeT IN OUTBOUND p possui
      COLLECT nomeTag = t.nome INTO grupo
      LET total = LENGTH(grupo)
      SORT total DESC
      LIMIT 1
      RETURN nomeTag
```

### 2.3 - Query 3
``` AQL
WITH Post, Usuario

FOR p IN Post
  FOR u IN OUTBOUND p publicadoPor
    COLLECT cidade = u.cidade
      AGGREGATE totalEngajamento = SUM(
        p.qtdCurtidas + p.qtdComentarios + p.qtdCompartilhamentos
      )
    SORT totalEngajamento DESC
    LIMIT 10
    RETURN { cidade, totalEngajamento }
```

## 3 - Queries em Gremlin

### 3.1 - Query 1
``` Gremlin
g.V()
  .has('Usuario', 'id', 'u1')
  .out('segue')
  .out('segue')
  .values('id', 'nome')
```

### 3.2 - Query 2
``` Gremlin
g.V()
  .has('Usuario', 'id', 'u100')
  .out().hasLabel('Post')
  .out().hasLabel('Tag')
  .group()
    .by('nome')
    .by(count())
  .order(local).by(values, decr)
  .limit(local,1)
  .unfold()
  .select(keys).as('topTag')
```

### 3.3 - Query 3
``` Gremlin
g.V()
  .hasLabel('Post').as('p')
  .out('publicadoPor').as('u')
  .group()
    .by(select('u').values('cidade'))
    .by(select('p').values('qtdCurtidas', 'qtdComentarios', 'qtdCompartilhamentos').sum())
  .unfold()
  .order().by(values, decr)
  .limit(10)
```

## 4 - Comandos shell

### 4.1 - Comando para execução do script de dump no container do Neo4j
``` shell
export ANTES=$(date); cypher-shell -u neo4j -p test1234 -f /var/tmp/script.cypher; echo "Antes: $ANTES; Depois: $(date)"
```

### 4.2 - Comando para execução do script de dump no container do ArangoDB

``` shell
export ANTES=$(date); arangosh --server.endpoint tcp://127.0.0.1:8529 --server.username root --server.password test1234 --server.database _system < /var/tmp/script.aql; echo "Antes: $ANTES; Depois: $(date)"
```

## 5 - Criação dos índices no Neo4j
``` Cypher
CREATE CONSTRAINT usuario_id_unique FOR (u:Usuario) REQUIRE u.id IS UNIQUE;
CREATE CONSTRAINT post_id_unique FOR (p:Post) REQUIRE p.id IS UNIQUE;
CREATE CONSTRAINT tag_id_unique FOR (t:Tag) REQUIRE t.id IS UNIQUE;
```

