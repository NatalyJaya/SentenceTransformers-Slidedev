---
theme: dracula
title: Sentence Transformers
transition: slide-left # https://sli.dev/guide/animations#builtin-transitions aqui hi han mes transicions
---

# Sentence Transformers

<div class="absolute bottom-10 right-10 text-sm">
  Toni López Llao
</div>

---
theme: dracula
transition: slide-left
---
# Qué és Sentence Transformers?

Una llibreria Python que converteix **frases en vectors numèrics**.

**Casos d'ús:**

- Semantic Search
- Computing Embeddings
- Reranking

**El problema:**

- Keyword search: paraules exactes
- Semantic search: pel significat

::right::

**Exemple:**

Pregunta: "Gat negre"

Resultat:

1. "El gat negre és maco" ✓
2. "El felí negre meulla" ✓
3. "Els cotxes negres" ✗

---
theme: dracula
transition: slide-left # la estrictura aquesta de
---

<!-- te ajuda a dir com vols que siguie la estructura de la slide, llavors ja va el contingut -->  
# La pregunta de partida

## Com passa **pooling** de tokens a vector?

<pre style="font-family: monospace; background: #1e1e1e; padding: 20px; border-radius: 8px; color: #a8ff60;">
Frase: "Hola mundo"
    ↓
[Transformer]
    ↓
Token 1: [0.1, 0.2, 0.3, ..., 0.5]  (768 números)
Token 2: [0.4, 0.5, 0.6, ..., 0.8]  (768 números)
    ↓ ¿POOLING?
    ↓
[Vector únic que representa la frase]
</pre>

Per què? Necessites 1 vector per comparar frases.

---
theme: dracula
transition: slide-left # https://sli.dev/guide/animations#builtin-transitions aqui hi han mes transicions
---
# Qué és Pooling?

**El problema:**

<pre style="font-family: monospace; background: #1e1e1e; padding: 20px; border-radius: 8px; color: #a8ff60;">
Frase: "Gat negre"
    ↓
BERT tokenitza i processa
    ↓
Token 1: [0.1,  0.2,  0.3]
Token 2: [0.5,  0.6,  0.7]

❌ Tenim 2 vectors
✅ Necessitem 1 per comparar frases
</pre>

**Estratègies:**

1. **Mean Pooling**: promig
2. **Max Pooling**: màxim
3. **CLS Pooling**: primer token

---
theme: dracula
transition: slide-left # https://sli.dev/guide/animations#builtin-transitions aqui hi han mes transicions
---
# Mean Pooling Visual

<pre style="font-family: monospace; background: #1e1e1e; padding: 20px; border-radius: 8px; color: #a8ff60;">
Token 1: [0.1,  0.2,  0.3]
Token 2: [0.5,  0.6,  0.7]
         --------
Mean:    [0.3,  0.4,  0.5]  ← suma i divideix per 2

Operació: (0.1+0.5)/2 = 0.3
          (0.2+0.6)/2 = 0.4
          (0.3+0.7)/2 = 0.5
</pre>

**Avantatges:**

- Simple i ràpid
- Representa la mitjana

---
theme: dracula
transition: slide-left <!-- https://sli.dev/guide/animations#builtin-transitions
  aqui hi han mes transicions -->
---

# Codi: Classe Pooling
<!-- dentro de las " | | |" pongo las linias que van a ser subralladas i luego la otra estructura ya es etetica de poner le codigo en python y asi evito poner el tag <pre> -->  
```python {all|1-2|4-11|13-17|18-23}{lines:true,maxHeight:'420px'} 
class Pooling(nn.Module):
    """Pooling layer que fusiona tokens en 1 vector"""

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

---
theme: dracula
transition: slide-left 
---

# Mètode forward

```python {all|2-3|5-7}{lines:true}
def forward(self, token_embeddings):
    # Input:  (batch, tokens, 768)
    # Output: (batch, 768)

    if self.pooling_mode == "mean":
        return token_embeddings.mean(dim=1)
        # (32, 50, 768) → (32, 768)
```

**Explicació:**

- `dim=1` → opera sobre tokens
- `.mean()` → calcula mitjana
- Resultat: **1 vector per frase**

---
theme: dracula
transition: slide-left 
---

# PyTorch Dimensions

<pre style="font-family: monospace; background: #1e1e1e; padding: 20px; border-radius: 8px; color: #a8ff60;">
# token_embeddings té 3 dimensiones:
# dim=0: batch (32 frases)
# dim=1: tokens (50 paraules per frase)
# dim=2: embedding (768 números per token)

Shape: (32, 50, 768)
        │   │   └─ embedding_dim
        │   └───── num_tokens
        └───────── batch_size

.mean(dim=1) suma sobre els tokens
(32, 50, 768) → (32, 768)
</pre>

---
theme: dracula
transition: slide-right 
---

# Semantic Search

**Pregunta:** "Gats negres"

**Embedding:** [0.12, -0.45, 0.67, ...]

**BD:**

- "El gat negre és maco"
  → [0.11, -0.43, 0.65, ...] ✓ MÉS SIMILAR

- "Els cotxes negres"
  → [0.5, 0.2, -0.1, ...] ✗ MENYS

**Resultat:** Pooling permet comparar vectors

---
theme: dracula
transition: slide-right
---

# Connexió amb la carrera

| Concepte  | A la carrera  | En pooling            |
| --------- | ------------- | --------------------- |
| Arrays    | Àlgebra       | Vectors de tokens     |
| Reducció  | Estadística   | Molts → 1             |
| Batches   | Deep learning | 32 frases             |
| Derivades | Càlcul        | .mean() diferenciable |
| OOP       | Programació   | Composició            |

---
theme: dracula
transition: slide-right
---

# Conclusions

✅ Pooling és simple pero potent

✅ Necessari per semantic search

✅ Mean, Max, CLS per casos diferents

✅ Codi obert: git, PR, tests

✅ Connexió amb la carrera

---
layout: cover <!-- https://sli.dev/guide/layout hi han layouts per estructura ns si es lo que buscaves aqui por si sirve xd -->
theme: dracula
transition: slide-right
---

# Gràcies!

Preguntes?

---
theme: dracula
transition: slide-right 
---

# Recursos

**Codi:**

- GitHub: sentence-transformers
- Fitxer: pooling.py
- Tests: test_pooling.py

**Docs:**

- deepwiki.com
- pytorch.org
