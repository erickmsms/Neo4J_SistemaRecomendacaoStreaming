# 🎵 Sistema de Recomendação de Músicas com Neo4j e Louvain

Este projeto implementa um **sistema de recomendação de músicas baseado em similaridade entre usuários**, utilizando o banco de grafos **Neo4j** e o algoritmo de **detecção de comunidades de Louvain** da biblioteca **Graph Data Science (GDS)**.


## 🚀 **Objetivo**

Recomendar músicas para um usuário com base nos gostos de outros usuários que pertencem à mesma comunidade de interesse musical.


## 🧩 **Etapas do Projeto**

### 1. Criação do Banco de Dados

O script inicializa o grafo com:

* Usuários e seus gostos (gênero, artista e música favorita);
* Nós de **Gênero**, **Artista** e **Música**;
* Relações entre usuários e seus gostos (`GOSTA_DE_GENERO`, `GOSTA_DE_ARTISTA`, `OUVE`).

Exemplo de nó:

```cypher
(u1:Usuario {nome: 'Erick', genero_favorito: 'Rock', artista_favorito: 'Queen', musica_favorita: 'Bohemian Rhapsody'})
```


### 2. Criação do Grafo de Similaridade

Cria relações `PARECIDO_COM` entre usuários que compartilham gostos semelhantes:

* Mesmo gênero → +1 ponto
* Mesmo artista → +2 pontos

Somente conexões com pontuação positiva são criadas:

```cypher
CREATE (u1)-[:PARECIDO_COM {peso: score}]->(u2)
```


### 3. Detecção de Comunidades com Louvain

Utiliza o algoritmo **Louvain** da GDS para agrupar usuários com base em suas conexões de similaridade:

```cypher
CALL gds.louvain.write('usuarios_similares', { writeProperty: 'comunidadeId' });
```

Cada usuário recebe um identificador de comunidade (`comunidadeId`).


### 4. Visualização das Comunidades

Lista os usuários agrupados por comunidade:

```cypher
RETURN u.nome, u.genero_favorito, u.artista_favorito, u.comunidadeId
```


### 5. Geração de Recomendações

Recomenda músicas ouvidas por usuários da mesma comunidade que o usuário-alvo:

```cypher
MATCH (u:Usuario {nome: 'Erick'})
MATCH (outros)-[:OUVE]->(musica:Musica)
WHERE outros.comunidadeId = u.comunidadeId AND NOT (u)-[:OUVE]->(musica)
RETURN DISTINCT musica.titulo AS Recomendacao
```


## 🧠 **Tecnologias Utilizadas**

* **Neo4j** (Banco de grafos)
* **Cypher** (Linguagem de consultas)
* **Neo4j Graph Data Science (GDS)**
* **Algoritmo de Louvain** (Detecção de comunidades)

## 📊 **Resultado Esperado**

Usuários com gostos parecidos são agrupados na mesma comunidade, e o sistema é capaz de sugerir novas músicas relevantes para cada um.

