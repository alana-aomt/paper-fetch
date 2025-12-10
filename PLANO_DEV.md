# Plano de Desenvolvimento - PaperFetch

**Início:** 10 de Dezembro de 2025
**Responsável:** Alana - UFMA

---

## Fase 1 - MVP (4 semanas)

### Semana 1: Setup e Busca Individual Básica

#### 1.1 Configuração Inicial do Projeto
- [x] Criar projeto React com Vite
- [x] Configurar Tailwind CSS
- [x] Configurar estrutura de pastas (src/components, src/services, src/utils, src/hooks)
- [x] Instalar dependências principais (axios, citation-js, lucide-react)
- [ ] Configurar ESLint e Prettier

#### 1.2 Parser de Referências - DOI e Título
- [ ] Implementar `referenceParser.js` básico
- [ ] Detectar formato DOI (regex)
- [ ] Detectar título simples
- [ ] Extrair metadados básicos
- [ ] Criar testes unitários para parser

#### 1.3 Interface de Busca Individual
- [ ] Criar componente `SearchInput.jsx`
- [ ] Campo de texto com placeholder de exemplos
- [ ] Botão "Buscar Artigo" com estado de loading
- [ ] Indicador de formato detectado (badge)
- [ ] Layout responsivo básico

### Semana 2: Integração com APIs Principais

#### 2.1 API Clients - CrossRef e Unpaywall
- [ ] Implementar `apiClients/crossref.js`
  - [ ] Função de busca por DOI
  - [ ] Função de busca por título
  - [ ] Tratamento de erros
- [ ] Implementar `apiClients/unpaywall.js`
  - [ ] Busca por DOI para PDF aberto
  - [ ] Validação de disponibilidade
- [ ] Configurar axios interceptors para retry logic
- [ ] Implementar timeout de 10s

#### 2.2 Search Orchestrator
- [ ] Criar `searchOrchestrator.js`
- [ ] Implementar busca paralela com Promise.allSettled
- [ ] Consolidação de resultados
- [ ] Deduplicação por DOI/título
- [ ] Ordenação por disponibilidade de PDF

#### 2.3 Hook de Busca
- [ ] Criar `useSearch.js`
- [ ] Estados: loading, results, error
- [ ] Integração com parser e orchestrator
- [ ] Cache de resultados (localStorage)

### Semana 3: Mais Fontes e Exibição de Resultados

#### 3.1 API Clients - SciELO e arXiv
- [ ] Implementar `apiClients/scielo.js`
  - [ ] Busca por título
  - [ ] Busca por DOI
  - [ ] Extração de metadados
- [ ] Implementar `apiClients/arxiv.js`
  - [ ] Busca por título
  - [ ] Busca por arXiv ID
  - [ ] Rate limiting (1 req/3s)
- [ ] Adicionar ambos ao orchestrator

#### 3.2 Componentes de Resultado
- [ ] Criar `ResultCard.jsx`
  - [ ] Exibição de título, autores, ano
  - [ ] Badge de disponibilidade (🟢 PDF / 🟡 Link / 🔴 Restrito)
  - [ ] Botões: Baixar PDF, Abrir no site
- [ ] Criar `ResultsList.jsx`
  - [ ] Lista de ResultCards
  - [ ] Mensagem "Nenhum resultado encontrado"
  - [ ] Indicadores de loading

#### 3.3 Validação de PDFs
- [ ] Implementar `pdfValidator.js`
- [ ] HEAD request para verificar disponibilidade
- [ ] Classificação de tipo de acesso (Gold, Green, Hybrid)
- [ ] Timeout de 3s por validação

### Semana 4: Testes, UI/UX e Deploy

#### 4.1 Sistema de Notificações
- [ ] Criar componente `LoadingIndicator.jsx`
- [ ] Criar componente `ErrorMessage.jsx`
- [ ] Feedback visual durante busca
- [ ] Mensagens de erro amigáveis

#### 4.2 Melhorias de UI/UX
- [ ] Ajustes de responsividade mobile
- [ ] Animações de loading
- [ ] Estados vazios (empty states)
- [ ] Acessibilidade: navegação por teclado
- [ ] Acessibilidade: ARIA labels

#### 4.3 Testes e Deploy
- [ ] Configurar Vitest
- [ ] Testes de integração para parsers
- [ ] Testes de integração para API clients
- [ ] Build de produção
- [ ] Deploy na Vercel/Netlify
- [ ] Documentação do README

---

## Fase 2 - Processamento em Lote (4 semanas)

### Semana 5: Upload e Parser BibTeX

#### 5.1 Componente de Upload
- [ ] Criar `BatchUpload.jsx`
- [ ] Área de drag & drop
- [ ] Botão "Selecionar arquivo"
- [ ] Validação de tipo (.bib)
- [ ] Indicador de tamanho do arquivo

#### 5.2 Parser BibTeX
- [ ] Instalar `@retorquere/bibtex-parser`
- [ ] Implementar `bibtexParser.js`
- [ ] Extrair todas as entradas
- [ ] Suporte para tipos: @article, @inproceedings, @book, @misc, etc.
- [ ] Validação de entradas
- [ ] Tratamento de erros por entrada

#### 5.3 Preview de Referências
- [ ] Tabela de preview com checkboxes
- [ ] Colunas: checkbox, título, autor, ano
- [ ] Contador de referências selecionadas
- [ ] Botão "Selecionar Todos/Nenhum"
- [ ] Indicador de entradas inválidas

### Semana 6: Processamento em Lote

#### 6.1 Batch Processor
- [ ] Criar `batchProcessor.js`
- [ ] Implementar fila de processamento
- [ ] Controle de concorrência (max 5 paralelas)
- [ ] Rate limiting por API
- [ ] Armazenamento de progresso (IndexedDB)

#### 6.2 Hook de Busca em Lote
- [ ] Criar `useBatchSearch.js`
- [ ] Estados: processing, progress, results, errors
- [ ] Funções: start, pause, resume, cancel
- [ ] Atualização em tempo real

#### 6.3 Componentes de Progresso
- [ ] Criar `BatchProgressBar.jsx`
  - [ ] Barra visual (0-100%)
  - [ ] Contador: "X/Y processados"
  - [ ] Tempo estimado restante
- [ ] Criar `BatchSummary.jsx`
  - [ ] Estatísticas finais
  - [ ] Gráfico de status (encontrados/não encontrados)

### Semana 7: Interface de Resultados em Lote

#### 7.1 Tabela de Resultados
- [ ] Criar `ResultsTable.jsx`
- [ ] Colunas: checkbox, título, autores, ano, status, ações
- [ ] Ícones de status: ✅ PDF / 🔗 Link / ❌ Não encontrado / ⏳ Processando
- [ ] Ações rápidas: baixar, abrir link, copiar citação

#### 7.2 Filtros e Ordenação
- [ ] Dropdown de filtros: Todos / Apenas com PDF / Não encontrados
- [ ] Dropdown de ordenação: Ordem original / Título / Ano / Status
- [ ] Busca dentro dos resultados
- [ ] Paginação (20 por página)

#### 7.3 Ações em Lote
- [ ] Botão "Baixar PDFs Selecionados"
- [ ] Abrir múltiplas tabs com PDFs
- [ ] Feedback de progresso de download

### Semana 8: Exportação de Resultados

#### 8.1 Export Service
- [ ] Criar `exportService.js`
- [ ] Implementar exportação CSV (papaparse)
- [ ] Implementar exportação JSON
- [ ] Implementar exportação Markdown
- [ ] Implementar exportação BibTeX enriquecido

#### 8.2 Componente de Exportação
- [ ] Criar `ExportMenu.jsx`
- [ ] Dropdown com 4 formatos
- [ ] Preview antes de exportar
- [ ] Nome de arquivo com timestamp
- [ ] Opção: incluir apenas encontrados ou todos

#### 8.3 File Handlers
- [ ] Instalar `file-saver`
- [ ] Implementar `fileHandlers.js`
- [ ] Download de arquivos gerados
- [ ] Tratamento de erros de download

#### 8.4 Otimização e Testes
- [ ] Implementar cache de resultados (24h)
- [ ] Otimizar rate limiting
- [ ] Testes com arquivos .bib reais (10, 30, 50 refs)
- [ ] Medição de performance
- [ ] Atualizar documentação

---

## Fase 3 - Melhorias e Features Avançadas (3 semanas)

### Semana 9: Parser Robusto e PubMed

#### 9.1 Parser de Citações ABNT/APA
- [ ] Regex para citação ABNT
- [ ] Regex para citação APA
- [ ] Extração de autor, título, ano, revista
- [ ] Normalização de nomes de autores
- [ ] Testes com citações reais

#### 9.2 Integração PubMed
- [ ] Implementar `apiClients/pubmed.js`
- [ ] Busca por PMID
- [ ] Busca por título
- [ ] Rate limiting (3 req/s)
- [ ] Adicionar ao orchestrator

#### 9.3 Semantic Scholar
- [ ] Implementar `apiClients/semanticScholar.js`
- [ ] Busca semântica por título
- [ ] Rate limiting
- [ ] Fallback para outras fontes

### Semana 10: Histórico e Cache

#### 10.1 Sistema de Cache Avançado
- [ ] Implementar `storage.js` com IndexedDB
- [ ] Cache de resultados por 24h
- [ ] Cache de metadados de APIs
- [ ] Limpeza automática de cache antigo

#### 10.2 Histórico de Buscas
- [ ] Criar `useHistory.js`
- [ ] Armazenar últimas 20 buscas individuais
- [ ] Armazenar últimos 5 lotes
- [ ] Componente `HistoryPanel.jsx`
- [ ] Clique para re-executar busca

#### 10.3 Sistema de Favoritos
- [ ] Armazenar artigos salvos (localStorage)
- [ ] Botão "⭐ Salvar" em ResultCard
- [ ] Lista de favoritos
- [ ] Exportar favoritos como BibTeX

### Semana 11: Testes com Usuários e Ajustes

#### 11.1 Funcionalidades de Citação
- [ ] Instalar biblioteca de formatação de citações
- [ ] Implementar `citationFormatter.js`
- [ ] Formatos: ABNT, APA, Vancouver, BibTeX, RIS
- [ ] Botão "Copiar citação" com dropdown
- [ ] Feedback visual ao copiar

#### 11.2 Melhorias de Performance
- [ ] Web Workers para processamento em lote
- [ ] Lazy loading de componentes
- [ ] Otimização de re-renders
- [ ] Lighthouse audit (>90 em todas categorias)

#### 11.3 Testes com Usuários
- [ ] Recrutar 5-10 usuários da UFMA
- [ ] Sessões de teste de usabilidade
- [ ] Coletar feedback estruturado
- [ ] Identificar bugs e problemas de UX

#### 11.4 Ajustes Finais
- [ ] Implementar melhorias do feedback
- [ ] Correções de bugs
- [ ] Atualização da documentação
- [ ] Release notes

---

## Checklist de Qualidade (Aplicar em Cada Feature)

### Antes de Marcar como Completo:
- [ ] Código funciona conforme especificado
- [ ] Tratamento de erros implementado
- [ ] UI responsiva (mobile + desktop)
- [ ] Acessibilidade básica (navegação por teclado, ARIA)
- [ ] Performance aceitável (sem travamentos)
- [ ] Testado manualmente com casos de sucesso e falha
- [ ] Commit realizado com mensagem descritiva

---

## Métricas de Acompanhamento

### Fase 1 (MVP)
- [ ] Taxa de sucesso de busca por DOI: >80%
- [ ] Taxa de sucesso de busca por título: >60%
- [ ] Tempo médio de busca: <10s
- [ ] 4 fontes integradas funcionais

### Fase 2 (Lote)
- [ ] Processa 50 referências em <6min
- [ ] Taxa de sucesso em lote: >65%
- [ ] Parser BibTeX: >95% de entradas válidas processadas
- [ ] 4 formatos de exportação funcionais

### Fase 3 (Avançado)
- [ ] Parser ABNT/APA: >85% de detecção correta
- [ ] 6 fontes integradas funcionais
- [ ] Lighthouse score: >90
- [ ] Feedback de usuários: >4.0/5.0

---

## Notas de Desenvolvimento

### Convenções de Commit
```
feat: adiciona nova funcionalidade
fix: corrige bug
refactor: refatoração de código
style: mudanças de estilo/formatação
docs: atualização de documentação
test: adiciona ou modifica testes
perf: melhoria de performance
chore: tarefas de manutenção
```

### Boas Práticas
- Sempre testar manualmente após implementação
- Commits frequentes com mensagens descritivas
- Código limpo e comentado onde necessário
- Componentes pequenos e reutilizáveis
- Tratamento de erros em todas as chamadas de API
- Loading states para operações assíncronas

---

**Última atualização:** 10 de Dezembro de 2025