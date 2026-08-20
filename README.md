# Compositor e Ouvinte: Análise Computacional de Emoções na Música Popular Brasileira Utilizando BERTimbau

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-conclu%C3%ADdo-brightgreen)]()

> Trabalho de Conclusão de Curso (Artigo Científico) - Análise e Desenvolvimento de Sistemas, Instituto Federal de Educação, Ciência e Tecnologia do Piauí (IFPI), Campus Picos.
>
> **Autora:** Thávyne Kerolly Dias Ribeiro

---

## 📖 Sobre o projeto

Este repositório reúne o código, os dados e os artefatos utilizados na pesquisa que investiga a **lacuna entre a emoção expressa pelo compositor** (na letra de uma música) **e a emoção percebida pelo ouvinte** (em comentários do YouTube), no contexto da Música Popular Brasileira (MPB).

Para isso, foram desenvolvidos **dois modelos BERTimbau especializados**, cada um ajustado via *fine-tuning* para um domínio textual distinto:

- **Modelo de Letras** — treinado com rótulos gerados por um **léxico emocional expandido** (176 termos), capturando a intenção emocional do compositor.
- **Modelo de Comentários** — treinado com rótulos gerados por **weak supervision** (Snorkel), combinando 19 *Labeling Functions* heurísticas e o modelo de linguagem **Gemini 2.5 Flash**, capturando a percepção emocional do ouvinte.

As predições dos dois modelos foram então cruzadas para medir a **convergência e divergência emocional entre compositor e ouvinte**, complementadas por análises temporais (por década e por artista) e por modelagem de tópicos (LDA) por categoria emocional.

### Taxonomia emocional

Com base na Roda das Emoções de Plutchik (1980), adaptada ao contexto cultural da MPB, foram consideradas seis categorias:

`Alegria` · `Tristeza` · `Nostalgia` · `Saudade` · `Amor composto` · `Neutro`

---

## 🎯 Objetivo

Analisar computacionalmente as emoções presentes nas letras da MPB e compará-las com as emoções percebidas pelos ouvintes em comentários do YouTube, avaliando o grau de convergência entre a intenção emocional do compositor e a resposta afetiva do público.

---

## 🧩 Metodologia (visão geral)

```
1. Coleta de Dados        → 8.400 letras (40 artistas) + 48.820 comentários do YouTube
2. Rotulação Silver        → Léxico expandido (letras) | Snorkel + Gemini 2.5 Flash (comentários)
3. Fine-tuning              → 2 modelos BERTimbau independentes (neuralmind/bert-base-portuguese-cased)
4. Inferência Emocional     → Classificação dos corpora completos em 6 categorias
5. Análises e Descobertas   → Convergência compositor x ouvinte, análise temporal, LDA
```

### Coleta

- **Letras**: *web scraping* do [Letras.mus.br](https://www.letras.mus.br) com `Requests` e `BeautifulSoup`, com metadados cronológicos obtidos via API do **MusicBrainz**.
- **Comentários**: extração via `Selenium` de vídeos oficiais (canal "topic") no YouTube, excluindo covers, instrumentais, lives informais e reacts.

### Rotulação

- **Letras**: léxico afetivo expandido com 176 termos distribuídos em 5 categorias emocionais, com pontuação normalizada por tamanho de lista e critério de desempate por especificidade semântica.
- **Comentários**: 19 *Labeling Functions* (palavras-chave, padrões temporais, estrutura textual, referências a perda/luto, entre outras) agregadas probabilisticamente pelo **Label Model** do Snorkel, incluindo uma LF baseada no **Gemini 2.5 Flash** (zero-shot).

### Modelos

| Modelo       | Domínio      | Rotulação          | Max. tokens | Batch size | Épocas |
|--------------|--------------|---------------------|-------------|------------|--------|
| Modelo A     | Comentários  | Snorkel + Gemini    | 128         | 16         | 2      |
| Modelo B     | Letras       | Léxico expandido    | 512 (sliding window) | 8 | 4 (melhor checkpoint na época 2) |

Ambos inicializados a partir de `neuralmind/bert-base-portuguese-cased`, com taxa de aprendizado de 2×10⁻⁵, *weight decay* de 0,01, *warmup ratio* de 0,10 e *Cosine Scheduler*.

---

## 📊 Principais resultados

### Desempenho dos classificadores

| Modelo               | Acurácia | F1-macro | Kappa de Cohen |
|-----------------------|----------|----------|-----------------|
| Comentários (Modelo A)| 79,89%   | 0,744    | 0,734           |
| Letras (Modelo B)     | 80,48%   | 0,764    | 0,747           |

### Convergência emocional compositor vs. ouvinte

- **Taxa global de convergência:** 26,3% (κ = 0,053), sobre 4.479 músicas presentes em ambos os corpora.
- Emoções de valência positiva convergem mais: **amor composto** (41,7%) e **alegria** (40,2%).
- Emoções de valência negativa/reflexiva convergem menos: **nostalgia** (14,8%), **tristeza** (13,4%), **saudade** (11,6%), **neutro** (1,9%).
- Padrão recorrente: emoções negativas nas letras (tristeza, saudade) são frequentemente percebidas pelos ouvintes como **amor composto** ou **alegria** nos comentários — evidência empírica da *lacuna semântico-perceptual* (Hu et al., 2026).

### Distribuição emocional nas letras

`Amor composto` predomina em 35,0% do corpus, seguido por `tristeza` (21,6%), `alegria` (16,8%), `neutro` (15,0%), `saudade` (7,9%) e `nostalgia` (3,7%) — padrão consistente entre décadas (1960–2020) e entre os principais artistas analisados.

### Modelagem de tópicos (LDA)

Coerência (Cv) por categoria emocional variando entre 0,285 (nostalgia) e 0,403 (amor composto), com tópicos qualitativamente interpretáveis e coerentes com cada categoria afetiva.

---

## 📁 Estrutura do repositório

> Ajuste esta seção conforme a organização real das pastas do seu repositório.

```
mpb-emotion-analysis/
├── data/
│   ├── raw/                  # Letras e comentários brutos coletados
│   └── processed/            # Corpora pré-processados e rotulados (silver labels)
├── scraping/                 # Scripts de coleta (Letras.mus.br, MusicBrainz, YouTube/Selenium)
├── labeling/
│   ├── lexicon/               # Léxico emocional expandido (176 termos)
│   └── snorkel/                # 19 Labeling Functions + integração com Gemini 2.5 Flash
├── models/
│   ├── lyrics_model/           # Fine-tuning do BERTimbau para letras
│   └── comments_model/         # Fine-tuning do BERTimbau para comentários
├── analysis/
│   ├── convergence/            # Análise compositor x ouvinte, Kappa de Cohen
│   ├── temporal/                # Análise por década e por artista
│   └── topic_modeling/          # Modelagem de tópicos com LDA
├── notebooks/                  # Notebooks de exploração e geração de figuras
├── environment.yml / requirements.txt
└── README.md
```

---

## ⚙️ Como reproduzir

### Pré-requisitos

- Python 3.10+
- GPU recomendada para o *fine-tuning* dos modelos BERTimbau
- Conta na Vertex AI (Google Cloud) para uso do Gemini 2.5 Flash na rotulação dos comentários

### Instalação

```bash
git clone https://github.com/thavyne-KDR/mpb-emotion-analysis.git
cd mpb-emotion-analysis
pip install -r requirements.txt
```

### Pipeline

```bash
# 1. Coleta de dados
python scraping/collect_lyrics.py
python scraping/collect_comments.py

# 2. Rotulação silver
python labeling/lexicon/label_lyrics.py
python labeling/snorkel/label_comments.py

# 3. Fine-tuning dos modelos
python models/lyrics_model/train.py
python models/comments_model/train.py

# 4. Inferência emocional
python models/lyrics_model/infer.py
python models/comments_model/infer.py

# 5. Análises
python analysis/convergence/compare_composer_listener.py
python analysis/temporal/decade_analysis.py
python analysis/topic_modeling/lda_by_emotion.py
```

> ⚠️ Adapte os caminhos e nomes de script acima aos arquivos reais do repositório.

---

## 🔍 Principais limitações

- Rótulos *silver* (não humanos) gerados por estratégias distintas para letras e comentários, o que exige interpretar a comparação compositor–ouvinte como **exploratória**.
- Viés de seleção nos comentários do YouTube (usuários tendem a comentar músicas com maior identificação emocional).
- Dependência parcial de um modelo de linguagem proprietário (Gemini 2.5 Flash) na etapa de rotulação.
- Corpus delimitado a 40 artistas consagrados da MPB, não representando a totalidade do gênero.
- Taxonomia emocional restrita a 6 categorias (não contempla medo, raiva, surpresa, esperança, culpa, etc.).

Consulte a Seção 4.6 (Ameaças à Validade) do artigo completo para a discussão detalhada.

---

## 📚 Citação

Este trabalho ainda não possui uma publicação formal associada. Enquanto isso, para citar o TCC:

```bibtex
@misc{ribeiro2026compositorouvinte,
  author       = {Ribeiro, Thávyne Kerolly Dias},
  title        = {Compositor e Ouvinte: Análise Computacional de Emoções na Música Popular Brasileira Utilizando BERTimbau},
  year         = {2026},
  howpublished = {Trabalho de Conclusão de Curso, Instituto Federal de Educação, Ciência e Tecnologia do Piauí, Campus Picos},
  url          = {https://github.com/thavyne-KDR/mpb-emotion-analysis}
}
```

> Este README será atualizado com os dados de citação corretos (DOI, venue, ano) assim que o artigo for publicado.

---

## 🗂️ Dados e reprodutibilidade

Para garantir a reprodutibilidade integral dos experimentos, este repositório disponibiliza:

- Scripts de coleta de dados
- As 19 *Labeling Functions* utilizadas no Snorkel
- O léxico afetivo expandido (176 termos)
- Os *datasets* consolidados
- As especificações de ambiente

---

## 📄 Licença

Este projeto está licenciado sob os termos da licença [MIT](LICENSE) - ajuste conforme a licença efetivamente adotada no repositório.

---

## 📬 Contato

Dúvidas, sugestões ou interesse em colaborar? Abra uma *issue* ou entre em contato:

**Thávyne Kerolly Dias Ribeiro** - thavynekerolly126@gmail.com
