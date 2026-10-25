# 🎵 Sistema de Recomendação de Músicas com Neo4j

Este projeto foi desenvolvido como parte do desafio da plataforma DIO. O objetivo é demonstrar a aplicação de um banco de dados orientado a grafos (Neo4j) para resolver um problema clássico: recomendação de conteúdo.

## 🧠 Contexto e Justificativa (Por que Grafos?)

No mundo de engenharia de dados (pipelines, ETL e modelagem relacional), sistemas de recomendação em bancos de dados SQL tradicionais geralmente esbarram em gargalos de performance. Cruzar dados de usuários, músicas, artistas, gêneros e histórico de reproduções exige múltiplos `JOINs` pesados e tabelas de junção imensas, o que degrada o tempo de resposta em consultas profundas.

O Neo4j foi escolhido porque trata os relacionamentos como elementos de primeira classe. O algoritmo não precisa varrer tabelas inteiras; ele apenas "caminha" pelos nós conectados (O(1) pointer hopping). Isso torna consultas de filtragem colaborativa (ex: "quem ouviu o que eu ouvi, também ouviu X") extremamente eficientes.

## 🗂️ Modelo do Grafo

O banco foi modelado com as seguintes entidades e relações:
- **Nós:** `User`, `Song`, `Artist`, `Genre`.
- **Relacionamentos:** 
  - `(User)-[:LISTENED_TO {play_count}]->(Song)`
  - `(Song)-[:COMPOSED_BY]->(Artist)`
  - `(Song)-[:IN_GENRE]->(Genre)`

## ❓ Queries de Negócio

Aqui estão as perguntas que o sistema responde para gerar recomendações.

### 1. Recomendação baseada no gosto de usuários similares (Filtro Colaborativo)
*Se a Camila ouviu uma música, quais outras músicas os usuários que também ouviram essa mesma música recomendariam?*
```cypher
MATCH (u1:User {name: 'Camila'})-[:LISTENED_TO]->(s:Song)<-[:LISTENED_TO]-(u2:User)
MATCH (u2)-[:LISTENED_TO]->(recom:Song)
WHERE NOT (u1)-[:LISTENED_TO]->(recom)
RETURN recom.title AS Recomendacao, COUNT(recom) AS Forca_Recomendacao
ORDER BY Forca_Recomendacao DESC
LIMIT 5

<img width="1276" height="289" alt="recomendacao" src="https://github.com/user-attachments/assets/a5cdfe55-e7db-4b29-b67c-3e82084409a1" />
