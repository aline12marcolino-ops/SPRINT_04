# Checkpoint 02 - Pipeline RAG Completo do Domínio GoodWe

**Curso:** Ciência da Computação / Engenharia - FIAP 2026  
**Disciplina:** Prompt Engineering and Artificial Intelligence  
**Professor:** Jorge Luiz Gomes  
**Módulo:** Módulo 02 - Do Embedding ao RAGAS  

---
##  Integrantes do Grupo

| Nome Completo | RM |
| :--- | :--- |
| Aline Medri Marcolino | RM 569349 |
| Luis Fernando de Azevedo | RM 574167 |
| Gabriel dos Santos Siqueira | RM 572200 |
| Eduardo Novaki Santos Coelho | RM 572649 |
| Pedro Arthur Campos Reis | RM 569913 |
---

##  Visão Geral do Projeto

Este projeto implementa um **pipeline RAG (*Retrieval-Augmented Generation*) completo e modular** focado no ecossistema de soluções de energia da **GoodWe** (inversores fotovoltaicos, carregadores veiculares EV, baterias e portal de monitoramento SEMS).

O pipeline recupera trechos de documentos técnicos oficiais para fundamentar as respostas do modelo LLM (**Gemma via Ollama Cloud**), garantindo respostas precisas, sem alucinações e com citação direta de fontes.

O sistema foi submetido a uma **análise comparativa quantitativa via RAGAS**, testando duas estratégias de divisão de texto (*chunking*) para determinar a configuração com maior índice de *faithfulness* (fidelidade).

---

##  Base de Conhecimento Reais (GoodWe)
O projeto utiliza 5 documentos oficiais e regulatórios reais do domínio de sistemas fotovoltaicos, carregadores de veículos elétricos e normas técnicas de energia:
GW_Compatibility-list-of-GoodWe-EV-Charger.pdf
Descrição: Lista oficial de compatibilidade entre os carregadores para veículos elétricos (EV) e os inversores da GoodWe.
Fonte: Documentação Oficial GoodWe
GW_HCA-G2_Datasheet-PT.pdf
Descrição: Especificações técnicas resumidas (datasheet) da linha de carregadores veiculares HCA-G2 em Português.
Fonte: Datasheet Oficial GoodWe
GW_HCA-G2_User-Manual-PT.pdf
Descrição: Manual completo de instalação, operação e segurança para os utilizadores dos carregadores HCA-G2.
Fonte: Manuais GoodWe PT
GW_SEMS-PLUS_User-Manual-EN.pdf
Descrição: Manual técnico de configuração e monitoramento inteligente de energia através da plataforma SEMS Plus.
Fonte: Portal SEMS GoodWe
NDU 042 - Fornecimento de energia para es...pdf
Descrição: Norma técnica regulatória aplicada ao fornecimento de energia elétrica e requisitos para conexão de sistemas de geração.
Fonte: Normas Técnicas de Concessionária / Regulamentação Setorial


---

##  Arquitetura Técnica do Pipeline

* **Document Loader:** `PyPDFLoader` com injeção automática de metadados (`source_url`, `categoria`, `data_upload`).
* **Text Splitter:** `RecursiveCharacterTextSplitter` com `separators=["\n\n", "\n", ". ", " ", ""]`.
* **Embedding Model:** `nomic-embed-text` via Ollama (`OllamaEmbeddings`).
* **Vector Store:** `ChromaDB` local (`PersistentClient`).
* **Reranker (Diferencial +0,5):** `cross-encoder/ms-marco-MiniLM-L-6-v2` reordenando os $k$ documentos recuperados antes do envio ao LLM.
* **LLM Model:** `gemma` via **Ollama Cloud** (`temperature=0`).
* **Interface (Diferencial +0,5):** Interface Web construída com `Gradio`.

---

##  Comparativo de Chunking e Métricas RAGAS

Foram avaliadas 2 configurações de *chunking* sobre um dataset de 5 perguntas de teste específicas sobre o domínio GoodWe:

| Parâmetro | Configuração A | Configuração B (Vencedora) |
| :--- | :--- | :--- |
| **`chunk_size`** | 256 | **512** |
| **`chunk_overlap`** | 30 (11,7%) | **60 (11,7%)** |
| **Faithfulness Médio** | 0.74 | **0.91**  |
| **Answer Relevancy Médio** | 0.81 | **0.88** |

**Conclusão da Análise:** A **Configuração B** (`chunk_size=512`) apresentou um ganho expressivo na métrica de *faithfulness* (0.91 vs 0.74), pois *chunks* de 256 caracteres fragmentavam trechos de procedimentos técnicos do manual de instalação, perdendo contexto vital durante o *retrieval*.

---

##  Guia Passos a Passo: Como Adicionar Novos Documentos à Base

Para expandir a base de conhecimento do RAG com novos manuais ou datasheets da GoodWe, siga os passos descritos abaixo:

### Passo 1: Preparar o Arquivo
1. Baixe o documento em formato **PDF**, **TXT** ou **Markdown** diretamente de fontes confiáveis (ex.: portal oficial da GoodWe).
2. Salve o arquivo na pasta do projeto ou faça o upload para a raiz do seu ambiente Google Colab.

### Passo 2: Atualizar o Mapeamento no Código
No notebook Python, localize o dicionário de carregamento `documentos_goodwe` e adicione a nova entrada mantendo o padrão de metadados:

```python
# Exemplo de inclusão de um novo arquivo datasheet
documentos_goodwe.append({
	"path": "novo_manual_bateria_lynx.pdf",
	"fonte": "https://en.goodwe.com/Ftp/Downloads/User%20Manual/GW_Lynx_User_Manual.pdf",
	"categoria": "baterias_armazenamento"
})
```

### Passo 3: Executar a Reindexação do VectorDB
Rode novamente as funções de ingestão para processar o novo documento, gerar os chunks e persisti-los no ChromaDB:

```python
# Reindexar a coleção ChromaDB com o novo documento adicionado
processar_e_indexar_documentos(
	documentos_info=documentos_goodwe,
	chunk_size=512,
	chunk_overlap=60,
	collection_name="goodwe_knowledge_base"
)
```

A partir desse momento, a função `buscar()` e a interface do Gradio passarão a consultar as informações do novo documento automaticamente.

---

##  Como Executar o Projeto
1. Abra o arquivo `CKP02_DocMind_RAG.ipynb` no **Google Colab**.
2. Configure a chave de API do Ollama Cloud no painel de **Secrets** (ícone de chave no menu esquerdo) com o nome `OLLAMA_API_KEY`.
3. Certifique-se de que os 5 arquivos PDF estejam salvos na raiz do ambiente Colab.
4. Clique em **Runtime > Run All** (ou `Ctrl + F9`).
5. Execute a última célula para gerar o link público da interface **Gradio**.
No README
