---
layout: default
title: EPUB Reader & Library
---

<!-- Include required libraries for ePub.js -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/epubjs/dist/epub.min.js"></script>

<style>
  .epub-library-container {
    display: flex;
    flex-direction: column;
    gap: 20px;
    margin-top: 20px;
  }

  .file-list {
    list-style-type: none;
    padding: 0;
    margin: 0;
  }

  .file-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 12px 16px;
    border: 1px solid #e1e4e8;
    border-radius: 6px;
    margin-bottom: 8px;
    background-color: #fff;
    transition: background-color 0.2s ease;
  }

  .file-item:hover {
    background-color: #f6f8fa;
  }

  .file-title {
    font-weight: 600;
    color: #0366d6;
    cursor: pointer;
    text-decoration: none;
    flex-grow: 1;
    margin-right: 15px;
  }

  .file-title:hover {
    text-decoration: underline;
  }

  .actions {
    display: flex;
    gap: 8px;
  }

  .btn {
    padding: 6px 12px;
    font-size: 14px;
    border-radius: 4px;
    border: 1px solid #d1d5da;
    background-color: #fafbfc;
    cursor: pointer;
    text-decoration: none;
    color: #24292e;
    display: inline-flex;
    align-items: center;
  }

  .btn:hover {
    background-color: #f3f4f6;
  }

  .btn-primary {
    background-color: #2ea44f;
    color: #fff;
    border-color: rgba(27,31,35,0.15);
  }

  .btn-primary:hover {
    background-color: #2c974b;
  }

  #viewer-container {
    display: none;
    margin-top: 20px;
    padding: 20px;
    border: 1px solid #d1d5da;
    border-radius: 6px;
    background-color: #fff;
  }

  #viewer-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
  }

  #viewer {
    width: 100%;
    height: 600px;
    border: 1px solid #e1e4e8;
    background: #fafafa;
  }

  .viewer-controls {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-top: 15px;
  }
</style>

<h2>Available Books</h2>

<ul class="file-list">
  {% assign found_epubs = false %}
  {% for file in site.static_files %}
    {% if file.extname == '.epub' %}
      {% assign found_epubs = true %}
      {% assign file_name = file.basename | replace: "_", " " | replace: "-", " " %}
      <li class="file-item">
        <span class="file-title" onclick="loadBook('{{ file.path | relative_url }}', '{{ file.basename }}')">
          📖 {{ file_name }}
        </span>
        <div class="actions">
          <button class="btn btn-primary" onclick="loadBook('{{ file.path | relative_url }}', '{{ file.basename }}')">Preview</button>
          <a class="btn" href="{{ file.path | relative_url }}" download>Download</a>
        </div>
      </li>
    {% endif %}
  {% endfor %}

  {% unless found_epubs %}
    <p><em>No .epub files found. Place your .epub files inside an <code>epubs/</code> directory in your repository.</em></p>
  {% endunless %}
</ul>

<div id="viewer-container">
  <div id="viewer-header">
    <h3 id="current-title" style="margin: 0;">Reading...</h3>
    <button class="btn" onclick="closeViewer()">✕ Close Preview</button>
  </div>
  
  <div id="viewer"></div>

  <div class="viewer-controls">
    <button class="btn" id="prev" onclick="prevPage()">← Previous</button>
    <button class="btn" id="next" onclick="nextPage()">Next →</button>
  </div>
</div>

<script>
  let currentBook = null;
  let rendition = null;

  function loadBook(url, title) {
    const container = document.getElementById("viewer-container");
    const titleElement = document.getElementById("current-title");
    
    // Display section and update header title
    container.style.display = "block";
    titleElement.textContent = "Previewing: " + title;
    
    // Smooth scroll down to viewer
    container.scrollIntoView({ behavior: 'smooth' });

    // Clean up previous book instance if open
    if (rendition) {
      rendition.destroy();
    }

    // Initialize ePub.js
    currentBook = ePub(url);
    rendition = currentBook.renderTo("viewer", {
      width: "100%",
      height: "100%",
      spread: "always"
    });

    rendition.display();
  }

  function nextPage() {
    if (rendition) rendition.next();
  }

  function prevPage() {
    if (rendition) rendition.prev();
  }

  function closeViewer() {
    document.getElementById("viewer-container").style.display = "none";
    if (rendition) {
      rendition.destroy();
      rendition = null;
    }
  }

  // Keyboard navigation
  document.addEventListener("keyup", function(e) {
    if (!rendition) return;
    if (e.key === "ArrowLeft") prevPage();
    if (e.key === "ArrowRight") nextPage();
  });
</script>