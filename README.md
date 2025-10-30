# Algoritmo de Recomendação de Músicas Baseado em Grafos (Neo4j/Cypher)

Este projeto demonstra a modelagem de um sistema de recomendação de músicas utilizando o modelo de **Banco de Dados Orientado a Grafos** (Graph Database), implementado com a linguagem de consulta **Cypher** (para Neo4j).

A arquitetura em grafos é ideal para sistemas de recomendação, pois permite encontrar padrões complexos de relacionamento (ex: "músicas ouvidas por usuários com gostos musicais semelhantes") de forma eficiente e intuitiva.

---

## Modelo de Dados (Esquema de Grafo)

O modelo é baseado no paradigma **LPA (Labels, Properties, Relationships)**.

### Nodos (Labels/Entidades)

| Label | Propriedades Chave | Descrição |
| :---: | :---: | :---: |
| `User` | `userId`, `nome` | Usuário do sistema (20+). |
| `Song` | `songId`, `titulo`, `lancamento` | Uma música no catálogo (20+). |
| `Artist` | `artistId`, `nome` | Cantor, banda ou artista (10+). |
| `Genre` | `nome` | Gênero musical (ex: 'Rock', 'Pop'). |

### Relacionamentos (Relationships)

| Relação (Tipo) | De | Para | Propriedades |
| :---: | :---: | :---: | :---: |
| **`:LISTENED_TO`** | `User` | `Song` | `rating` (Integer: 1-5), `timestamp` |
| **`:PERFORMED_BY`** | `Song` | `Artist` | - |
| **`:HAS_GENRE`** | `Song` | `Genre` | - |
| **`:HAS_GENRE`** | `Artist` | `Genre` | - |

### Diagrama do Grafo

```mermaid
graph LR
    User -->|LISTENED_TO {rating}| Song
    Song -->|PERFORMED_BY| Artist
    Song -->|HAS_GENRE| Genre
    Artist -->|HAS_GENRE| Genre

// Remover restrições e dados existentes para um novo início
MATCH (n) DETACH DELETE n;

// Criar restrições de unicidade para os IDs
CREATE CONSTRAINT IF NOT EXISTS for_userId ON (u:User) REQUIRE u.userId IS UNIQUE;
CREATE CONSTRAINT IF NOT EXISTS for_songId ON (s:Song) REQUIRE s.songId IS UNIQUE;
CREATE CONSTRAINT IF NOT EXISTS for_artistId ON (a:Artist) REQUIRE a.artistId IS UNIQUE;
CREATE CONSTRAINT IF NOT EXISTS for_genreName ON (g:Genre) REQUIRE g.nome IS UNIQUE;

// 5 Gêneros
CREATE (:Genre {nome: 'Rock'});
CREATE (:Genre {nome: 'Pop'});
CREATE (:Genre {nome: 'Jazz'});
CREATE (:Genre {nome: 'Hip Hop'});
CREATE (:Genre {nome: 'Eletrônica'});

// 10 Artistas (Com conexões para Gênero)
MATCH (gR:Genre {nome: 'Rock'}), (gP:Genre {nome: 'Pop'}), (gJ:Genre {nome: 'Jazz'}), (gH:Genre {nome: 'Hip Hop'}), (gE:Genre {nome: 'Eletrônica'})
CREATE (:Artist {artistId: 'AR001', nome: 'Banda Zero'})-[:HAS_GENRE]->(gR),
       (:Artist {artistId: 'AR002', nome: 'Diva Pop'})-[:HAS_GENRE]->(gP),
       (:Artist {artistId: 'AR003', nome: 'Jazz Man'})-[:HAS_GENRE]->(gJ),
       (:Artist {artistId: 'AR004', nome: 'Mestre do Rap'})-[:HAS_GENRE]->(gH),
       (:Artist {artistId: 'AR005', nome: 'The Sound'})-[:HAS_GENRE]->(gE),
       (:Artist {artistId: 'AR006', nome: 'Rock & Rollers'})-[:HAS_GENRE]->(gR),
       (:Artist {artistId: 'AR007', nome: 'Cantora Suave'})-[:HAS_GENRE]->(gP),
       (:Artist {artistId: 'AR008', nome: 'Groove Trio'})-[:HAS_GENRE]->(gJ),
       (:Artist {artistId: 'AR009', nome: 'Flow Urbano'})-[:HAS_GENRE]->(gH),
       (:Artist {artistId: 'AR010', nome: 'DJ Beats'})-[:HAS_GENRE]->(gE);

// 20 Canções (Com conexões para Artista e Gênero)
MATCH (a1:Artist {artistId: 'AR001'}), (a2:Artist {artistId: 'AR002'}), (a3:Artist {artistId: 'AR003'}), (a4:Artist {artistId: 'AR004'}), (a5:Artist {artistId: 'AR005'})
MATCH (a6:Artist {artistId: 'AR006'}), (a7:Artist {artistId: 'AR007'}), (a8:Artist {artistId: 'AR008'}), (a9:Artist {artistId: 'AR009'}), (a10:Artist {artistId: 'AR010'})
MATCH (gR:Genre {nome: 'Rock'}), (gP:Genre {nome: 'Pop'}), (gJ:Genre {nome: 'Jazz'}), (gH:Genre {nome: 'Hip Hop'}), (gE:Genre {nome: 'Eletrônica'})
CREATE (:Song {songId: 'S01', titulo: 'Rocker 1', lancamento: 2020}) -[:PERFORMED_BY]-> (a1) -[:HAS_GENRE]-> (gR),
       (:Song {songId: 'S02', titulo: 'Rocker 2', lancamento: 2021}) -[:PERFORMED_BY]-> (a1) -[:HAS_GENRE]-> (gR),
       (:Song {songId: 'S03', titulo: 'Pop Star', lancamento: 2022}) -[:PERFORMED_BY]-> (a2) -[:HAS_GENRE]-> (gP),
       (:Song {songId: 'S04', titulo: 'Hit do Verão', lancamento: 2023}) -[:PERFORMED_BY]-> (a2) -[:HAS_GENRE]-> (gP),
       (:Song {songId: 'S05', titulo: 'Noite de Jazz', lancamento: 2018}) -[:PERFORMED_BY]-> (a3) -[:HAS_GENRE]-> (gJ),
       (:Song {songId: 'S06', titulo: 'Sax Blue', lancamento: 2019}) -[:PERFORMED_BY]-> (a3) -[:HAS_GENRE]-> (gJ),
       (:Song {songId: 'S07', titulo: 'Ritmo da Rua', lancamento: 2021}) -[:PERFORMED_BY]-> (a4) -[:HAS_GENRE]-> (gH),
       (:Song {songId: 'S08', titulo: 'Flow Pesado', lancamento: 2022}) -[:PERFORMED_BY]-> (a4) -[:HAS_GENRE]-> (gH),
       (:Song {songId: 'S09', titulo: 'Vibe Dance', lancamento: 2023}) -[:PERFORMED_BY]-> (a5) -[:HAS_GENRE]-> (gE),
       (:Song {songId: 'S10', titulo: 'Tech Future', lancamento: 2024}) -[:PERFORMED_BY]-> (a5) -[:HAS_GENRE]-> (gE),
       (:Song {songId: 'S11', titulo: 'Hard Rock', lancamento: 2015}) -[:PERFORMED_BY]-> (a6) -[:HAS_GENRE]-> (gR),
       (:Song {songId: 'S12', titulo: 'Soft Rock', lancamento: 2016}) -[:PERFORMED_BY]-> (a6) -[:HAS_GENRE]-> (gR),
       (:Song {songId: 'S13', titulo: 'Melodia Pura', lancamento: 2017}) -[:PERFORMED_BY]-> (a7) -[:HAS_GENRE]-> (gP),
       (:Song {songId: 'S14', titulo: 'Amor Pop', lancamento: 2018}) -[:PERFORMED_BY]-> (a7) -[:HAS_GENRE]-> (gP),
       (:Song {songId: 'S15', titulo: 'Cool Jazz', lancamento: 2020}) -[:PERFORMED_BY]-> (a8) -[:HAS_GENRE]-> (gJ),
       (:Song {songId: 'S16', titulo: 'Swing', lancamento: 2021}) -[:PERFORMED_BY]-> (a8) -[:HAS_GENRE]-> (gJ),
       (:Song {songId: 'S17', titulo: 'Mixtape 1', lancamento: 2019}) -[:PERFORMED_BY]-> (a9) -[:HAS_GENRE]-> (gH),
       (:Song {songId: 'S18', titulo: 'Verso Livre', lancamento: 2020}) -[:PERFORMED_BY]-> (a9) -[:HAS_GENRE]-> (gH),
       (:Song {songId: 'S19', titulo: 'House Party', lancamento: 2022}) -[:PERFORMED_BY]-> (a10) -[:HAS_GENRE]-> (gE),
       (:Song {songId: 'S20', titulo: 'Trance', lancamento: 2023}) -[:PERFORMED_BY]-> (a10) -[:HAS_GENRE]-> (gE);

// 20 Usuários
UNWIND range(1, 20) AS i
CREATE (:User {userId: 'U' + i, nome: 'Usuario ' + i});

// Conexões de Audição (LISTENED_TO) - Interações Focadas
MATCH (u1:User {userId: 'U1'}), (s1:Song {songId: 'S01'}), (s2:Song {songId: 'S02'}), (s11:Song {songId: 'S11'})
CREATE (u1)-[:LISTENED_TO {rating: 5, timestamp: 1678886400}]->(s1),
       (u1)-[:LISTENED_TO {rating: 4, timestamp: 1678972800}]->(s2),
       (u1)-[:LISTENED_TO {rating: 5, timestamp: 1679059200}]->(s11);

MATCH (u2:User {userId: 'U2'}), (s3:Song {songId: 'S03'}), (s4:Song {songId: 'S04'}), (s13:Song {songId: 'S13'})
CREATE (u2)-[:LISTENED_TO {rating: 5, timestamp: 1678886400}]->(s3),
       (u2)-[:LISTENED_TO {rating: 5, timestamp: 1678972800}]->(s4),
       (u2)-[:LISTENED_TO {rating: 4, timestamp: 1679059200}]->(s13);

MATCH (u3:User {userId: 'U3'}), (s1:Song {songId: 'S01'}), (s5:Song {songId: 'S05'}), (s6:Song {songId: 'S06'})
CREATE (u3)-[:LISTENED_TO {rating: 4, timestamp: 1678886400}]->(s1),
       (u3)-[:LISTENED_TO {rating: 5, timestamp: 1678972800}]->(s5),
       (u3)-[:LISTENED_TO {rating: 5, timestamp: 1679059200}]->(s6);

// Inserir mais interações aleatórias para os outros usuários
WITH ['U4', 'U5', 'U6', 'U7', 'U8', 'U9', 'U10', 'U11', 'U12', 'U13', 'U14', 'U15', 'U16', 'U17', 'U18', 'U19', 'U20'] AS users
UNWIND users AS userId
MATCH (u:User {userId: userId})
WITH u, range(1, 20) AS songIndices 
UNWIND songIndices AS songIndex
WITH u, 'S' + CASE WHEN songIndex < 10 THEN '0' + toString(songIndex) ELSE toString(songIndex) END AS songId
WHERE rand() < 0.2 // Cerca de 20% de chance de o usuário ouvir a música
MATCH (s:Song {songId: songId})
CREATE (u)-[:LISTENED_TO {rating: toInteger(rand() * 4) + 2, timestamp: toInteger(rand() * 10000000) + 1678886400}]->(s);

MATCH (u1:User {userId: 'U1'})-[:LISTENED_TO]->(s1:Song)-[:HAS_GENRE]->(g:Genre {nome: 'Rock'})

// 1. Encontrar usuários similares (u2) que gostaram de músicas do mesmo gênero
MATCH (u2:User)-[l2:LISTENED_TO]->(s2:Song)-[:HAS_GENRE]->(g)
WHERE u2 <> u1 AND l2.rating >= 4 

// 2. Encontrar músicas (s_rec) que u2 ouviu e gostou (alta avaliação)
MATCH (u2)-[l_rec:LISTENED_TO]->(s_rec:Song)
WHERE l_rec.rating >= 4 

// 3. Excluir as músicas que u1 JÁ ouviu
AND NOT EXISTS((u1)-[:LISTENED_TO]->(s_rec))

// 4. Retornar e ordenar as recomendações
RETURN s_rec.titulo AS Recomendacao, 
       s_rec.lancamento AS Ano,
       COUNT(s_rec) AS FrequenciaDeRecomendacao, 
       AVG(l_rec.rating) AS AvaliacaoMedia
ORDER BY FrequenciaDeRecomendacao DESC, AvaliacaoMedia DESC
LIMIT 5

