# Product Requirements Document (PRD)
## PaperFetch - Busca e Download Automático de Artigos Acadêmicos

**Versão:** 1.1  
**Data:** Dezembro 2025  
**Autor:** Alana - UFMA  
**Status:** Draft

---

## 1. Visão Geral do Produto

### 1.1 Resumo Executivo
**PaperFetch** é um aplicativo web para automatizar a busca e download de artigos científicos a partir de referências bibliográficas em diversos formatos. O sistema identifica automaticamente o tipo de referência fornecida e busca o artigo em múltiplas fontes acadêmicas, priorizando versões de acesso aberto.

### 1.2 Problema
Pesquisadores e educadores perdem tempo significativo:
- Copiando referências para diferentes bases de dados
- Tentando múltiplas fontes para encontrar PDFs acessíveis
- Lidando com paywalls e acesso restrito
- Convertendo entre formatos de citação

### 1.3 Solução
Uma interface única que aceita qualquer formato de referência e automatiza a busca em múltiplas fontes acadêmicas, retornando links diretos para download quando disponíveis.

### 1.4 Público-Alvo
- **Primário:** Pesquisadores acadêmicos, professores universitários, estudantes de pós-graduação
- **Secundário:** Estudantes de graduação, bibliotecários, profissionais de revisão sistemática
- **Contexto:** Principalmente no Brasil, com suporte para repositórios nacionais (SciELO, periódicos CAPES)

### 1.5 Objetivos de Negócio
- Reduzir tempo médio de busca de artigos de 5-10 minutos para <30 segundos
- Aumentar taxa de sucesso na localização de PDFs de acesso aberto
- Criar ferramenta de uso frequente para comunidade acadêmica brasileira

---

## 2. Requisitos Funcionais

### 2.1 Parser de Referências

#### RF-001: Detecção Automática de Formato
**Prioridade:** ALTA  
**Descrição:** O sistema deve identificar automaticamente o tipo de referência fornecida pelo usuário.

**Formatos Suportados:**
- DOI (Digital Object Identifier)
  - Exemplos: `10.1234/example`, `https://doi.org/10.1234/example`
- Título completo do artigo
- Citação ABNT
  - Exemplo: `SOBRENOME, Nome. Título. Revista, v. X, n. Y, p. Z-W, ano.`
- Citação APA
  - Exemplo: `Autor, A. (Ano). Título. Revista, volume(número), páginas.`
- URL de artigo
  - Google Scholar, SciELO, PubMed, arXiv, etc.
- PMID (PubMed ID)
- arXiv ID

**Critérios de Aceitação:**
- Sistema identifica corretamente 95%+ dos formatos padrão
- Feedback visual indica formato detectado
- Permite correção manual se detecção falhar

#### RF-002: Extração de Metadados
**Prioridade:** ALTA  
**Descrição:** Extrair informações estruturadas da referência fornecida.

**Campos Extraídos:**
- Título do artigo (obrigatório)
- Autor(es) principal(is)
- Ano de publicação
- DOI (quando disponível)
- Nome da revista/conferência
- Volume, número, páginas

**Critérios de Aceitação:**
- Título extraído com 99%+ de acurácia
- Outros campos com 85%+ de acurácia
- Sistema tolera variações de formatação

#### RF-003: Normalização de Dados
**Prioridade:** MÉDIA  
**Descrição:** Padronizar dados extraídos para facilitar busca.

**Processamento:**
- Remover pontuação excessiva
- Normalizar espaços e quebras de linha
- Converter acentuação para busca
- Identificar e separar nomes de autores

---

### 2.2 Sistema de Busca Multi-fonte

#### RF-004: Integração com Fontes Acadêmicas
**Prioridade:** ALTA  
**Descrição:** Buscar artigo em múltiplas bases de dados acadêmicas.

**Fontes Prioritárias (Fase 1):**
1. **CrossRef API** - Busca por DOI e metadados
2. **Unpaywall API** - Localiza versões de acesso aberto
3. **SciELO API** - Artigos latino-americanos
4. **PubMed/PMC API** - Artigos biomédicos
5. **arXiv API** - Preprints de física, matemática, CS
6. **Semantic Scholar API** - Busca semântica ampla

**Fontes Secundárias (Fase 2):**
- Google Scholar (via scraping cuidadoso)
- Core.ac.uk API
- BASE (Bielefeld Academic Search Engine)
- Repositórios institucionais brasileiros

**Critérios de Aceitação:**
- Busca paralela em todas as fontes (max 10s timeout)
- Retry automático em caso de falha temporária
- Consolidação de resultados duplicados
- Priorização por confiabilidade da fonte

#### RF-005: Estratégia de Busca Adaptativa
**Prioridade:** ALTA  
**Descrição:** Ajustar estratégia de busca baseado no tipo de referência.

**Fluxos:**
1. **Se DOI disponível:**
   - Busca direta via CrossRef
   - Verifica Unpaywall para PDF aberto
   - Fallback para busca por título se falhar

2. **Se apenas título:**
   - Busca por título exato em todas as fontes
   - Se <3 resultados, busca fuzzy (tolerância a erros)
   - Ranqueia por similaridade de metadados

3. **Se citação completa:**
   - Extrai título e busca primária por ele
   - Usa autor/ano para filtrar resultados
   - Valida match comparando metadados

**Critérios de Aceitação:**
- Taxa de sucesso >80% para DOIs
- Taxa de sucesso >60% para títulos completos
- Taxa de sucesso >50% para citações complexas

#### RF-006: Detecção de Acesso Aberto
**Prioridade:** ALTA  
**Descrição:** Identificar quais resultados possuem PDF disponível gratuitamente.

**Verificações:**
- Link direto para PDF na resposta da API
- Verificação de disponibilidade via HEAD request
- Classificação: "PDF Disponível", "Acesso Restrito", "Incerto"
- Indica tipo de acesso aberto (Gold, Green, Hybrid)

---

### 2.3 Interface do Usuário

#### RF-007: Tela de Busca
**Prioridade:** ALTA  
**Descrição:** Interface principal para inserção de referências.

**Modos de Entrada:**

**Modo 1: Busca Individual**
- Campo de texto amplo (textarea) para colar referência
- Placeholder com exemplos de formatos aceitos
- Botão "Buscar Artigo" com estado de loading
- Indicador de formato detectado (badge)
- Opção "Limpar" para nova busca

**Modo 2: Upload de Arquivo BibTeX**
- Área de drag & drop para arquivo .bib
- Botão "Selecionar arquivo" alternativo
- Validação de tipo de arquivo (.bib)
- Indicador de tamanho do arquivo
- Preview das referências detectadas
- Opção de selecionar/desmarcar referências
- Botão "Processar Lote"

**Toggle entre modos:**
- Abas ou toggle switch: "Busca Individual" | "Lote (.bib)"
- Estado mantido durante sessão

**Comportamento (Individual):**
- Auto-detecção ao colar texto
- Enter para buscar (com Shift+Enter para nova linha)
- Histórico de buscas recentes (últimas 10)

**Comportamento (Lote):**
- Drag & drop ou clique para upload
- Parser automático ao carregar arquivo
- Exibe tabela de preview com checkboxes
- Contador: "X referências selecionadas"

#### RF-008: Tela de Resultados
**Prioridade:** ALTA  
**Descrição:** Exibição clara dos artigos encontrados.

**Modo Individual:**

Para cada resultado:
- Título completo (destaque)
- Autores (primeiros 3 + "et al." se mais)
- Ano, revista, volume/número
- Badge de disponibilidade: 🟢 PDF disponível | 🟡 Link apenas | 🔴 Acesso restrito
- Botões de ação:
  - "Baixar PDF" (se disponível)
  - "Abrir no site" (link externo)
  - "Copiar citação"
  - "⭐ Salvar"
- Fonte da informação (pequeno texto)

Organização:
- Resultados com PDF primeiro
- Ordenação por relevância/confiabilidade
- Max 10 resultados por busca
- Indicador "Nenhum resultado encontrado" se aplicável

**Modo Lote:**

Tabela de resultados com colunas:
- ☑️ Checkbox para seleção
- Título (truncado com tooltip)
- Autores (principal + et al.)
- Ano
- Status visual: ✅ PDF | 🔗 Link | ❌ Não encontrado | ⏳ Processando
- Ações rápidas (ícones):
  - 📥 Baixar PDF
  - 🔗 Abrir link
  - 📋 Copiar citação
  - ⭐ Salvar

Controles da tabela:
- Filtros dropdown: [Todos | Apenas com PDF | Não encontrados]
- Ordenação por: [Ordem original | Título | Ano | Status]
- Busca dentro dos resultados (campo de texto)
- Paginação (20 por página)

Ações em lote (barra superior):
- "Baixar PDFs Selecionados" (X selecionados)
- "Exportar Resultados" (dropdown de formatos)
- "Selecionar Todos/Nenhum"

Sumário (card no topo):
```
📊 Resumo do Processamento
✅ 35 artigos encontrados (70%)
🟢 28 com PDF disponível (80%)
🟡 7 apenas link
❌ 15 não encontrados (30%)
⏱️ Processado em 4m 32s
```

**Comportamento Comum:**
- Atualização em tempo real durante busca em lote
- Indicadores de loading por item
- Scroll automático para novos resultados
- Mensagens de erro inline quando aplicável

#### RF-009: Sistema de Notificações
**Prioridade:** MÉDIA  
**Descrição:** Feedback visual do progresso e status.

**Estados - Busca Individual:**
- "Analisando referência..."
- "Buscando em X fontes..." (com contador)
- "Encontrados Y resultados"
- "Erro ao buscar em [fonte]" (warning, não bloqueia)
- "Nenhum artigo encontrado com essa referência"

**Estados - Busca em Lote:**
- "Analisando arquivo .bib..."
- "X referências detectadas, Y válidas"
- "Processando lote: X/Y completos"
- Barra de progresso visual (0-100%)
- "Processamento concluído: X encontrados, Y falharam"
- Tempo estimado restante (após 5 primeiros)
- Opção "Pausar processamento" / "Cancelar"

**Notificações de Ação:**
- "✓ PDF baixado com sucesso"
- "✓ Citação copiada"
- "✓ Resultados exportados"
- "✓ Arquivo salvo: nome-do-arquivo.csv"
- "⚠️ Alguns PDFs falharam ao baixar (X/Y)"

**Critérios de Aceitação:**
- Feedback visual em <500ms
- Indicadores de progresso para operações >2s
- Erros não impedem exibição de resultados parciais
- Notificações desaparecem após 3s (ou clique para fechar)
- Barra de progresso atualiza a cada item processado

#### RF-010: Download de PDFs
**Prioridade:** ALTA  
**Descrição:** Facilitar acesso ao arquivo do artigo.

**Funcionalidades:**
- Botão "Baixar PDF" abre em nova aba (permite salvar)
- Nome do arquivo sugerido: `Autor_Ano_Titulo.pdf`
- Indicador de tamanho do arquivo (quando disponível)
- Preview rápido antes de baixar (iframe, opcional)

---

### 2.4 Funcionalidades Complementares

#### RF-011: Exportação de Citações
**Prioridade:** MÉDIA  
**Descrição:** Copiar referência em diversos formatos.

**Formatos:**
- ABNT
- APA 7ª edição
- Vancouver
- BibTeX
- RIS (para gerenciadores de referência)

**Critérios de Aceitação:**
- Botão "Copiar citação" com dropdown de formatos
- Feedback visual ao copiar
- Formatação correta segundo normas atualizadas

#### RF-012: Upload e Processamento de Arquivo BibTeX
**Prioridade:** ALTA  
**Descrição:** Permitir upload de arquivos .bib para processamento em lote de múltiplas referências.

**Funcionalidades:**
- Drag & drop ou seleção de arquivo .bib
- Parser de formato BibTeX
- Extração de todas as entradas do arquivo
- Validação de formato e estrutura
- Preview das referências detectadas antes de processar

**Tipos de entrada BibTeX suportados:**
- @article
- @inproceedings
- @book
- @incollection
- @phdthesis
- @misc
- Outros tipos comuns

**Campos extraídos:**
- title (obrigatório)
- author
- year
- doi (quando disponível)
- journal/booktitle
- volume, number, pages
- url

**Critérios de Aceitação:**
- Aceita arquivos até 5MB
- Parser tolera variações de formatação BibTeX
- Identifica e reporta entradas com erros
- Permite selecionar quais entradas processar
- Exibe contador: "X de Y referências válidas"

#### RF-013: Busca em Lote
**Prioridade:** ALTA  
**Descrição:** Processar múltiplas referências de uma vez e consolidar resultados.

**Funcionalidades:**
- Processa até 50 referências por lote
- Busca paralela com controle de concorrência (max 5 simultâneas)
- Barra de progresso com status individual
- Pausa/retomada de processamento
- Exportação de resultados completos

**Interface:**
- Tabela com colunas:
  - Checkbox de seleção
  - Título da referência
  - Status: ⏳ Aguardando | 🔍 Buscando | ✅ Encontrado | ❌ Não encontrado
  - Ações (quando encontrado)
- Filtros: Mostrar apenas [Encontrados | Não encontrados | Todos]
- Ações em lote:
  - "Baixar todos os PDFs disponíveis" (abre em tabs)
  - "Exportar resultados" (CSV/JSON)
  - "Tentar novamente" (apenas falhados)

**Estratégia de Processamento:**
1. Usuário faz upload do .bib
2. Sistema exibe preview das N referências
3. Usuário pode desmarcar referências indesejadas
4. Clica "Processar Lote"
5. Sistema processa 5 referências por vez (rate limiting)
6. Atualiza status em tempo real
7. Ao concluir, exibe sumário:
   - X PDFs disponíveis
   - Y apenas links
   - Z não encontrados
8. Permite ações individuais ou em lote

**Critérios de Aceitação:**
- Processa 50 refs em <5 minutos (média 6s/ref)
- Não trava interface durante processamento
- Permite cancelar processamento a qualquer momento
- Salva progresso (pode fechar e retornar)
- Relatório final exportável

#### RF-014: Exportação de Resultados em Lote
**Prioridade:** MÉDIA  
**Descrição:** Exportar resultados de processamento em lote em diversos formatos.

**Formatos de Exportação:**

1. **CSV** - Para análise em planilhas
   - Colunas: Título, Autores, Ano, DOI, Status, Link PDF, Fonte
   
2. **JSON** - Para processamento programático
   - Estrutura completa com todos os metadados
   
3. **BibTeX atualizado** - Arquivo .bib enriquecido
   - Adiciona campos encontrados (DOI, URL, etc.)
   - Marca entradas não encontradas com comentário
   
4. **Markdown** - Relatório legível
   - Lista formatada com links clicáveis
   - Separado por status (encontrados/não encontrados)

**Funcionalidades:**
- Botão "Exportar Resultados" com dropdown de formatos
- Inclui timestamp no nome do arquivo
- Opção de incluir apenas encontrados ou todos
- Preview antes de exportar

**Critérios de Aceitação:**
- Exportação instantânea (<1s para 50 refs)
- Formato válido e importável
- Nome de arquivo descritivo

#### RF-015: Histórico de Buscas
**Prioridade:** BAIXA  
**Descrição:** Armazenar buscas recentes localmente.

**Características:**
- Últimas 20 buscas individuais no localStorage
- Últimos 5 lotes processados (com data/hora)
- Clique para re-executar busca/lote
- Opção de limpar histórico
- Não armazena dados sensíveis
- Histórico de lotes mostra sumário (X de Y encontrados)

#### RF-016: Favoritos/Salvos
**Prioridade:** BAIXA  
**Descrição:** Salvar artigos para referência posterior.

**Características:**
- Botão "⭐ Salvar" em cada resultado
- Lista de salvos acessível
- Exportar lista de salvos como BibTeX
- Armazenamento local (não requer conta)
- Funciona tanto para buscas individuais quanto lotes

---

## 3. Requisitos Não-Funcionais

### 3.1 Performance

**RNF-001: Tempo de Resposta**
- Detecção de formato: <200ms
- Busca completa em todas as fontes: <10s
- Renderização de resultados: <500ms
- Interface responsiva durante operações assíncronas

**RNF-002: Escalabilidade**
- Suportar 100 buscas simultâneas
- Cache de resultados por 24h (mesma referência)
- Rate limiting para proteger APIs externas

### 3.2 Usabilidade

**RNF-003: Acessibilidade**
- WCAG 2.1 Level AA compliance
- Navegação completa por teclado
- Screen reader friendly
- Contraste adequado (mínimo 4.5:1)

**RNF-004: Responsividade**
- Design mobile-first
- Funcional em telas 320px+
- Touch-friendly (botões min 44x44px)

### 3.3 Compatibilidade

**RNF-005: Navegadores**
- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Mobile browsers (iOS Safari, Chrome Android)

**RNF-006: Internacionalização**
- Interface em português brasileiro (Fase 1)
- Preparado para inglês e espanhol (Fase 2)
- Suporte a caracteres UTF-8

### 3.4 Segurança

**RNF-007: Privacidade**
- Nenhum dado de busca enviado para servidor próprio
- Comunicação com APIs externas via HTTPS
- Não coletar informações pessoais
- Dados locais apenas (localStorage)

**RNF-008: Conformidade Legal**
- Respeitar robots.txt e Terms of Service das fontes
- Rate limiting para não sobrecarregar APIs
- Disclaimer sobre direitos autorais dos PDFs

### 3.5 Confiabilidade

**RNF-009: Disponibilidade**
- Graceful degradation se uma fonte falhar
- Não bloquear interface por erro em API externa
- Mensagens claras sobre falhas

**RNF-010: Precisão**
- Taxa de falsos positivos <5%
- Validação de links antes de exibir
- Indicação clara de grau de confiança

---

## 4. Arquitetura Técnica

### 4.1 Stack Tecnológico

**Frontend:**
- React 18+ (Hooks, Context API)
- Vite para build e dev server
- Tailwind CSS para estilização
- Lucide React para ícones

**Bibliotecas Específicas:**
- `citation-js` - Parser de citações bibliográficas
- `bibtex-parse` ou `@retorquere/bibtex-parser` - Parser de arquivos .bib
- `axios` - HTTP client para APIs
- `pdf-lib` ou `react-pdf` - Preview de PDFs (opcional)
- `dompurify` - Sanitização de HTML
- `papaparse` - Exportação de CSV
- `file-saver` - Download de arquivos gerados

**APIs Externas:**
- CrossRef REST API (sem auth necessária)
- Unpaywall API (email em query params)
- SciELO API (REST pública)
- PubMed E-utilities (sem auth)
- arXiv API (OAI-PMH)
- Semantic Scholar API (rate limited, sem auth)

### 4.2 Arquitetura de Componentes

```
src/
├── components/
│   ├── SearchInput.jsx          # Campo de busca com parser
│   ├── BatchUpload.jsx          # Componente de upload .bib
│   ├── ResultsList.jsx          # Lista de resultados (individual)
│   ├── ResultsTable.jsx         # Tabela de resultados (lote)
│   ├── ResultCard.jsx           # Card individual de artigo
│   ├── BatchProgressBar.jsx    # Barra de progresso lote
│   ├── BatchSummary.jsx        # Sumário de processamento
│   ├── LoadingIndicator.jsx    # Estados de carregamento
│   ├── ErrorMessage.jsx        # Tratamento de erros
│   ├── ExportMenu.jsx          # Menu de exportação
│   └── HistoryPanel.jsx        # Histórico de buscas
├── services/
│   ├── referenceParser.js      # Lógica de parsing
│   ├── bibtexParser.js         # Parser específico para .bib
│   ├── batchProcessor.js       # Orquestração de lote
│   ├── exportService.js        # Geração de exports (CSV, JSON, etc)
│   ├── apiClients/
│   │   ├── crossref.js
│   │   ├── unpaywall.js
│   │   ├── scielo.js
│   │   ├── pubmed.js
│   │   └── arxiv.js
│   └── searchOrchestrator.js   # Coordena buscas paralelas
├── utils/
│   ├── citationFormatter.js    # Formatação de citações
│   ├── pdfValidator.js         # Valida links de PDF
│   ├── fileHandlers.js         # Upload e download de arquivos
│   └── storage.js              # LocalStorage wrapper
└── hooks/
    ├── useSearch.js            # Hook customizado para busca
    ├── useBatchSearch.js       # Hook para busca em lote
    └── useHistory.js           # Gerenciamento de histórico
```

### 4.3 Fluxo de Dados

**Busca Individual:**
```
1. Usuário cola referência
   ↓
2. SearchInput aciona referenceParser
   ↓
3. Parser extrai metadados estruturados
   ↓
4. searchOrchestrator dispara buscas paralelas em todas as fontes
   ↓
5. Cada apiClient retorna array de resultados
   ↓
6. Consolidação e deduplicação de resultados
   ↓
7. Validação de links de PDF
   ↓
8. Ordenação por disponibilidade e relevância
   ↓
9. ResultsList renderiza cards
   ↓
10. Usuário interage (baixa PDF, copia citação, etc.)
```

**Busca em Lote:**
```
1. Usuário faz upload de arquivo .bib
   ↓
2. BatchUpload aciona bibtexParser
   ↓
3. Parser extrai todas as entradas com metadados
   ↓
4. Exibe tabela de preview com checkboxes
   ↓
5. Usuário confirma seleção e clica "Processar Lote"
   ↓
6. batchProcessor inicia processamento
   ↓
7. Para cada referência (5 paralelas, fila para resto):
   ├─ searchOrchestrator busca em todas as fontes
   ├─ Atualiza status na tabela em tempo real
   ├─ Incrementa barra de progresso
   └─ Armazena resultado em array
   ↓
8. Ao concluir todas:
   ├─ Exibe BatchSummary com estatísticas
   ├─ ResultsTable renderiza todos os resultados
   └─ Habilita ações em lote
   ↓
9. Usuário pode:
   ├─ Filtrar/ordenar resultados
   ├─ Baixar PDFs selecionados
   ├─ Exportar resultados
   └─ Salvar lote para referência
```

**Exportação de Resultados:**
```
1. Usuário clica "Exportar" e escolhe formato
   ↓
2. exportService processa dados:
   ├─ CSV: converte para tabela (papaparse)
   ├─ JSON: serializa array de resultados
   ├─ BibTeX: reconstrói com campos adicionados
   └─ Markdown: formata lista humanizada
   ↓
3. Gera Blob com conteúdo
   ↓
4. file-saver dispara download
   ↓
5. Notificação de sucesso
```

---

## 5. Casos de Uso Detalhados

### UC-001: Busca por DOI
**Ator:** Pesquisador  
**Pré-condição:** Usuário possui DOI do artigo  
**Fluxo Principal:**
1. Usuário cola DOI no campo de busca: `10.1590/S0102-311X2008000800003`
2. Sistema detecta formato DOI
3. Sistema busca via CrossRef API
4. CrossRef retorna metadados completos
5. Sistema consulta Unpaywall para PDF aberto
6. Unpaywall retorna link do SciELO
7. Sistema valida disponibilidade do PDF
8. Exibe resultado com botão "Baixar PDF"
9. Usuário clica e PDF abre em nova aba

**Fluxo Alternativo 1:** PDF não disponível
- 6a. Unpaywall não encontra versão aberta
- 7a. Sistema marca como "Acesso Restrito"
- 8a. Exibe link para página do publisher

**Pós-condição:** Artigo localizado e acessível

### UC-002: Busca por Citação ABNT
**Ator:** Estudante  
**Pré-condição:** Usuário possui citação completa  
**Fluxo Principal:**
1. Usuário cola citação: `SILVA, J. A. Educação e tecnologia. Revista Brasileira de Educação, v. 25, n. 3, p. 45-67, 2020.`
2. Sistema detecta formato ABNT
3. Parser extrai: título="Educação e tecnologia", autor="Silva, J. A.", ano=2020
4. Sistema busca em múltiplas fontes pelo título
5. SciELO retorna 1 resultado com match exato
6. Sistema valida que autor e ano correspondem
7. Exibe resultado com alta confiança
8. Usuário baixa PDF disponível

**Fluxo Alternativo 1:** Múltiplos resultados
- 5a. Retornam 5 artigos com títulos similares
- 6a. Sistema filtra por autor e ano
- 7a. Exibe 2 resultados mais prováveis
- 8a. Usuário escolhe o correto

**Pós-condição:** Artigo correto identificado

### UC-004: Processamento em Lote de Arquivo BibTeX
**Ator:** Pesquisador conduzindo revisão sistemática  
**Pré-condição:** Usuário possui arquivo .bib com 30 referências de sua pesquisa  
**Fluxo Principal:**
1. Usuário acessa modo "Lote (.bib)" na interface
2. Faz drag & drop do arquivo `referencias_revisao.bib`
3. Sistema valida arquivo (2.3 MB, válido)
4. Parser detecta 30 entradas BibTeX
5. Sistema exibe tabela de preview:
   ```
   ☑️ [Título 1] | Autor A et al. | 2023
   ☑️ [Título 2] | Autor B et al. | 2022
   ☑️ [Título 3] | Autor C et al. | 2024
   ... (30 entradas)
   ```
6. Contador mostra "30 referências selecionadas"
7. Usuário clica "Processar Lote"
8. Sistema inicia processamento:
   - Barra de progresso: "Processando: 5/30 (17%)"
   - Status individual atualiza em tempo real
9. Após 4 minutos, processamento completa
10. Sistema exibe sumário:
    ```
    ✅ 24 artigos encontrados (80%)
    🟢 19 com PDF disponível (79%)
    🟡 5 apenas link
    ❌ 6 não encontrados (20%)
    ```
11. Tabela mostra todos os resultados com status
12. Usuário filtra "Apenas com PDF"
13. Seleciona os 19 artigos
14. Clica "Baixar PDFs Selecionados"
15. PDFs abrem em 19 novas abas
16. Usuário clica "Exportar Resultados" → CSV
17. Arquivo `resultados_lote_20251210_143055.csv` é baixado

**Fluxo Alternativo 1:** Arquivo com entradas inválidas
- 4a. Parser detecta 30 entradas, mas 3 estão malformadas
- 5a. Sistema exibe warning: "3 entradas com erros (clique para detalhes)"
- 6a. Usuário visualiza quais entradas falharam e porquê
- 7a. Contador mostra "27 referências válidas, 3 ignoradas"
- 8a. Usuário prossegue com as 27 válidas

**Fluxo Alternativo 2:** Processamento interrompido
- 8a. Após processar 15/30, usuário clica "Pausar"
- 9a. Sistema congela processamento
- 10a. Usuário pode "Retomar" ou "Cancelar"
- 11a. Ao retomar, continua de onde parou
- 12a. Resultados parciais já ficam visíveis

**Pós-condição:** Usuário tem arquivo CSV com resultados e PDFs salvos localmente

### UC-005: Exportação de Resultados Enriquecidos
**Ator:** Pesquisador  
**Pré-condição:** Lote de 30 referências já processado  
**Fluxo Principal:**
1. Na tela de resultados do lote, usuário clica "Exportar Resultados"
2. Dropdown mostra opções:
   - CSV
   - JSON
   - BibTeX Atualizado
   - Relatório Markdown
3. Usuário escolhe "BibTeX Atualizado"
4. Sistema reconstrói arquivo .bib original:
   - Adiciona campo `doi = {...}` onde encontrado
   - Adiciona campo `url = {...}` com link do PDF
   - Adiciona campo `note = {PDF disponível em: ...}`
   - Marca entradas não encontradas: `% NÃO ENCONTRADO`
5. Arquivo `referencias_revisao_enriquecido.bib` é gerado
6. Sistema dispara download
7. Notificação: "✓ Arquivo exportado: referencias_revisao_enriquecido.bib"
8. Usuário importa arquivo enriquecido no Zotero
9. Todos os metadados e links estão atualizados

**Fluxo Alternativo 1:** Exportação apenas dos encontrados
- 3a. Usuário marca checkbox "Apenas artigos encontrados"
- 5a. Sistema exporta apenas 24 entradas (exclui 6 não encontrados)

**Pós-condição:** Arquivo .bib enriquecido com metadados adicionais disponível

---

## 6. Roadmap de Desenvolvimento

### Fase 1 - MVP (4 semanas)
**Semana 1-2:**
- Setup do projeto React + Vite
- Componente de busca individual básico
- Parser para DOI e títulos simples
- Integração com CrossRef e Unpaywall

**Semana 3:**
- Integração com SciELO e arXiv
- Componente de exibição de resultados
- Sistema básico de detecção de PDFs

**Semana 4:**
- Testes de integração
- Ajustes de UI/UX
- Deploy em Vercel/Netlify
- Documentação básica

**Entregáveis MVP:**
- Busca por DOI funcional
- Busca por título funcional  
- Integração com 4 fontes principais
- Interface responsiva básica

### Fase 2 - Processamento em Lote (4 semanas)
**Semana 5-6:**
- Componente de upload de arquivo .bib
- Parser robusto de BibTeX
- Sistema de processamento em lote
- Barra de progresso e controles (pausar/cancelar)

**Semana 7:**
- Interface de resultados em tabela
- Filtros e ordenação
- Ações em lote (baixar todos, exportar)
- Sistema de exportação (CSV, JSON, Markdown)

**Semana 8:**
- Exportação de BibTeX enriquecido
- Testes com arquivos .bib reais
- Otimização de performance (rate limiting)
- Documentação de funcionalidades de lote

**Entregáveis Fase 2:**
- Upload e processamento de arquivos .bib
- Processamento paralelo de até 50 referências
- Exportação em 4 formatos
- Interface de gerenciamento de lote completa

### Fase 3 - Melhorias e Features Avançadas (3 semanas)
**Semana 9-10:**
- Parser robusto para citações ABNT/APA
- Integração com PubMed
- Sistema de cache avançado
- Histórico de buscas individuais e lotes

**Semana 11:**
- Sistema de favoritos
- Melhorias de performance
- Testes com usuários reais (UFMA)
- Ajustes baseados em feedback

---

## 7. Métricas de Sucesso

### 7.1 Métricas de Produto
- **Taxa de sucesso de busca (individual):** >70% encontram PDF ou link
- **Taxa de sucesso de busca (lote):** >65% encontram PDF ou link por entrada
- **Tempo médio de busca individual:** <30 segundos do input ao resultado
- **Tempo médio de processamento em lote:** <6 segundos por referência
- **Taxa de satisfação:** >4.0/5.0 em pesquisa de usuários
- **Precisão do parser BibTeX:** >95% de entradas válidas processadas corretamente
- **Precisão do parser de citações:** >90% de detecção correta de formato

### 7.2 Métricas de Uso
- **Usuários ativos semanais:** Meta 100 em 3 meses
- **Buscas individuais por usuário:** Média 5+ por sessão
- **Lotes processados por usuário:** Média 2+ por mês
- **Tamanho médio de lote:** 20-50 referências
- **Taxa de retorno:** >40% retornam dentro de 7 dias
- **Taxa de conclusão (individual):** >80% completam busca até resultado
- **Taxa de conclusão (lote):** >90% completam processamento (não cancelam)
- **Taxa de exportação:** >60% dos lotes são exportados

### 7.3 Métricas Técnicas
- **Uptime:** >99.5%
- **Tempo de resposta individual (p95):** <8s
- **Tempo de processamento de lote 50 refs (p95):** <6min
- **Taxa de erro de APIs:** <5%
- **Performance Lighthouse:** >90 em todas as categorias
- **Taxa de upload bem-sucedido de .bib:** >98%

---

## 8. Riscos e Mitigações

### 8.1 Riscos Técnicos

**R-001: APIs externas instáveis**
- **Probabilidade:** Alta
- **Impacto:** Médio
- **Mitigação:** 
  - Implementar retry logic com backoff exponencial
  - Cache agressivo de resultados
  - Graceful degradation (continua com outras fontes)
  - Monitoramento de uptime das APIs

**R-002: Rate limiting de APIs**
- **Probabilidade:** Média
- **Impacto:** Alto
- **Mitigação:**
  - Implementar rate limiting no client
  - Queue de requisições
  - Informar usuário sobre limites
  - Considerar proxy/backend para pooling

**R-003: Parsing impreciso**
- **Probabilidade:** Média
- **Impacto:** Médio
- **Mitigação:**
  - Feedback loop para melhorar regex
  - Permitir edição manual de campos
  - Múltiplas estratégias de parsing
  - Logs de casos que falharam

### 8.2 Riscos Legais

**R-004: Violação de ToS das fontes**
- **Probabilidade:** Baixa
- **Impacto:** Crítico
- **Mitigação:**
  - Revisar ToS de cada fonte cuidadosamente
  - Usar apenas APIs oficiais (evitar scraping)
  - Respeitar rate limits
  - Incluir disclaimer sobre uso responsável

**R-005: Questões de copyright dos PDFs**
- **Probabilidade:** Média
- **Impacto:** Médio
- **Mitigação:**
  - Nunca hospedar PDFs
  - Apenas links para fontes legítimas
  - Priorizar acesso aberto
  - Disclaimer claro sobre direitos autorais

### 8.3 Riscos de Produto

**R-006: Baixa taxa de sucesso frustra usuários**
- **Probabilidade:** Média
- **Impacto:** Alto
- **Mitigação:**
  - Expectativas claras (não garante 100%)
  - Sugestões quando não encontra
  - Facilitar busca manual como fallback
  - Melhorar continuamente baseado em feedback
  - Para lotes, mostrar estatísticas realistas (65-80% de sucesso)

**R-007: Processamento de lotes muito lento**
- **Probabilidade:** Média
- **Impacto:** Alto
- **Mitigação:**
  - Rate limiting inteligente (5 paralelas)
  - Cache agressivo para evitar re-busca
  - Opção de pausar/retomar
  - Processamento em background (Web Workers)
  - Feedback claro de progresso e tempo estimado

**R-008: Arquivos .bib malformados causam erros**
- **Probabilidade:** Alta
- **Impacto:** Médio
- **Mitigação:**
  - Parser robusto e tolerante a erros
  - Validação linha por linha
  - Reportar entradas problemáticas individualmente
  - Continuar processando entradas válidas
  - Oferecer correção manual inline

---

## 9. Considerações de Acessibilidade

### 9.1 Navegação por Teclado
- Tab navega entre elementos focáveis
- Enter aciona busca e downloads
- Esc fecha modais e limpa busca
- Atalhos: Ctrl+K para focar busca

### 9.2 Screen Readers
- Landmarks semânticos (header, main, nav)
- ARIA labels descritivos
- Anúncio de mudanças dinâmicas (resultados carregados)
- Alt text para ícones informativos

### 9.3 Visual
- Contraste mínimo 4.5:1 (texto) e 3:1 (UI)
- Tamanhos de fonte ajustáveis
- Não depender apenas de cor (usar ícones)
- Foco visível claro

### 9.4 Motor
- Alvos de toque ≥44x44px
- Espaçamento adequado entre elementos
- Sem requisito de gestos complexos

---

## 10. Questões em Aberto

### 10.1 Decisões Pendentes
1. **Backend necessário?**
   - Prós: Rate limiting melhor, cache centralizado, analytics, processamento de lotes mais robusto
   - Contras: Custo, complexidade, manutenção
   - **Decisão:** Começar 100% client-side, avaliar necessidade após MVP. Para lotes grandes (>50), considerar backend na Fase 3.

2. **Limite de tamanho de lote?**
   - 50 refs parece razoável para client-side
   - Acima disso, performance pode degradar
   - **Decisão:** Limite de 50 no MVP, avisar usuário para dividir lotes maiores. Avaliar aumento para 100 com otimizações.

3. **Monetização futura?**
   - Gratuito com limites?
   - Modelo freemium?
   - Patrocínio institucional (UFMA)?
   - **Decisão:** Gratuito no MVP, avaliar sustentabilidade depois

4. **Integração com gerenciadores de referência?**
   - Zotero, Mendeley connector
   - Exportação direta para esses apps
   - **Decisão:** Exportação BibTeX na Fase 2, integração direta Fase 4 (futura)

5. **Armazenamento de lotes processados?**
   - LocalStorage (limite ~5MB)
   - IndexedDB (sem limite prático)
   - **Decisão:** IndexedDB para armazenar histórico de lotes (até 10 últimos)

### 10.2 Pesquisa Necessária
- Quais repositórios institucionais brasileiros têm APIs?
- Google Scholar permite uso programático? (resposta: não oficialmente, scraping é contra ToS)
- Existem bibliotecas prontas de parsing de citações em PT-BR?
- Qual a melhor biblioteca Node.js para parsing robusto de BibTeX?
- Performance de Web Workers para processamento paralelo de lotes?
- Limites práticos de IndexedDB em diferentes navegadores?
- APIs com rate limit que permitem batch requests?

---

## 11. Apêndices

### 11.1 Glossário
- **DOI:** Digital Object Identifier, identificador único de publicações
- **OA:** Open Access, acesso aberto
- **API:** Application Programming Interface
- **Parser:** Componente que analisa e extrai estrutura de texto
- **Rate Limiting:** Limite de requisições por período de tempo
- **Graceful Degradation:** Sistema continua funcionando com funcionalidades reduzidas
- **BibTeX:** Formato de arquivo para referências bibliográficas
- **Batch Processing:** Processamento em lote
- **IndexedDB:** Sistema de armazenamento local no navegador para grandes volumes de dados
- **Web Workers:** API do navegador para processamento em segundo plano

### 11.2 Referências
- CrossRef API: https://www.crossref.org/documentation/retrieve-metadata/rest-api/
- Unpaywall API: https://unpaywall.org/products/api
- SciELO API: https://api.scielo.org/
- PubMed E-utilities: https://www.ncbi.nlm.nih.gov/books/NBK25501/
- arXiv API: https://arxiv.org/help/api/
- Citation.js: https://citation.js.org/
- BibTeX Format: http://www.bibtex.org/Format/
- @retorquere/bibtex-parser: https://github.com/retorquere/bibtex-parser
- PapaParse (CSV): https://www.papaparse.com/
- FileSaver.js: https://github.com/eligrey/FileSaver.js/

### 11.3 Contato
**Product Owner:** Alana - UFMA  
**Feedback:** Via GitHub Issues (após publicação do repositório)

---

**Changelog:**
- v1.0 (Dez 2025): Versão inicial do PRD
- v1.1 (Dez 2025): Adicionado processamento em lote via upload de arquivos .bib
  - RF-012 a RF-016: Funcionalidades de lote e exportação
  - Atualização de RF-007 a RF-009: Suporte para modo lote
  - Novos casos de uso UC-004 e UC-005
  - Roadmap expandido para 11 semanas (3 fases)
  - Métricas específicas para processamento em lote
  - Novos riscos e decisões pendentes relacionadas a lotes
  - Arquitetura atualizada com componentes de lote