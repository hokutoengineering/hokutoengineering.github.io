---
layout: default
title: EPUB Reader
---

<!-- Include required libraries for ePub.js -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/epubjs/dist/epub.min.js"></script>

<style>
  .epub-page,
  .epub-page * {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif;
  }

  .epub-page {
    max-width: 1080px;
    margin: 1.5rem auto 3rem;
    color: #24292f;
  }

  .epub-header {
    margin-bottom: 1.5rem;
  }

  .epub-header h2 {
    margin: 0 0 0.5rem;
    font-size: clamp(1.8rem, 4vw, 2.5rem);
    color: #1f2328;
    letter-spacing: -0.02em;
  }

  .epub-intro {
    margin: 0;
    color: #57606a;
    line-height: 1.7;
    max-width: 60ch;
  }

  .epub-toolbar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    margin: 1.5rem 0;
    padding: 0.8rem 1rem;
    background: #f6f8fa;
    border: 1px solid #d0d7de;
    border-radius: 10px;
    color: #24292f;
  }

  .epub-status {
    font-weight: 700;
    color: #1f2328;
  }

  .epub-toolbar small {
    color: #57606a;
    font-size: 0.82rem;
  }

  .book-list {
    list-style: none;
    padding: 0;
    margin: 0;
    display: flex;
    flex-direction: column;
    gap: 0.85rem;
  }

  .book-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    padding: 1rem 1.1rem;
    border: 1px solid #d0d7de;
    border-radius: 10px;
    background: #ffffff;
    transition: border-color 0.2s ease, box-shadow 0.2s ease, transform 0.2s ease;
  }

  .book-item:hover {
    border-color: #5aa9ff;
    box-shadow: 0 6px 18px rgba(31, 35, 40, 0.08);
    transform: translateY(-1px);
  }

  .book-main {
    display: flex;
    align-items: center;
    gap: 0.85rem;
    flex: 1;
    min-width: 0;
  }

  .book-icon {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 2.25rem;
    height: 2.25rem;
    border-radius: 8px;
    background: #eef6ff;
    font-size: 1.2rem;
  }

  .book-title {
    font-size: 1.02rem;
    font-weight: 600;
    color: #0f172a;
    line-height: 1.4;
    word-break: break-word;
  }

  .book-actions {
    display: flex;
    align-items: center;
    gap: 0.6rem;
    flex-shrink: 0;
  }

  .btn {
    appearance: none;
    border: 1px solid #d0d7de;
    border-radius: 6px;
    background: #f6f8fa;
    color: #24292f;
    padding: 0.6rem 0.9rem;
    font-size: 0.9rem;
    font-weight: 600;
    line-height: 1.2;
    cursor: pointer;
    text-decoration: none;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    transition: background-color 0.2s ease, border-color 0.2s ease, color 0.2s ease;
  }

  .btn:hover {
    background: #eef2f7;
    border-color: #b6c2cf;
    color: #111827;
    text-decoration: none;
  }

  .btn-primary {
    background: #2da44e;
    border-color: rgba(27, 31, 36, 0.15);
    color: #ffffff;
  }

  .btn-primary:hover {
    background: #2c974b;
    border-color: rgba(27, 31, 36, 0.2);
    color: #ffffff;
  }

  .book-empty {
    padding: 1.25rem;
    border: 1px dashed #d0d7de;
    border-radius: 10px;
    background: #fafbfc;
    color: #57606a;
    line-height: 1.6;
  }

  #viewer-container {
    display: none;
    margin-top: 2rem;
    padding: 1.25rem;
    border: 1px solid #d0d7de;
    border-radius: 12px;
    background: #ffffff;
    box-shadow: 0 8px 26px rgba(31, 35, 40, 0.06);
  }

  #viewer-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;
    margin-bottom: 1rem;
  }

  .viewer-kicker {
    display: block;
    margin-bottom: 0.2rem;
    font-size: 0.72rem;
    font-weight: 700;
    letter-spacing: 0.08em;
    color: #57606a;
    text-transform: uppercase;
  }

  #current-title {
    margin: 0;
    font-size: clamp(1.2rem, 2vw, 1.5rem);
    color: #1f2328;
    word-break: break-word;
  }

  #viewer {
    width: 100%;
    height: 680px;
    border: 1px solid #d8dee4;
    border-radius: 8px;
    background: #f8fafc;
    overflow: auto;
    overscroll-behavior: contain;
  }

  #viewer iframe {
    width: 100%;
    height: 100%;
    border: 0;
    background: white;
    overflow: auto;
  }

  .viewer-status {
    margin-top: 0.8rem;
    font-size: 0.9rem;
    color: #57606a;
  }

  .viewer-status.error {
    color: #b42318;
  }

  .viewer-controls {
    display: flex;
    justify-content: center;
    gap: 0.75rem;
    margin-top: 1rem;
  }

  @media (max-width: 720px) {
    .book-item {
      flex-direction: column;
      align-items: stretch;
    }

    .book-actions {
      justify-content: flex-end;
      width: 100%;
    }

    .epub-toolbar {
      flex-direction: column;
      align-items: flex-start;
    }

    #viewer {
      height: 520px;
    }
  }
</style>

<div class="epub-page">
  <header class="epub-header">
    <h2>EPUB Library</h2>
    <p class="epub-intro">
      Browse the available books in this repository, preview them in the browser, or download the original EPUB file.
    </p>
  </header>

  <div class="epub-toolbar">
    <div class="epub-status" id="book-count-label">0 books available</div>
    <small>Use the preview button to read the file inline.</small>
  </div>

  <ul class="book-list">
    {% assign found_epubs = false %}
    {% for file in site.static_files %}
      {% if file.extname == '.epub' %}
        {% assign found_epubs = true %}
        {% assign file_name = file.basename | replace: "_", " " | replace: "-", " " %}
        <li class="book-item">
          <div class="book-main">
            <span class="book-icon" aria-hidden="true">📖</span>
            <span class="book-title">{{ file_name }}</span>
          </div>
          <div class="book-actions">
            <button
              class="btn btn-primary"
              type="button"
              data-book-url="{{ file.path | relative_url }}"
              data-book-title="{{ file.basename }}"
              aria-label="Preview {{ file_name }}"
            >
              Preview
            </button>
            <a class="btn" href="{{ file.path | relative_url }}" download aria-label="Download {{ file_name }}">Download</a>
          </div>
        </li>
      {% endif %}
    {% endfor %}

    {% unless found_epubs %}
      <li class="book-empty">
        No EPUB files were found yet. Add your files under the <strong>epubs/</strong> folder in this repository and refresh the page.
      </li>
    {% endunless %}
  </ul>

  <div id="viewer-container" role="dialog" aria-live="polite" aria-label="EPUB preview">
    <div id="viewer-header">
      <div>
        <span class="viewer-kicker">Reader</span>
        <h3 id="current-title">Select a book</h3>
      </div>
      <button class="btn" type="button" onclick="closeViewer()" aria-label="Close preview">✕ Close</button>
    </div>

    <div id="viewer" aria-label="EPUB reading area"></div>

    <div class="viewer-controls">
      <button class="btn" type="button" id="prev" onclick="prevPage()" aria-label="Previous page">← Previous</button>
      <button class="btn" type="button" id="next" onclick="nextPage()" aria-label="Next page">Next →</button>
    </div>

    <div id="viewer-status" class="viewer-status">Ready</div>
  </div>
</div>

<script>
  let currentBook = null;
  let rendition = null;

  function updateBookCount() {
    const bookCountLabel = document.getElementById("book-count-label");
    const count = document.querySelectorAll(".book-item").length;
    if (!bookCountLabel) return;
    bookCountLabel.textContent = count === 1 ? "1 book available" : count + " books available";
  }

  function setViewerStatus(message, isError) {
    const status = document.getElementById("viewer-status");
    if (!status) return;
    status.textContent = message;
    status.classList.toggle("error", !!isError);
  }

  function destroyRendition() {
    if (rendition) {
      try {
        rendition.destroy();
      } catch (error) {
        console.warn("Could not destroy rendering instance:", error);
      }
      rendition = null;
    }
  }

  function closeViewer() {
    const viewer = document.getElementById("viewer-container");
    if (viewer) {
      viewer.style.display = "none";
    }
    destroyRendition();
    setViewerStatus("Ready", false);
  }

  function loadBook(url, title) {
    const container = document.getElementById("viewer-container");
    const titleElement = document.getElementById("current-title");

    if (!container || !titleElement) return;

    container.style.display = "block";
    titleElement.textContent = title;
    setViewerStatus("Loading preview…", false);
    container.scrollIntoView({ behavior: "smooth", block: "start" });

    destroyRendition();

    try {
      currentBook = ePub(url);
      rendition = currentBook.renderTo("viewer", {
        width: "100%",
        height: "100%",
        spread: "always"
      });

      rendition.display().then(() => {
        setViewerStatus("Preview ready", false);
      }).catch((error) => {
        console.error("Failed to display EPUB:", error);
        setViewerStatus("This book could not be opened in the preview viewer.", true);
      });
    } catch (error) {
      console.error("Error creating EPUB instance:", error);
      setViewerStatus("The EPUB file could not be loaded.", true);
    }
  }

  function nextPage() {
    if (rendition) {
      rendition.next();
    }
  }

  function prevPage() {
    if (rendition) {
      rendition.prev();
    }
  }

  document.addEventListener("click", function (event) {
    const trigger = event.target.closest("[data-book-url]");
    if (!trigger) return;

    const url = trigger.dataset.bookUrl;
    const title = trigger.dataset.bookTitle || "Untitled";
    loadBook(url, title);
  });

  document.addEventListener("keyup", function (event) {
    if (!rendition) return;

    if (event.key === "ArrowLeft") {
      prevPage();
    }

    if (event.key === "ArrowRight") {
      nextPage();
    }

    if (event.key === "Escape") {
      closeViewer();
    }
  });

  updateBookCount();
</script>
