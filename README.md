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
