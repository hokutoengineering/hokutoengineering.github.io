---
layout: page
title: ""
permalink: /library/
---

<!-- Include CDN dependencies for Markdown parsing, Code Highlighting, and Math rendering -->
<script src="https://cdn.jsdelivr.net/npm/marked@12.0.2/marked.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/styles/atom-one-dark.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/highlight.min.js"></script>

<!-- Embedded Book Catalog from _data/books.yml -->
<script id="books-data" type="application/json">
{{ site.data.books | jsonify }}
</script>

<div class="library-app">
  <!-- Top Reading Progress Indicator (visible in reader mode) -->
  <div id="read-progress" class="read-progress-bar"></div>

  <!-- VIEW 1: LIBRARY VIEW -->
  <div id="library-view" class="view-section">
    <div class="library-header">
      <div class="library-search-box">
        <input type="text" id="library-search" placeholder="Search by title, author, or keyword..." autocomplete="off">
        <span id="search-count" class="search-count"></span>
      </div>
    </div>

    <div class="books-grid" id="books-grid">
      <!-- Populated dynamically via JS from site.data.books -->
    </div>
  </div>

  <!-- VIEW 2: READER VIEW -->
  <div id="reader-view" class="view-section" style="display: none;">
    <div class="reader-toolbar">
      <button id="btn-back-library" class="reader-nav-btn">
        <span class="icon">&larr;</span> Back
      </button>
      <div class="reader-meta-compact" id="reader-header-meta"></div>
      <div class="reader-controls">
        <button id="btn-toggle-writing-mode" class="ctrl-btn" title="Toggle Vertical / Horizontal Writing Mode">横書き</button>
        <button id="btn-font-decrease" class="ctrl-btn" title="Decrease font size">A-</button>
        <button id="btn-font-increase" class="ctrl-btn" title="Increase font size">A+</button>
        <button id="btn-toggle-fullscreen" class="ctrl-btn" title="Toggle Fullscreen">⛶</button>
        <button id="btn-toggle-toc" class="ctrl-btn" title="Toggle Table of Contents">TOC</button>
      </div>
    </div>

    <div class="reader-layout vertical-mode" id="reader-layout">
      <!-- Sticky Sidebar for Table of Contents (hidden by default) -->
      <aside id="reader-toc-sidebar" class="reader-toc-sidebar" style="display: none;">
        <h4>Table of Contents</h4>
        <nav id="reader-toc-nav"></nav>
      </aside>

      <!-- Main Document Content -->
      <main class="reader-content-wrap vertical-mode" id="reader-content-wrap">
        <div id="reader-loading" class="reader-loading">
          <div class="spinner"></div> Loading document...
        </div>
        <div id="reader-error" class="reader-error" style="display: none;"></div>
        <article id="reader-article" class="reader-article"></article>
      </main>
    </div>
  </div>
</div>

<style>
.page-header,
.page-title {
  display: none !important;
}

/* Library & Reader Container */
.library-app {
  margin: 1.5rem 0 3rem 0;
  position: relative;
}

/* Progress bar */
.read-progress-bar {
  position: fixed;
  top: 0;
  left: 0;
  height: 3px;
  background: #5aa9ff;
  width: 0%;
  z-index: 9999;
  transition: width 0.1s ease;
}

/* Library Header & Search */
.library-header {
  margin-bottom: 2rem;
}
.library-search-box {
  position: relative;
  display: flex;
  align-items: center;
}
.library-search-box input {
  width: 100%;
  padding: 0.75rem 1rem;
  background: #1e1e1e;
  border: 1px solid #3d3d3d;
  border-radius: 8px;
  color: #eee;
  font-size: 1rem;
  outline: none;
  transition: border-color 0.2s;
}
.library-search-box input:focus {
  border-color: #5aa9ff;
}
.search-count {
  position: absolute;
  right: 1rem;
  font-size: 0.85rem;
  color: #888;
}

/* Books Grid */
.books-grid {
  display: flex;
  flex-direction: column;
  gap: 0.85rem;
}

/* Book Card */
.book-card {
  background: #181818;
  border: 1px solid #2e2e2e;
  border-radius: 8px;
  padding: 1rem 1.25rem;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  gap: 1.5rem;
  cursor: pointer;
  transition: transform 0.15s ease, border-color 0.15s ease, box-shadow 0.15s ease;
}
.book-card:hover {
  transform: translateX(4px);
  border-color: #5aa9ff;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.35);
}
.book-card-info {
  flex-grow: 1;
  min-width: 0;
}
.book-card-title {
  font-size: 1.05rem;
  font-weight: 600;
  line-height: 1.4;
  color: #fff;
  margin-bottom: 0.25rem;
}
.book-card-authors {
  font-size: 0.88rem;
  color: #aaa;
  line-height: 1.3;
}
.book-card-actions {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-shrink: 0;
}
.book-read-btn {
  background: #5aa9ff;
  color: #111;
  font-weight: 600;
  font-size: 0.85rem;
  padding: 0.45rem 1rem;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  white-space: nowrap;
  flex-shrink: 0;
  transition: background-color 0.15s;
}
.book-read-btn:hover {
  background: #7ec0ff;
}
.book-external-links {
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-end;
  gap: 0.35rem 0.5rem;
}
.book-external-links a {
  color: #8ac0ff;
  text-decoration: none;
  font-size: 0.75rem;
  line-height: 1.2;
}
.book-external-links a:hover {
  text-decoration: underline;
}

/* Reader Toolbar */
.reader-toolbar {
  position: sticky;
  top: 0;
  z-index: 100;
  background: #141414;
  border-bottom: 1px solid #333;
  padding: 0.75rem 0;
  margin-bottom: 1.5rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}
.reader-nav-btn {
  background: #252525;
  border: 1px solid #444;
  color: #eee;
  padding: 0.45rem 0.9rem;
  border-radius: 5px;
  font-size: 0.9rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.4rem;
  transition: background 0.15s, border-color 0.15s;
}
.reader-nav-btn:hover {
  background: #333;
  border-color: #5aa9ff;
  color: #5aa9ff;
}
.reader-meta-compact {
  font-size: 0.9rem;
  color: #888;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.reader-controls {
  display: flex;
  gap: 0.4rem;
}
.ctrl-btn {
  background: #222;
  border: 1px solid #444;
  color: #ccc;
  padding: 0.35rem 0.65rem;
  border-radius: 4px;
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.15s ease;
}
.ctrl-btn:hover {
  border-color: #5aa9ff;
  color: #fff;
}
.ctrl-btn.active {
  background: #19385c;
  border-color: #5aa9ff;
  color: #5aa9ff;
  font-weight: 600;
}

.reader-view-fullscreen {
  background: #0f0f0f;
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  padding: 0;
  margin: 0;
}

.reader-view-fullscreen .reader-toolbar {
  margin-bottom: 0.5rem;
}

.reader-view-fullscreen .reader-layout {
  flex: 1 1 auto;
  width: 100%;
  height: calc(100vh - 82px);
  min-height: 0;
}

.reader-view-fullscreen .reader-content-wrap {
  height: 100%;
  min-height: 0;
}

/* Reader Layout */
.reader-layout {
  display: flex;
  gap: 1.5rem;
  align-items: flex-start;
}
.reader-layout.vertical-mode {
  align-items: stretch;
}

/* Table of Contents Sidebar */
.reader-toc-sidebar {
  width: 220px;
  flex-shrink: 0;
  position: sticky;
  top: 70px;
  max-height: calc(100vh - 90px);
  overflow-y: auto;
  background: #181818;
  border: 1px solid #282828;
  border-radius: 6px;
  padding: 1rem;
  font-size: 0.85rem;
}
.reader-toc-sidebar h4 {
  margin-top: 0;
  margin-bottom: 0.75rem;
  font-size: 0.9rem;
  text-transform: uppercase;
  color: #777;
  letter-spacing: 0.05em;
}
.reader-toc-sidebar ul {
  list-style: none;
  padding-left: 0;
  margin: 0;
}
.reader-toc-sidebar li {
  margin-bottom: 0.45rem;
}
.reader-toc-sidebar li.toc-level-3 {
  padding-left: 1rem;
}
.reader-toc-sidebar a {
  color: #aaa;
  text-decoration: none;
  display: block;
  line-height: 1.35;
  transition: color 0.15s;
}
.reader-toc-sidebar a:hover,
.reader-toc-sidebar a.active {
  color: #5aa9ff;
}

/* Reader Content Wrap */
.reader-content-wrap {
  flex-grow: 1;
  min-width: 0;
  background: #161616;
  border: 1px solid #282828;
  border-radius: 8px;
  padding: 2.5rem 3rem;
  font-size: 1.05rem;
  line-height: 1.75;
  color: #d8d8d8;
  box-sizing: border-box;
}

/* Ruby Typography Support (Furigana / ルビ) */
ruby {
  ruby-position: over;
  -webkit-ruby-position: before;
  font-family: inherit;
}
rt {
  font-size: 0.52em;
  line-height: 1;
  color: #8ac0ff;
  font-weight: normal;
  user-select: none;
  -webkit-user-select: none;
  letter-spacing: 0;
}
rp {
  display: none;
}

/* =========================================================
   VERTICAL WRITING MODE (縦書き・左送りスクロール)
   ========================================================= */
.reader-content-wrap.vertical-mode {
  height: 75vh;
  min-height: 520px;
  max-height: 850px;
  overflow-x: auto;
  overflow-y: hidden;
  padding: 2.5rem 3rem;
  writing-mode: vertical-rl;
  -webkit-writing-mode: vertical-rl;
  text-orientation: mixed;
  line-height: 2.2;
  letter-spacing: 0.06em;
  font-family: "Hiragino Mincho ProN", "Yu Mincho", "YuMincho", "Noto Serif CJK JP", "Noto Serif JP", "BIZ UDPMincho", "MS Mincho", "Georgia", serif;
  scrollbar-width: thin;
  scrollbar-color: #5aa9ff #1c1c1c;
}

.reader-view-fullscreen .reader-content-wrap.vertical-mode {
  height: 100%;
  min-height: 0;
  max-height: none;
}

.reader-content-wrap.vertical-mode::-webkit-scrollbar {
  height: 8px;
}
.reader-content-wrap.vertical-mode::-webkit-scrollbar-track {
  background: #1c1c1c;
  border-radius: 4px;
}
.reader-content-wrap.vertical-mode::-webkit-scrollbar-thumb {
  background: #444;
  border-radius: 4px;
}
.reader-content-wrap.vertical-mode::-webkit-scrollbar-thumb:hover {
  background: #5aa9ff;
}

/* Vertical Typography adjustments */
.reader-content-wrap.vertical-mode .reader-article {
  height: 100%;
}
.reader-content-wrap.vertical-mode .reader-article p {
  text-indent: 1em;
  margin: 0 1rem;
}
.reader-content-wrap.vertical-mode .reader-article h1 {
  font-size: 1.85rem;
  color: #fff;
  border-bottom: none;
  border-right: 3px solid #5aa9ff;
  padding-bottom: 0;
  padding-right: 0.6rem;
  margin: 0 2rem 0 1rem;
  letter-spacing: 0.08em;
}
.reader-content-wrap.vertical-mode .reader-article h2 {
  font-size: 1.4rem;
  color: #e5e5e5;
  border-bottom: none;
  border-right: 2px solid #555;
  padding-bottom: 0;
  padding-right: 0.5rem;
  margin: 0 1.6rem 0 0.9rem;
}
.reader-content-wrap.vertical-mode .reader-article h3 {
  font-size: 1.15rem;
  color: #ccc;
  margin: 0 1.3rem 0 0.8rem;
}
.reader-content-wrap.vertical-mode .reader-article hr {
  height: 100%;
  width: 1px;
  border: none;
  background: #333;
  margin: 0 1.5rem;
}
.reader-content-wrap.vertical-mode .reader-article pre {
  writing-mode: horizontal-tb;
  -webkit-writing-mode: horizontal-tb;
  text-orientation: initial;
  text-indent: 0;
  margin: 0 1.5rem;
  max-height: 85%;
  max-width: 550px;
}
.reader-content-wrap.vertical-mode .reader-article table {
  writing-mode: horizontal-tb;
  -webkit-writing-mode: horizontal-tb;
  text-indent: 0;
  margin: 0 1.5rem;
  max-height: 85%;
}
.reader-content-wrap.vertical-mode .reader-article blockquote {
  writing-mode: vertical-rl;
  -webkit-writing-mode: vertical-rl;
  border-left: none;
  border-right: 3px solid #5aa9ff;
  margin: 0 1.5rem;
  padding: 1rem 1.25rem;
}
.reader-content-wrap.vertical-mode .reader-article ul,
.reader-content-wrap.vertical-mode .reader-article ol {
  margin: 0 1.2rem;
  padding-top: 0.5rem;
}

/* Ruby in vertical mode */
.reader-content-wrap.vertical-mode ruby rt {
  font-size: 0.5em;
  letter-spacing: 0;
  padding-left: 2px;
}

/* Horizontal Standard Typography */
.reader-article h1 {
  font-size: 2rem;
  color: #fff;
  border-bottom: 1px solid #333;
  padding-bottom: 0.5rem;
  margin-top: 0;
}
.reader-article h2 {
  font-size: 1.45rem;
  color: #e5e5e5;
  border-bottom: 1px solid #282828;
  padding-bottom: 0.35rem;
  margin-top: 2rem;
}
.reader-article h3 {
  font-size: 1.2rem;
  color: #ccc;
  margin-top: 1.5rem;
}
.reader-article pre {
  background: #1e1e1e !important;
  border: 1px solid #333;
  border-radius: 6px;
  padding: 1rem;
  overflow-x: auto;
}
.reader-article blockquote {
  border-left: 4px solid #5aa9ff;
  background: #1c232d;
  margin: 1.5rem 0;
  padding: 0.75rem 1.25rem;
  color: #b0c4de;
}
.reader-article table {
  width: 100%;
  border-collapse: collapse;
  margin: 1.5rem 0;
}
.reader-article table th,
.reader-article table td {
  border: 1px solid #333;
  padding: 0.6rem 0.9rem;
}
.reader-article table th {
  background: #202020;
}

/* Loading & Error States */
.reader-loading {
  padding: 3rem;
  text-align: center;
  color: #888;
}
.reader-error {
  padding: 1.5rem;
  background: #381a1a;
  border: 1px solid #772b2b;
  border-radius: 6px;
  color: #ff9999;
}

/* Responsive adjustments */
@media (max-width: 860px) {
  .reader-layout {
    flex-direction: column;
  }
  .reader-toc-sidebar {
    width: 100%;
    position: static;
    max-height: none;
    margin-bottom: 1.5rem;
  }
  .reader-content-wrap {
    padding: 1.5rem;
  }
  .reader-content-wrap.vertical-mode {
    height: 70vh;
    padding: 1.5rem;
  }
}
</style>

<script>
(function() {
  // State
  let booksData = [];
  let currentBook = null;
  let currentFontSize = 1.05; // rem
  let isVertical = true; // Vertical layout by default
  let isTocVisible = false; // TOC hidden by default

  // Elements
  const libraryView = document.getElementById('library-view');
  const readerView = document.getElementById('reader-view');
  const readerLayout = document.getElementById('reader-layout');
  const contentWrap = document.getElementById('reader-content-wrap');
  const booksGrid = document.getElementById('books-grid');
  const searchInput = document.getElementById('library-search');
  const searchCount = document.getElementById('search-count');
  const btnBack = document.getElementById('btn-back-library');
  const articleEl = document.getElementById('reader-article');
  const loadingEl = document.getElementById('reader-loading');
  const errorEl = document.getElementById('reader-error');
  const headerMeta = document.getElementById('reader-header-meta');
  const tocNav = document.getElementById('reader-toc-nav');
  const tocSidebar = document.getElementById('reader-toc-sidebar');
  const btnToggleToc = document.getElementById('btn-toggle-toc');
  const btnToggleWritingMode = document.getElementById('btn-toggle-writing-mode');
  const btnToggleFullscreen = document.getElementById('btn-toggle-fullscreen');
  const btnFontDec = document.getElementById('btn-font-decrease');
  const btnFontInc = document.getElementById('btn-font-increase');
  const progressBar = document.getElementById('read-progress');

  // Initialize Marked with Highlight.js
  if (window.marked) {
    marked.setOptions({
      highlight: function(code, lang) {
        if (lang && hljs.getLanguage(lang)) {
          try {
            return hljs.highlight(code, { language: lang }).value;
          } catch (e) {}
        }
        return hljs.highlightAuto(code).value;
      },
      breaks: false,
      gfm: true
    });
  }

  // Load books JSON
  try {
    const raw = document.getElementById('books-data').textContent;
    booksData = JSON.parse(raw) || [];
  } catch(e) {
    console.error("Failed to parse books data:", e);
  }

  // Set Writing Mode
  function setWritingMode(vertical) {
    isVertical = vertical;
    if (isVertical) {
      contentWrap.classList.add('vertical-mode');
      readerLayout.classList.add('vertical-mode');
      btnToggleWritingMode.classList.add('active');
      btnToggleWritingMode.textContent = '横書き';
      // Reset horizontal scroll to right edge (start of vertical Japanese text)
      setTimeout(() => {
        contentWrap.scrollLeft = 0;
        updateVerticalProgress();
      }, 50);
    } else {
      contentWrap.classList.remove('vertical-mode');
      readerLayout.classList.remove('vertical-mode');
      btnToggleWritingMode.classList.remove('active');
      btnToggleWritingMode.textContent = '縦書き';
      progressBar.style.width = '0%';
    }
  }

  // Toggle writing mode
  btnToggleWritingMode.onclick = () => {
    setWritingMode(!isVertical);
  };

  // Convert mouse wheel in vertical mode to horizontal scrolling (leftward scroll)
  contentWrap.addEventListener('wheel', (e) => {
    if (!isVertical) return;
    if (Math.abs(e.deltaY) > Math.abs(e.deltaX)) {
      e.preventDefault();
      // In vertical-rl, scrolling into content moves scrollLeft in the negative (or opposite) direction
      contentWrap.scrollLeft -= e.deltaY;
      updateVerticalProgress();
    }
  }, { passive: false });

  contentWrap.addEventListener('scroll', () => {
    if (isVertical) {
      updateVerticalProgress();
    }
  });

  function updateVerticalProgress() {
    const maxScroll = contentWrap.scrollWidth - contentWrap.clientWidth;
    if (maxScroll <= 0) {
      progressBar.style.width = '0%';
      return;
    }
    const current = Math.abs(contentWrap.scrollLeft);
    const progress = Math.min(100, Math.max(0, (current / maxScroll) * 100));
    progressBar.style.width = `${progress}%`;
  }

  // Render library cards
  function renderLibrary(books) {
    booksGrid.innerHTML = '';
    searchCount.textContent = `${books.length} item${books.length !== 1 ? 's' : ''}`;

    if (books.length === 0) {
      booksGrid.innerHTML = '<div style="color: #888; grid-column: 1/-1; padding: 2rem 0;">No books matched your search.</div>';
      return;
    }

    books.forEach(b => {
      const card = document.createElement('div');
      card.className = 'book-card';
      card.onclick = () => openReader(b.id, true);

      const builtInLinks = [
        { label: 'PDF', url: b.pdf },
        { label: 'EPUB', url: b.epub },
        { label: 'Text', url: b.text }
      ].filter(link => link.url && link.url.trim());

      const customLinks = Array.isArray(b.links)
        ? b.links
            .filter(link => link && link.url && link.url.trim())
            .map(link => ({ label: link.label || 'Link', url: link.url }))
        : [];

      const allLinks = [...builtInLinks, ...customLinks];

      const linksHtml = allLinks.length > 0 ? `
        <div class="book-external-links">
          ${allLinks.map(link => `<a href="${escapeHtml(link.url)}" target="_blank" rel="noopener noreferrer">${escapeHtml(link.label)}</a>`).join('')}
        </div>
      ` : '';

      card.innerHTML = `
        <div class="book-card-info">
          <div class="book-card-title">${escapeHtml(b.title)}</div>
          <div class="book-card-authors">${escapeHtml(b.authors || '')}</div>
        </div>
        <div class="book-card-actions">
          ${linksHtml}
          <button class="book-read-btn">Read &rarr;</button>
        </div>
      `;
      booksGrid.appendChild(card);
    });
  }

  // Filter books on search
  searchInput.addEventListener('input', (e) => {
    const query = e.target.value.toLowerCase().trim();
    if (!query) {
      renderLibrary(booksData);
      return;
    }
    const filtered = booksData.filter(b => {
      return (b.title && b.title.toLowerCase().includes(query)) ||
             (b.authors && b.authors.toLowerCase().includes(query)) ||
             (b.venue && b.venue.toLowerCase().includes(query)) ||
             (b.description && b.description.toLowerCase().includes(query));
    });
    renderLibrary(filtered);
  });

  // Open Reader
  async function openReader(bookId, updateUrl = true) {
    const book = booksData.find(b => b.id === bookId);
    if (!book) {
      showLibrary();
      return;
    }

    currentBook = book;
    if (updateUrl) {
      const newUrl = new URL(window.location);
      newUrl.searchParams.set('book', book.id);
      window.history.pushState({ bookId: book.id }, '', newUrl);
    }

    // Switch views
    libraryView.style.display = 'none';
    readerView.style.display = 'block';
    window.scrollTo({ top: 0, behavior: 'smooth' });

    headerMeta.textContent = `${book.title} — ${book.authors || ''}`;

    loadingEl.style.display = 'block';
    errorEl.style.display = 'none';
    articleEl.innerHTML = '';
    tocNav.innerHTML = '';

    // Always default to vertical layout unless explicitly specified otherwise
    const useVertical = book.direction ? book.direction === 'vertical' : true;
    setWritingMode(useVertical);

    // Keep TOC off by default
    isTocVisible = false;
    tocSidebar.style.display = 'none';
    btnToggleToc.classList.remove('active');

    try {
      const filePath = book.file || `/books/${book.id}.md`;
      const res = await fetch(filePath);
      if (!res.ok) {
        throw new Error(`Failed to load document (${res.status} ${res.statusText})`);
      }
      let markdownText = await res.text();

      // Strip YAML front matter if present in the markdown
      if (markdownText.startsWith('---')) {
        const secondDelim = markdownText.indexOf('---', 3);
        if (secondDelim !== -1) {
          markdownText = markdownText.substring(secondDelim + 3).trim();
        }
      }

      // Parse Markdown
      const htmlContent = marked.parse(markdownText);
      articleEl.innerHTML = htmlContent;
      loadingEl.style.display = 'none';

      // Build Table of Contents
      buildToc();

      // Trigger MathJax re-render if available
      if (window.MathJax && window.MathJax.Hub) {
        MathJax.Hub.Queue(["Typeset", MathJax.Hub, articleEl]);
      }
    } catch(err) {
      loadingEl.style.display = 'none';
      errorEl.style.display = 'block';
      errorEl.textContent = `Could not load markdown content: ${err.message}. Please check that ${book.file} exists.`;
    }
  }

  // Show Library
  function showLibrary(updateUrl = true) {
    if (updateUrl) {
      const newUrl = new URL(window.location);
      newUrl.searchParams.delete('book');
      window.history.pushState({}, '', newUrl);
    }
    readerView.style.display = 'none';
    libraryView.style.display = 'block';
    progressBar.style.width = '0%';
    currentBook = null;
  }

  // Build Table of Contents from rendered headings
  function buildToc() {
    const headings = articleEl.querySelectorAll('h2, h3');
    tocNav.innerHTML = '';
    if (headings.length === 0) {
      tocSidebar.style.display = 'none';
      btnToggleToc.style.display = 'none';
      return;
    }
    btnToggleToc.style.display = 'inline-block';
    tocSidebar.style.display = isTocVisible ? 'block' : 'none';

    const ul = document.createElement('ul');
    headings.forEach((h, idx) => {
      const id = h.id || `heading-${idx}`;
      h.id = id;

      const li = document.createElement('li');
      li.className = h.tagName.toLowerCase() === 'h3' ? 'toc-level-3' : 'toc-level-2';

      const a = document.createElement('a');
      a.href = `#${id}`;
      a.textContent = h.textContent.replace(/^#+\s*/, '');
      a.onclick = (e) => {
        e.preventDefault();
        if (isVertical) {
          h.scrollIntoView({ behavior: 'smooth', inline: 'start', block: 'nearest' });
        } else {
          h.scrollIntoView({ behavior: 'smooth', block: 'start' });
        }
      };

      li.appendChild(a);
      ul.appendChild(li);
    });
    tocNav.appendChild(ul);
  }

  // Reading progress and active TOC highlight for horizontal mode
  window.addEventListener('scroll', () => {
    if (readerView.style.display === 'none' || isVertical) return;

    const scrollTop = window.scrollY;
    const docHeight = document.documentElement.scrollHeight - window.innerHeight;
    const progress = docHeight > 0 ? (scrollTop / docHeight) * 100 : 0;
    progressBar.style.width = `${Math.min(100, Math.max(0, progress))}%`;

    // Highlight active heading in TOC
    const headings = Array.from(articleEl.querySelectorAll('h2, h3'));
    let activeId = null;
    for (let h of headings) {
      const rect = h.getBoundingClientRect();
      if (rect.top <= 120) {
        activeId = h.id;
      } else {
        break;
      }
    }
    if (activeId) {
      const links = tocNav.querySelectorAll('a');
      links.forEach(l => {
        if (l.getAttribute('href') === `#${activeId}`) {
          l.classList.add('active');
        } else {
          l.classList.remove('active');
        }
      });
    }
  });

  // Controls
  btnBack.onclick = () => showLibrary(true);

  btnToggleToc.onclick = () => {
    isTocVisible = !isTocVisible;
    tocSidebar.style.display = isTocVisible ? 'block' : 'none';
    if (isTocVisible) {
      btnToggleToc.classList.add('active');
    } else {
      btnToggleToc.classList.remove('active');
    }
  };

  function updateFullscreenButton() {
    const isFullscreen = !!document.fullscreenElement;
    btnToggleFullscreen.classList.toggle('active', isFullscreen);
    btnToggleFullscreen.textContent = isFullscreen ? 'Exit' : '⛶';
    btnToggleFullscreen.title = isFullscreen ? 'Exit Fullscreen' : 'Toggle Fullscreen';
    readerView.classList.toggle('reader-view-fullscreen', isFullscreen);

    if (isFullscreen) {
      requestAnimationFrame(() => {
        if (isVertical) {
          contentWrap.scrollLeft = 0;
          updateVerticalProgress();
        }
      });
    }
  }

  async function toggleFullscreen() {
    try {
      if (!document.fullscreenElement) {
        if (readerView.requestFullscreen) {
          await readerView.requestFullscreen();
        }
      } else if (document.exitFullscreen) {
        await document.exitFullscreen();
      }
    } catch (err) {
      console.error('Fullscreen toggle failed:', err);
    }
  }

  btnToggleFullscreen.onclick = toggleFullscreen;
  document.addEventListener('fullscreenchange', updateFullscreenButton);

  btnFontInc.onclick = () => {
    currentFontSize = Math.min(1.4, currentFontSize + 0.05);
    articleEl.style.fontSize = `${currentFontSize}rem`;
  };

  btnFontDec.onclick = () => {
    currentFontSize = Math.max(0.85, currentFontSize - 0.05);
    articleEl.style.fontSize = `${currentFontSize}rem`;
  };

  // Popstate for browser Back/Forward
  window.addEventListener('popstate', (e) => {
    const params = new URLSearchParams(window.location.search);
    const bookParam = params.get('book');
    if (bookParam) {
      openReader(bookParam, false);
    } else {
      showLibrary(false);
    }
  });

  function escapeHtml(str) {
    if (!str) return '';
    return str.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;');
  }

  // Initial load: check URL params
  renderLibrary(booksData);
  const initialParams = new URLSearchParams(window.location.search);
  const initialBook = initialParams.get('book');
  if (initialBook) {
    openReader(initialBook, false);
  }
})();
</script>
