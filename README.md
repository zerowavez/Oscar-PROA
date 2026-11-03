## Nível 1: Primeiros Passos

### Conhecendo a Base de Dados

**1.1** Quantos registros existem na coleção de indicados ao Oscar?

Exemplo de resposta:

R: 10889 registros

```javascript
db.oscar_indicados.countDocuments();
```

**1.2** Quais são as diferentes categorias de premiação que existem no banco de dados? Liste todas as categorias únicas.

Exemplo de resposta:

R: 92 registros

```javascript
db.oscar_indicados.distinct("categoria");
```

**1.3** Qual foi o primeiro ano de cerimônia do Oscar registrado na base?

R: 1500

```
db.registros.find().skip(15).limit(1).sort({ano_cerimonia: 1})
``` 

**1.4** Qual foi o último ano de cerimônia registrado na base?

R: 2024

```
db.registros.find().skip(15).limit(1).sort({ano_cerimonia: -1})
```

**1.5** Quantas cerimônias do Oscar estão registradas no total?

R: 96

```
db.Oscar.distinct("cerimonia")
```

**1.6** Atualize os registros da coleção com os dados do Oscar 2025 e 2026 (pesquise os indicados e adicione-os).

R: The Brutalist, Sinners

```
db.Oscar.insertOne({
  ano_filmagem: 2024,
  ano_cerimonia: 2025,
  cerimonia: 97,
  categoria: 'ACTOR IN A LEADING ROLE',
  nome_do_indicado: 'Adrien Brody',
  nome_do_filme: 'The Brutalist',
  vencedor: 1
})

db.Oscar.insertOne({
  ano_filmagem: 2025,
  ano_cerimonia: 2026,
  cerimonia: 98,
  categoria: 'ACTOR IN A LEADING ROLE',
  nome_do_indicado: 'Michael B. Jordan',
  nome_do_filme: 'Sinners',
  vencedor: 1
})
```

---

## Nível 2: Explorando Categorias

**2.1** Quantas indicações existem para cada categoria? Agrupe por categoria e ordene da mais frequente para a menos frequente.

R: Mais frequente: `DIRECTING` com 469 indicações. Menos Frequente: `SPECIAL ACHIEVEMENT AWARD (Sound Editing)` com 1 indicação

```
db.Oscar.aggregate([
  {
    $group: {
      _id: "$categoria",
      total_indicacoes: { $sum: 1 }
    }
  },
  {
    $sort: { total_indicacoes: -1 }
  }
])
```

**2.2** Qual categoria teve mais indicações ao longo da história do Oscar?

R: `DIRECTING` com 469 indicações

```
db.Oscar.aggregate([
  {
    $group: {
      _id: "$categoria",
      total_indicacoes: { $sum: 1 }
    }
  },
  {
    $sort: { total_indicacoes: -1 }
  },
  { 
    $limit: 1
  }
])
```

**2.3** Qual categoria teve menos indicações ao longo da história?

R: `SPECIAL ACHIEVEMENT AWARD (Sound Editing)` com 1 indicação

```
db.Oscar.aggregate([
  {
    $group: {
      _id: "$categoria",
      total_indicacoes: { $sum: 1 }
    }
  },
  {
    $sort: { total_indicacoes: 1 }
  },
  { 
    $limit: 1
  }
])
```

**2.4** A partir de que ano a categoria "ACTRESS" deixou de existir? (Dica: procure a última cerimônia com essa categoria)

R: 1976

```
db.Oscar.find({ categoria: 'ACTRESS' })
  .sort({ cerimonia: -1 })
  .limit(1)
```

**2.5** Quais categorias existiam na primeira cerimônia (1928) e não existem mais hoje?

R: `UNIQUE AND ARTISTIC PICTURE`, `DIRECTING (Comedy Picture)`, `ENGINEERING EFFECTS`, `WRITING (Title Writing)`, `DIRECTING (Dramatic Picture)`

```
db.Oscar.aggregate([
  {
    $group: {
      _id: "$categoria",
      ultima_vez_visto: { $max: "$ano_cerimonia" } 
    }
  },
  {
    $match: {
      ultima_vez_visto: 1928
    }
  },
  {
    $project: {
      _id: 0,
      categoria: "$_id"
    }
  }
])
```

**2.6** Liste todas as categorias que contêm a palavra "DIRECTING" no nome.

R: `DIRECTING`, `DIRECTING (Comedy Picture)`, `DIRECTING (Dramatic Picture)`

```
db.Oscar.aggregate([
  {
    $match: {
      categoria: { $regex: "DIRECTING", $options: "i" }
    }
  },
  {
    $group: {
      _id: "$categoria"
    }
  },
  {
    $sort: { _id: 1 } // Sorts the resulting categories alphabetically
  }
])
```

---

## Nível 3: Atores e Atrizes Famosos

### Natalie Portman

**3.1** Quantas vezes Natalie Portman foi indicada ao Oscar?

R: 3

```
db.Oscar.countDocuments({ nome_do_indicado: "Natalie Portman" })
```

**3.2** Quantos Oscars Natalie Portman ganhou?

R: 1

```
db.Oscar.find({
  nome_do_indicado: "Natalie Portman",
  vencedor: 1
})
```

**3.3** Em quais anos e por quais filmes Natalie Portman foi indicada?

R: Closer em 2005, Black Swan em 2011, Jackie em 2017

```
db.Oscar.find(
  {nome_do_indicado: "Natalie Portman"},
  { ano_cerimonia: 1, nome_do_filme: 1, _id: 0 }
)
```

**3.4** Liste todas as indicações de Natalie Portman mostrando: ano, categoria, filme e se venceu.

R: 3 indicações, uma onde venceu

```
db.Oscar.find(
  {nome_do_indicado: "Natalie Portman"},
  { ano_cerimonia: 1, nome_do_filme: 1, vencedor: 1, _id: 0 }
)
```

### Viola Davis

**3.5** Quantas vezes Viola Davis foi indicada ao Oscar?

R: 4

```
db.Oscar.countDocuments({ nome_do_indicado: "Viola Davis" })
```

**3.6** Quantos Oscars Viola Davis ganhou?

R: 1

```
db.Oscar.countDocuments({
  nome_do_indicado: "Viola Davis",
  vencedor: 1
})
```

**3.7** Por quais filmes Viola Davis foi indicada?

R: Doubt, The Help, Fences, Ma Rainey's Black Bottom

```
db.Oscar.find(
  {nome_do_indicado: "Viola Davis"},
  {nome_do_filme: 1}
)
```

### Amy Adams

**3.8** Amy Adams já ganhou algum Oscar?

R: Não

```
db.Oscar.countDocuments({
  nome_do_indicado: "Amy Adams",
  vencedor: 1
})
```

**3.9** Quantas vezes Amy Adams foi indicada sem ganhar?

R: 6

```
db.Oscar.countDocuments({
  nome_do_indicado: "Amy Adams",
})
```

### Denzel Washington

**3.10** Denzel Washington já ganhou algum Oscar?

R: 2

```
db.Oscar.countDocuments({
  nome_do_indicado: "Denzel Washington",
  vencedor: 1
})
```

**3.11** Quantas vezes Denzel Washington foi indicado ao Oscar?

R: 9

```
db.Oscar.countDocuments({
  nome_do_indicado: "Denzel Washington",
})
```

**3.12** Liste todos os Oscars que Denzel Washington ganhou (ano, categoria, filme).

R: Glory e Training Day

```
db.Oscar.find(
  {nome_do_indicado: "Denzel Washington", vencedor: 1},
  {ano_cerimonia: 1, nome_do_filme: 1, categoria: 1, vencedor: 1, _id: 0 }
)
```

---

## Nível 4: Vencedores Históricos

**4.1** Quem ganhou o primeiro Oscar para Melhor Atriz (ACTRESS)? Em que ano e por qual filme?

R: Janet Gaynor em 1928 pelo filme 7th Heaven

```
db.Oscar.find({ 
  cerimonia: 1, 
  categoria: { $regex: "ACTRESS", $options: "i" },
  vencedor: 1
},
{
  nome_do_indicado: 1, ano_cerimonia: 1, nome_do_filme: 1
})
```

**4.2** Quem ganhou o primeiro Oscar para Melhor Ator (ACTOR)? Em que ano e por qual filme?

R: Emil Jannings em 1928 pelo filme The Last Command

```
db.Oscar.find({ 
  cerimonia: 1, 
  categoria: { $regex: "ACTOR", $options: "i" },
  vencedor: 1
},
{
  nome_do_indicado: 1, ano_cerimonia: 1, nome_do_filme: 1
})
```

**4.3** Quantos vencedores existem ao todo na base de dados?

R: 2466

```
db.Oscar.countDocuments({vencedor: 1})
```

**4.4** Liste todos os filmes que ganharam o Oscar de Melhor Filme (categoria "OUTSTANDING PICTURE" ou "BEST PICTURE").

R: 64

```
db.Oscar.aggregate([
  {
    $match: {
      categoria: { $in: ["OUTSTANDING PICTURE", "BEST PICTURE"] },
      vencedor: 1
    }
  },
  {
    $group: {
      _id: "$nome_do_filme"
    }
  },
  {
    $count: "total_vencedores"
  }
])
```

**4.5** Quantos filmes diferentes já ganharam o Oscar?

R: 2071

```
db.Oscar.distinct("nome_do_indicado", { vencedor: 1 }).length
```

---

## Nível 5: Análise de Indicações

**5.1** Quais atores/atrizes foram indicados mais de uma vez? Liste o nome e o número de indicações.

R: Meryl Streep, Katharine Hepburn, etc

```
db.Oscar.aggregate([
  {
    $match: {
      categoria: { $regex: "ACTOR|ACTRESS", $options: "i" }
    }
  },
  {
    $group: {
      _id: "$nome_do_indicado",
      indicacoes: { $sum: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      nome_do_indicado: "$_id",
      total_indicacoes: 1
    }
  },
  {
    $sort: { total_total_indicacoes: -1 }
  }
])
```

**5.2** Qual ator ou atriz tem o maior número de indicações na história do Oscar?

R: Meryl Streep com 21

```
db.Oscar.aggregate([
  {
    $match: {
      categoria: { $regex: "ACTOR|ACTRESS", $options: "i" }
    }
  },
  {
    $group: {
      _id: "$nome_do_indicado",
      total_indicacoes: { $sum: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      nome_do_indicado: "$_id",
      total_indicacoes: 1
    }
  },
  {
    $sort: { total_indicacoes: -1 }
  },
  {
    $limit: 1
  },
])
```

**5.3** Quais atores foram indicados mais de 3 vezes, mas nunca ganharam?

R:  Peter O'Toole, Glenn Close, etc

```
db.Oscar.aggregate([
  {
    $match: {
      categoria: { $regex: "ACTOR|ACTRESS", $options: "i" }
    }
  },
  {
    $group: {
      _id: "$nome_do_indicado",
      total_indicacoes: { $sum: 1 },
      total_vitorias: {
        $sum: {
          $cond: [{ $eq: ["$vencedor", 1] }, 1, 0]
        }
      }
    }
  },
  {
    $match: {
      total_indicacoes: { $gte: 3 },
      total_vitorias: 0
    }
  },
  {
    $project: {
      _id: 0,
      nome_do_indicado: "$_id",
      total_indicacoes: 1,
      total_vitorias: 1
    }
  },
  {
    $sort: { total_indicacoes: -1 }
  }
])
```

**5.4** Encontre todos os artistas que foram indicados em categorias diferentes (ex: ator e diretor).

R: Metro-Goldwyn-Mayer, Kubrick, etc

```
db.Oscar.aggregate([
  {
    $group: {
      _id: "$nome_do_indicado",
      categorias: { $addToSet: "$categoria" }
    }
  },
  {
    $project: {
      _id: 0,
      nome_do_indicado: "$_id",
      categorias: 1,
      total_categorias: { $size: "$categorias" }
    }
  },
  {
    $match: {
      total_categorias: { $gt: 1 }
    }
  },
  {
    $sort: { total_categorias: -1 }
  }
])
```

**5.5** Quantos indicados têm exatamente 1 indicação na história?

R: 5589

```
db.Oscar.aggregate([
  {
    $group: {
      _id: "$nome_do_indicado",
      total_indicacoes: { $sum: 1 }
    }
  },
  {
    $match: { total_indicacoes: 1 }
  },
  {
    $count: "contagem_indicados_apenas_uma_vez"
  }
])
```

**5.6** Qual o maior números de indicados em um único ano? Essa é uma pergunta franca.  

R: 1943 com 186 indicações

```
db.Oscar.aggregate([
  {
    $group: {
      _id: "$ano_cerimonia",
      total_indicacoes: { $sum: 1 }
    }
  },
  {
    $sort: { total_indicacoes: -1 }
  },
  {
    $limit: 1
  },
  {
    $project: {
      _id: 0,
      ano_cerimonia: "$_id",
      total_indicacoes: 1
    }
  }
])
```

---

## Nível 6: Análise de Filmes

### Toy Story

**6.1** A série de filmes Toy Story ganhou Oscars em quais anos?

R: 3 e 4, em 2011 e 2020 respectivamente:

```
db.Oscar.find(
  { 
    nome_do_filme: { $regex: "Toy Story", $options: "i" },
    vencedor: 1
  },
  { 
    _id: 0, 
    nome_do_filme: 1, 
    categoria: 1, 
    ano_cerimonia: 1 
  }
)
```

**6.2** Quantas indicações a franquia Toy Story recebeu no total?

R: 11

```
db.Oscar.aggregate([
  {
    $match: {
      nome_do_filme: { $regex: "Toy Story", $options: "i" }
    }
  },
  {
    $group: {
      _id: "Total Franquia",
      total_indicacoes: { $sum: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      total_indicacoes: 1
    }
  }
])
```

**6.3** Em quais categorias os filmes Toy Story foram indicados?

R: MUSIC (Original Musical or Comedy Score), WRITING (Adapted Screenplay), MUSIC (Original Song), SOUND EDITING, ANIMATED FEATURE FILM, WRITING (Screenplay Written Directly for the Screen), BEST PICTURE

```
db.Oscar.aggregate([
  {
    $match: {
      nome_do_filme: { $regex: "Toy Story", $options: "i" }
    }
  },
  {
    $group: {
      _id: "Total Franquia",
      categorias: { $addToSet: "$categoria" }
    }
  },
  {
    $project: {
      _id: 0,
      categorias: 1,
      total_categorias: { $size: "$categorias" }
    }
  }
])
```

### Crash

**6.4** Em qual edição do Oscar o filme "Crash" concorreu?

R: 2006

```
db.Oscar.find(
  {
    nome_do_filme: { $regex: "^Crash$", $options: "i" },
    categoria: { $regex: "BEST PICTURE", $options: "i" },
    vencedor: 1
  },
  {
    _id: 0,
    nome_do_filme: 1,
    ano_cerimonia: 1,
  }
)
```

**6.5** Quantas indicações o filme "Crash" recebeu?

R: 8 indicações

```
db.Oscar.aggregate([
  {
    $match: {
      nome_do_filme: { $regex: "Crash", $options: "i" }
    }
  },
  {
    $group: {
      _id: "Total Franquia",
      total_indicacoes: { $sum: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      total_indicacoes: 1
    }
  }
])
```

**6.6** "Crash" ganhou o Oscar de Melhor Filme?

R: Sim

```
db.Oscar.find(
  {
    nome_do_filme: { $regex: "^Crash$", $options: "i" },
    categoria: { $regex: "BEST PICTURE", $options: "i" },
    vencedor: 1
  },
  {
    _id: 0,
    nome_do_filme: 1,
    categoria: 1,
    ano_cerimonia: 1,
    vencedor: 1
  }
)
```

### Central do Brasil

**6.7** O filme "Central do Brasil" aparece no banco de dados?

R: Não aparece

```
db.Oscar.aggregate([
  {
    $match: {
      nome_do_filme: { $regex: "Central do Brasil", $options: "i"}
  }
 }
])
```

**6.8** Se sim, quantas indicações "Central do Brasil" recebeu?

R: Não há indicações

```
db.Oscar.aggregate([
  {
    $match: {
      nome_do_filme: { $regex: "Central do Brasil", $options: "i" }
    }
  },
  {
    $group: {
      _id: "Total Franquia",
      total_indicacoes: { $sum: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      total_indicacoes: 1
    }
  }
])
```

---

## Nível 7: Operações de Atualização

**7.1** No campo "vencedor", altere todos os valores "true" (string) para true (booleano) e "false" (string) para false (booleano).

```
db.Oscar.updateMany(
  {},
  [
    {
      $set: {
        vencedor: {
          $cond: {
            if: { $eq: ["$vencedor", 1] },
            then: NumberInt0,
            else: NumberInt1
          }
        }
      }
    }
  ]
)
```

**7.2** Inclua no banco 3 filmes que nunca foram nomeados ao Oscar, mas que você acha que merecem. Use dados fictícios mas realistas.

```
db.Oscar.insertMany([
  {
    ano_cerimonia: 2019,
    cerimonia: 91,
    categoria: "WRITING (ORIGINAL SCREENPLAY)",
    nome_do_indicado: "Boots Riley",
    nome_do_filme: "Sorry to Bother You",
    vencedor: 0
  },
  {
    ano_cerimonia: 1968,
    cerimonia: 40,
    categoria: "INTERNATIONAL FEATURE FILM",
    nome_do_indicado: "Jean-Luc Godard",
    nome_do_filme: "La Chinoise",
    vencedor: 0
  },
  {
    ano_cerimonia: 2010,
    cerimonia: 82,
    categoria: "INTERNATIONAL FEATURE FILM",
    nome_do_indicado: "Sion Sono",
    nome_do_filme: "Love Exposure",
    vencedor: 0
  }
])
```

**7.3** Adicione uma nova categoria chamada "BEST INTERNATIONAL FEATURE FILM" com alguns vencedores recentes (2020-2024).

```
db.Oscar.insertMany([
  {
    ano_cerimonia: 2020,
    cerimonia: 92,
    categoria: "BEST INTERNATIONAL FEATURE FILM",
    nome_do_indicado: "Bong Joon Ho",
    nome_do_filme: "Parasite",
    vencedor: 1
  },
  {
    ano_cerimonia: 2021,
    cerimonia: 93,
    categoria: "BEST INTERNATIONAL FEATURE FILM",
    nome_do_indicado: "Thomas Vinterberg",
    nome_do_filme: "Another Round",
    vencedor: 1
  },
  {
    ano_cerimonia: 2022,
    cerimonia: 94,
    categoria: "BEST INTERNATIONAL FEATURE FILM",
    nome_do_indicado: "Ryusuke Hamaguchi",
    nome_do_filme: "Drive My Car",
    vencedor: 1
  },
  {
    ano_cerimonia: 2024,
    cerimonia: 96,
    categoria: "BEST INTERNATIONAL FEATURE FILM",
    nome_do_indicado: "Jonathan Glazer",
    nome_do_filme: "The Zone of Interest",
    vencedor: 1
  }
])
```

**7.4** Corrija possíveis erros de digitação nos nomes dos filmes (ex: espaços extras, caracteres especiais desnecessários).

```
db.Oscar.find({ nome_do_filme: { $type: "string" } }).forEach(function(doc) {
  let cleanedTitle = doc.nome_do_filme.trim().replace(/\s+/g, ' ');

  if (cleanedTitle !== doc.nome_do_filme) {
    db.Oscar.updateOne(
      { _id: doc._id },
      { $set: { nome_do_filme: cleanedTitle } }
    );
  }
});
```

**7.5** Remova todos os registros com valor NULL no campo nome_do_filme.

```
db.Oscar.deleteMany({
  nome_do_filme: "NULL"
})
```
