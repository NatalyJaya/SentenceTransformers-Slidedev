---
theme: dracula
title: Sentence Transformers
transition: slide-left
---

# Sentence Transformers

## Pooling

<div class="absolute bottom-10 right-10 text-sm">
  Toni López Llao
</div>

---
theme: dracula

---
# Transformers

<TransformersDiagram/>

---
layout: two-cols
---

# Què és Sentence Transformers?

**Què és:** una llibreria de Python que converteix **frases en vectors numèrics** (embeddings). Frases amb un significat semblant donen vectors propers.

**Com ho fa:**

- Un Transformer (p. ex. BERT) genera un vector per a cada token
- El **pooling** els combina en un sol vector per frase

**Per a què serveix:**

- Semantic search: buscar pel significat, no per paraules exactes
- Calcular la similitud entre frases
- Reranking: reordenar resultats per rellevància

::right::

<div class="h-full flex flex-col justify-center pl-6">

**Keyword search vs semantic search**

Pregunta: **"Gat negre"**

| Frase | Keyword | Semantic |
| --- | :---: | :---: |
| "El gat negre és maco" | ✓ | ✓ |
| "El felí fosc miola" | ✗ | ✓ |
| "Els cotxes negres" | ✓ | ✗ |

<div class="mt-4 text-sm opacity-70">La keyword search només troba paraules iguals. La semantic search entén que "felí fosc" vol dir el mateix que "gat negre".</div>

</div>

---
layout: two-cols
---

# La pregunta de partida

**Què volem saber:** com passa el **pooling** de molts vectors de tokens a un sol vector?

**Com ho veiem:**

- La frase entra al Transformer
- Surt un vector de 768 números per a cada token
- Algú els ha de combinar: aquí és on entra el pooling

**Per què importa:** necessites **1 vector per frase** per poder comparar frases.

::right::

<div class="h-full flex items-center justify-center pl-4">
  <PoolingDiagram/>
</div>

---
layout: two-cols
---

# Què és Pooling?

**Què és:** el pas que converteix **molts vectors** (un per token) en **un sol vector** per a tota la frase.

**Com ho fa:** el Transformer retorna una matriu de *tokens × 768*. El pooling la redueix al llarg dels tokens i en surt un vector de 768 números.

**Per a què serveix:**

- Dues frases de llargades diferents acaben amb vectors de la mateixa mida
- Així es poden comparar directament (similitud del cosinus)
- Sense pooling no hi ha embedding de frase

::right::

<div class="h-full flex items-center justify-center pl-6">

<pre style="font-family: monospace; background: #1e1e1e; padding: 20px; border-radius: 8px; color: #a8ff60;">
Frase: "Gat negre"
    ↓
BERT tokenitza i processa
    ↓
Token 1: [0.1,  0.2,  0.3]
Token 2: [0.5,  0.6,  0.7]
    ↓
 Pooling (mean)
    ↓
Frase:   [0.3,  0.4,  0.5]

 2 vectors → 1 vector
</pre>

</div>
---
layout: two-cols
---

# Estratègies de pooling

**Què són:** maneres diferents de combinar els vectors dels tokens en **un sol vector** per a tota la frase.

**Com ho fan:** totes miren els vectors dels tokens columna a columna, però cada una es queda amb una informació diferent:

- **Mean:** fa la mitjana de tots els tokens
- **CLS:** agafa només el primer token, `[CLS]`
- **Max:** es queda amb el valor més alt de cada columna

**Per a què serveix triar-ne una:** l'estratègia ha de coincidir amb la que es va fer servir per entrenar el model. Si no, els vectors perden qualitat.

::right::

<div class="h-full flex flex-col justify-center pl-6">

| | Què agafa | Quan fer-la servir |
| --- | --- | --- |
| **Mean** | Tots els tokens | Opció per defecte, similitud entre frases |
| **CLS** | Només `[CLS]` | Models entrenats per resumir al `[CLS]` |
| **Max** | El més alt de cada columna | Quan una paraula clau marca el sentit |

<div class="mt-4 text-sm opacity-70">Entrada: N vectors (un per token) → Sortida: 1 vector per frase.</div>

</div>

---
layout: two-cols
---

# Mean Pooling

**Com ho fa:** fa la mitjana de tots els tokens, número a número.

**Per a què serveix:**

- Representació general de tota la frase
- Totes les paraules compten igual
- És l'opció per defecte de Sentence Transformers

<div class="mt-4 text-sm opacity-70">Ideal per a semantic search i similitud entre frases.</div>

::right::

<div class="h-full flex items-center justify-center">
  <PoolingStrategy mode="mean" />
</div>

---
layout: two-cols
---

# CLS Pooling

**Com ho fa:** es queda només amb el vector del token especial `[CLS]`, el primer, i ignora la resta.

**Per a què serveix:**

- Models entrenats perquè el `[CLS]` resumeixi la frase (p. ex. BERT per a classificació)
- Molt ràpid: no cal operar
- Si el model no s'ha entrenat així, el resultat és pitjor

::right::

<div class="h-full flex items-center justify-center">
  <PoolingStrategy mode="cls" />
</div>

---
layout: two-cols
---

# Max Pooling

**Com ho fa:** per a cada posició del vector, es queda amb el valor més alt entre tots els tokens.

**Per a què serveix:**

- Captura les característiques més destacades
- Útil quan una paraula clau marca el significat
- Menys sensible a paraules "de farciment"

::right::

<div class="h-full flex items-center justify-center">
  <PoolingStrategy mode="max" />
</div>

---
layout: two-cols
---

# Codi: Classe Pooling

**Què és:** un mòdul de PyTorch (`nn.Module`) que es col·loca just després del Transformer dins del model.

**Com ho fa:**

- `__init__` guarda la mida dels vectors i el mode de pooling
- `forward` rep els vectors dels tokens i aplica el mode triat
- Cada mode és **una sola línia** de PyTorch

**Per a què serveix:** per canviar d'estratègia només cal un altre `pooling_mode`.

::right::

<div class="h-full flex items-center pl-4">

<!-- dentro de las " | | |" pongo las linias que van a ser subralladas i luego la otra estructura ya es etetica de poner le codigo en python y asi evito poner el tag <pre> -->

```python {all|1-2|4-11|13-17|18-23}{lines:true,maxHeight:'420px'}
class Pooling(nn.Module):
    """Fusiona tokens en 1 vector"""

    def __init__(
        self,
        word_embedding_dimension: int,
        pooling_mode: str = "mean",
    ):
        super().__init__()
        self.word_embedding_dimension = word_embedding_dimension
        self.pooling_mode = pooling_mode

    def forward(self, token_embeddings):
        """
        Entrada: (batch, tokens, 768)
        Sortida: (batch, 768)
        """
        if self.pooling_mode == "mean":
            return token_embeddings.mean(dim=1)
        elif self.pooling_mode == "max":
            return token_embeddings.max(dim=1)[0]
        elif self.pooling_mode == "cls":
            return token_embeddings[:, 0, :]
```

</div>

<style>
/* columna del codi més ampla perquè les línies llargues no es tallin */
.two-columns {
  grid-template-columns: 4fr 5fr !important;
}
</style>

---
layout: two-cols
---

# Mètode forward

**Què és:** el mètode que PyTorch crida cada vegada que el model processa un batch de frases.

**Com ho fa:**

- `dim=1` → opera sobre els tokens
- `.mean()` → calcula la mitjana
- La dimensió dels tokens desapareix

**Per a què serveix:** retorna **1 vector per frase**, llest per comparar-lo amb altres.

::right::

<div class="h-full flex flex-col justify-center pl-4">

```python {all|2-3|5-7}{lines:true}
def forward(self, token_embeddings):
    # Input:  (batch, tokens, 768)
    # Output: (batch, 768)

    if self.pooling_mode == "mean":
        return token_embeddings.mean(dim=1)
        # (32, 50, 768) → (32, 768)
```

<div class="mt-4 text-sm opacity-70">Amb 32 frases de 50 tokens, entren 32 × 50 vectors i en surten només 32.</div>

</div>

---
layout: two-cols
---

# PyTorch Dimensions

**Què és:** `token_embeddings` és un tensor de **3 dimensions**:

- `dim=0`: batch (32 frases)
- `dim=1`: tokens (50 paraules per frase)
- `dim=2`: embedding (768 números per token)

**Com ho fa:** `.mean(dim=1)` fa la mitjana al llarg de la dimensió dels tokens, i aquesta dimensió desapareix.

**Per a què serveix:** processar moltes frases alhora a la GPU, en lloc d'una per una.

::right::

<div class="h-full flex items-center justify-center">
  <TensorShape />
</div>

---
layout: two-cols
---

# Semantic Search

**Què és:** buscar frases pel seu **significat**, no per les paraules exactes.

**Com ho fa:**

1. Es converteix la pregunta en un embedding
2. Es compara amb els embeddings de la BD (similitud del cosinus)
3. S'ordenen els resultats de més a menys similar

**Per a què serveix:** cercadors, chatbots amb documents (RAG), recomanacions…

<div class="mt-4 text-sm opacity-70">Pooling permet comparar vectors: sense 1 vector per frase, no hi ha res a comparar.</div>

::right::

<div class="h-full flex items-center justify-center pl-4">
  <SimilarityRanking />
</div>

---
layout: two-cols
---

# Connexió amb la carrera

**Què és:** el pooling no és màgia, són conceptes que ja hem vist a classe.

**Com ho fa:** combina àlgebra, estadística, càlcul i programació en poques línies de codi.

**Per a què serveix:** entendre el codi d'una llibreria real i poder-hi contribuir.

::right::

<div class="h-full flex flex-col justify-center pl-6">

| Concepte  | A la carrera  | En pooling            |
| --------- | ------------- | --------------------- |
| Arrays    | Àlgebra       | Vectors de tokens     |
| Reducció  | Estadística   | Molts → 1             |
| Batches   | Deep learning | 32 frases             |
| Derivades | Càlcul        | .mean() diferenciable |
| OOP       | Programació   | Composició            |

</div>

---
layout: two-cols
---

# Conclusions

✅ Pooling és simple però potent: una línia de codi

✅ Necessari per a la semantic search

✅ Mean, Max, CLS per a casos diferents

✅ Codi obert: git, PR, tests

✅ Connexió amb la carrera

::right::

<div class="h-full flex items-center justify-center">
  <PipelineRecap />
</div>

---

# Gràcies!

Preguntes?

---
layout: two-cols
---

# Recursos

**Codi:** per veure com està implementat

- GitHub: sentence-transformers
- Fitxer: pooling.py
- Tests: test_pooling.py

::right::

<div class="pl-6 pt-10">

**Docs:** per aprofundir

- deepwiki.com: explicació del repositori
- pytorch.org: `mean`, `max` i tensors

</div>
