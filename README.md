# Algoritmo de Recomendación de Músicas con Grafos - Neo4j

## 1. Modelado del Banco de Datos en Grafos

### Entidades (Nodos)
- **User**: representa los usuarios.
  - Propiedades: `user_id`, `name`, `email`
- **Song**: representa canciones.
  - Propiedades: `song_id`, `title`, `release_year`
- **Singer**: representa cantantes.
  - Propiedades: `singer_id`, `name`
- **Genre**: representa géneros musicales.
  - Propiedades: `name`
- **Recommendation**: representa recomendaciones generadas.
  - Propiedades: `rec_id`, `reason`

### Relaciones
- **WATCHED**: usuario escuchó la canción.
  - Propiedades: `rating` (1 a 5)
- **SINGS**: cantante interpreta la canción.
- **HAS_GENRE**: canción pertenece a un género.
- **RECOMMENDED**: canción recomendada a un usuario.

### Diagrama conceptual (esbozo textual)

---

## 2. Script Cypher de Creación de Nodos y Relaciones

```cypher
// --- Crear Usuarios (20) ---
CREATE
(u1:User {user_id:1, name:'Anna', email:'anna@email.com'}),
(u2:User {user_id:2, name:'Bruno', email:'bruno@email.com'}),
(u3:User {user_id:3, name:'Carla', email:'carla@email.com'}),
(u4:User {user_id:4, name:'Diego', email:'diego@email.com'}),
(u5:User {user_id:5, name:'Elena', email:'elena@email.com'}),
(u6:User {user_id:6, name:'Felipe', email:'felipe@email.com'}),
(u7:User {user_id:7, name:'Gabriela', email:'gabriela@email.com'}),
(u8:User {user_id:8, name:'Hugo', email:'hugo@email.com'}),
(u9:User {user_id:9, name:'Isabela', email:'isabela@email.com'}),
(u10:User {user_id:10, name:'João', email:'joao@email.com'}),
(u11:User {user_id:11, name:'Karen', email:'karen@email.com'}),
(u12:User {user_id:12, name:'Lucas', email:'lucas@email.com'}),
(u13:User {user_id:13, name:'Mariana', email:'mariana@email.com'}),
(u14:User {user_id:14, name:'Nicolas', email:'nicolas@email.com'}),
(u15:User {user_id:15, name:'Olivia', email:'olivia@email.com'}),
(u16:User {user_id:16, name:'Paulo', email:'paulo@email.com'}),
(u17:User {user_id:17, name:'Quintino', email:'quintino@email.com'}),
(u18:User {user_id:18, name:'Rafaela', email:'rafaela@email.com'}),
(u19:User {user_id:19, name:'Sergio', email:'sergio@email.com'}),
(u20:User {user_id:20, name:'Tatiana', email:'tatiana@email.com'});

// --- Crear Cantantes (10) ---
CREATE
(sg1:Singer {singer_id:1, name:'Adele'}),
(sg2:Singer {singer_id:2, name:'Ed Sheeran'}),
(sg3:Singer {singer_id:3, name:'Beyonce'}),
(sg4:Singer {singer_id:4, name:'Bruno Mars'}),
(sg5:Singer {singer_id:5, name:'Coldplay'}),
(sg6:Singer {singer_id:6, name:'Shakira'}),
(sg7:Singer {singer_id:7, name:'The Weeknd'}),
(sg8:Singer {singer_id:8, name:'Taylor Swift'}),
(sg9:Singer {singer_id:9, name:'Billie Eilish'}),
(sg10:Singer {singer_id:10, name:'Dua Lipa'});

// --- Crear Géneros ---
CREATE
(g1:Genre {name:'Pop'}),
(g2:Genre {name:'Rock'}),
(g3:Genre {name:'R&B'}),
(g4:Genre {name:'Hip-Hop'}),
(g5:Genre {name:'Indie'}),
(g6:Genre {name:'Latin'});

// --- Crear Canciones (50) ---
UNWIND range(1,50) AS id
CREATE (s:Song {
    song_id:id,
    title:'Song '+id,
    release_year:2015 + toInteger(rand()*8)
});

// --- Relacionar Cantantes con Canciones ---
MATCH (s:Singer), (song:Song)
WHERE id(s) = (id(song) % 10)
CREATE (s)-[:SINGS]->(song);

// --- Relacionar Canciones con Géneros aleatorios ---
MATCH (song:Song), (g:Genre)
WHERE id(g) = (id(song) % 6)
CREATE (song)-[:HAS_GENRE]->(g);

// --- Usuarios escuchando canciones con rating aleatorio ---
MATCH (u:User), (song:Song)
WHERE id(u) = (id(song) % 20)
CREATE (u)-[:WATCHED {rating: toInteger(rand()*5)+1}]->(song);

// --- Generar recomendaciones automáticas basadas en género favorito ---
MATCH (u:User)-[w:WATCHED]->(s:Song)-[:HAS_GENRE]->(g:Genre)
WITH u, g, collect(s) AS listened_songs
MATCH (rec:Song)-[:HAS_GENRE]->(g)
WHERE NOT rec IN listened_songs
WITH u, rec LIMIT 3
CREATE (u)-[:RECOMMENDED {reason:'Género similar'}]->(rec);
