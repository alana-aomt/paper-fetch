# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**PaperFetch** is a web application for automating the search and download of academic papers from bibliographic references in various formats. The system automatically identifies the type of reference provided and searches the article in multiple academic sources, prioritizing open access versions.

**Primary Users:** Academic researchers, university professors, graduate students (primarily in Brazil)

**Key Features:**
- Individual search: Parse any reference format (DOI, title, ABNT/APA citation, URL, PMID, arXiv ID) and search across multiple academic databases
- Batch processing: Upload .bib files and process up to 50 references in parallel
- Multi-source search: CrossRef, Unpaywall, SciELO, PubMed, arXiv, Semantic Scholar
- Export results: CSV, JSON, BibTeX (enriched), Markdown

## Technology Stack

**Frontend:**
- React 18+ with Hooks and Context API
- Vite for build and dev server
- Tailwind CSS for styling
- Lucide React for icons

**Key Libraries:**
- `citation-js` - Parse bibliographic citations
- `@retorquere/bibtex-parser` or `bibtex-parse` - Parse .bib files
- `axios` - HTTP client for API calls
- `papaparse` - CSV export
- `file-saver` - Download generated files
- `dompurify` - HTML sanitization

**External APIs (all without authentication in Phase 1):**
- CrossRef REST API
- Unpaywall API (requires email in query params)
- SciELO API
- PubMed E-utilities
- arXiv API (OAI-PMH)
- Semantic Scholar API (rate limited)

## Project Structure

The planned architecture follows this structure (see prd.md:556-591 for full details):

```
src/
├── components/
│   ├── SearchInput.jsx          # Single search field with parser
│   ├── BatchUpload.jsx          # .bib file upload component
│   ├── ResultsList.jsx          # Results list (single mode)
│   ├── ResultsTable.jsx         # Results table (batch mode)
│   ├── ResultCard.jsx           # Individual article card
│   ├── BatchProgressBar.jsx    # Batch progress indicator
│   ├── BatchSummary.jsx        # Processing summary
│   └── ExportMenu.jsx          # Export options menu
├── services/
│   ├── referenceParser.js      # Main parsing logic
│   ├── bibtexParser.js         # BibTeX-specific parser
│   ├── batchProcessor.js       # Batch orchestration (max 5 parallel)
│   ├── exportService.js        # Export generation (CSV, JSON, etc)
│   ├── apiClients/             # API integrations
│   │   ├── crossref.js
│   │   ├── unpaywall.js
│   │   ├── scielo.js
│   │   ├── pubmed.js
│   │   └── arxiv.js
│   └── searchOrchestrator.js   # Coordinates parallel searches
├── utils/
│   ├── citationFormatter.js    # Citation formatting
│   ├── pdfValidator.js         # Validate PDF links
│   ├── fileHandlers.js         # Upload and download
│   └── storage.js              # LocalStorage/IndexedDB wrapper
└── hooks/
    ├── useSearch.js            # Single search hook
    ├── useBatchSearch.js       # Batch search hook
    └── useHistory.js           # History management
```

## Development Commands

Since the project is not yet initialized, here are the expected commands once setup is complete:

**Development:**
```bash
npm run dev          # Start Vite dev server
npm run build        # Build for production
npm run preview      # Preview production build
```

**Testing (to be added):**
```bash
npm test             # Run all tests
npm run test:watch   # Run tests in watch mode
```

**Linting (to be added):**
```bash
npm run lint         # Run ESLint
npm run lint:fix     # Auto-fix linting issues
```

## Architecture Principles

### Search Flow (Individual)
1. User pastes reference → `referenceParser` extracts structured metadata
2. `searchOrchestrator` dispatches parallel searches to all API clients
3. Each `apiClient` returns array of results
4. Results are consolidated, deduplicated, and validated
5. Results sorted by availability (PDF available first) and relevance
6. `ResultsList` renders cards with download/copy actions

### Batch Processing Flow
1. User uploads .bib file → `bibtexParser` extracts all entries
2. Preview table shown with checkboxes for selection
3. User confirms → `batchProcessor` starts processing
4. Max 5 references processed in parallel (rate limiting)
5. Real-time status updates in table
6. Progress bar and summary statistics
7. Results exportable in 4 formats (CSV, JSON, BibTeX, Markdown)

### API Integration Strategy
- **Parallel searches:** All sources queried simultaneously with 10s timeout
- **Retry logic:** Automatic retry with exponential backoff for transient failures
- **Graceful degradation:** Continue with partial results if some APIs fail
- **Rate limiting:** Client-side throttling to respect API limits (critical for batch mode)
- **Caching:** Results cached for 24h for identical references

### Reference Detection Priority
When a reference is detected, the search strategy adapts:
1. **If DOI available:** Direct CrossRef lookup → Unpaywall for PDF → fallback to title search
2. **If title only:** Exact title match in all sources → fuzzy search if <3 results → rank by metadata similarity
3. **If full citation:** Extract title for primary search → filter by author/year → validate match

## Critical Implementation Details

### Batch Processing Constraints
- **Limit:** 50 references per batch (client-side processing limit)
- **Concurrency:** Max 5 simultaneous API requests to avoid rate limiting
- **Progress:** Real-time updates with estimated completion time
- **Cancellation:** Users can pause/cancel and resume later
- **Storage:** Use IndexedDB for batch history (last 10 batches)

### PDF Detection
1. Check for direct PDF link in API response
2. Verify availability via HEAD request
3. Classify: "PDF Disponível" | "Acesso Restrito" | "Incerto"
4. Indicate open access type: Gold, Green, Hybrid

### Export Formats (Batch Mode)
- **CSV:** Tabular data with columns: Title, Authors, Year, DOI, Status, PDF Link, Source
- **JSON:** Complete structured data with all metadata
- **BibTeX enriched:** Original .bib with added DOI, URL, and note fields; non-found entries marked with comments
- **Markdown:** Human-readable report with clickable links, separated by status

### Error Handling
- **API failures:** Show warning but don't block results from other sources
- **Malformed .bib:** Validate line-by-line, report problematic entries individually, continue with valid entries
- **Network errors:** Clear user-facing messages, retry suggestions
- **Partial results:** Always show what was found, indicate what failed

## Performance Requirements

See prd.md:463-473 for full details:
- Format detection: <200ms
- Complete multi-source search: <10s
- Batch processing: ~6s per reference average
- Interface remains responsive during async operations
- Results cached for 24h

## API Integration Notes

### CrossRef API
- No authentication required
- Endpoint: `https://api.crossref.org/works/{doi}`
- Rate limit: ~50 req/s
- Use for: DOI lookups, metadata validation

### Unpaywall API
- Requires email in query: `?email=your@email.com`
- Endpoint: `https://api.unpaywall.org/v2/{doi}`
- Rate limit: 100k/day per email
- Use for: Finding open access PDFs

### SciELO API
- Public REST API
- Endpoint: `https://api.scielo.org/`
- Use for: Latin American academic papers (priority for Brazilian users)

### PubMed E-utilities
- No authentication in Phase 1
- Endpoint: `https://eutils.ncbi.nlm.nih.gov/entrez/eutils/`
- Rate limit: 3 req/s without API key
- Use for: Biomedical literature

### arXiv API
- OAI-PMH protocol
- Endpoint: `http://export.arxiv.org/api/query`
- Rate limit: 1 req/3s
- Use for: Preprints in physics, math, CS

## Legal and Compliance

- **Never host PDFs:** Only provide links to legitimate sources
- **Respect robots.txt:** Use official APIs only, avoid scraping
- **Rate limiting:** Implement client-side throttling to avoid overwhelming APIs
- **Copyright disclaimer:** Clear messaging about PDF rights
- **Privacy:** No server-side storage, all data in localStorage/IndexedDB
- **ToS compliance:** Review and follow terms of service for each API

## Internationalization

- **Phase 1:** Portuguese (Brazil) only
- **Phase 2:** Prepared for English and Spanish
- UTF-8 support throughout
- Citation formats should support international characters and diacritics

## Accessibility Requirements

See prd.md:977-1001 for full WCAG 2.1 Level AA compliance:
- Keyboard navigation (Tab, Enter, Esc, Ctrl+K)
- ARIA labels and semantic landmarks
- 4.5:1 contrast ratio minimum
- Touch targets ≥44x44px
- Screen reader announcements for dynamic content

## Development Phases

### Phase 1 - MVP (4 weeks)
- Single search with DOI and title support
- Integration with CrossRef, Unpaywall, SciELO, arXiv
- Basic results display
- Responsive UI

### Phase 2 - Batch Processing (4 weeks)
- .bib file upload and parsing
- Batch processing with progress tracking
- Results table with filters
- Export in 4 formats

### Phase 3 - Advanced Features (3 weeks)
- Robust ABNT/APA citation parsing
- PubMed integration
- Search history and favorites
- Advanced caching

## Success Metrics

Key targets (see prd.md:863-889):
- **Search success rate (individual):** >70% find PDF or link
- **Search success rate (batch):** >65% per entry
- **Batch processing speed:** <6s per reference
- **BibTeX parser accuracy:** >95% of valid entries processed correctly
- **User satisfaction:** >4.0/5.0

## Known Risks

See prd.md:893-974 for full risk assessment:
- **API instability:** Implement retry logic and graceful degradation
- **Rate limiting:** Client-side throttling and queue management
- **Parsing accuracy:** Multiple strategies, allow manual correction
- **Slow batch processing:** 5 parallel requests max, caching, pause/resume capability
- **Malformed .bib files:** Tolerant parser, per-entry validation, clear error reporting

## Reference Documentation

- PRD: See `prd.md` for complete product requirements
- CrossRef API: https://www.crossref.org/documentation/retrieve-metadata/rest-api/
- Unpaywall API: https://unpaywall.org/products/api
- Citation.js: https://citation.js.org/
- BibTeX parser: https://github.com/retorquere/bibtex-parser
