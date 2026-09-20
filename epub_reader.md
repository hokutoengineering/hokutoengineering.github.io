---
layout: default
title: EPUB Reader
---

<!-- 1. External Libraries -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/epubjs/dist/epub.min.js"></script>

<!-- 2. Minimal CSS Styles -->
<style>
  .reader-container {
    display: flex;
    flex-direction: column;
    gap: 12px;
    max-width: 800px;
    margin: 0 auto;
    font-family: system-ui, -apple-system, sans-serif;
  }

  .file-select {
    width: 100%;
    padding: 10px;
    font-size: 1rem;
    border: 1px solid #ccc;
    border-radius: 6px;
    background-color: #fff;
  }

  #viewer {
    width: 100%;
    height: 600px;
    border: 1px solid #ddd;
    border-radius: 6px;
    background: #fafafa;
    box-shadow: 0 2px 8px rgba(0,0,0,0.05);
  }

  .controls {
    display: flex;
    justify-content: space-between;
    gap: 10px;
  }

  .btn {
    flex: 1;
    padding: 10px;
    font-size: 0.95rem;
    cursor: pointer;
    border: 1px solid #ccc;
    border-radius: 4px;
    background: #fff;
  }

  .btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
</style>

<!-- 3. HTML Layout -->
<div class="reader-container">
  
  <!-- Dynamic File Selector populated via Jekyll Liquid -->
  <select id="bookSelect" class="file-select">
    <option value="" disabled selected>-- Select an EPUB file --</option>
    {% for file in site.static_files %}
      {% if file.path contains '/epubs/' and file.extname == '.epub' %}
        <option value="{{ file.path | relative_url }}">{{ file.name }}</option>
      {% endif %}
    {% endfor %}
  </select>

  <!-- EPUB Render Area -->
  <div id="viewer"></div>

  <!-- Navigation Buttons -->
  <div class="controls">
    <button id="prev" class="btn" disabled>← Previous</button>
    <button id="next" class="btn" disabled>Next →</button>
  </div>
</div>

<!-- 4. Logic Script -->
<script>
  let book = null;
  let rendition = null;

  const bookSelect = document.getElementById("bookSelect");
  const prevBtn = document.getElementById("prev");
  const nextBtn = document.getElementById("next");

  // Load selected book
  bookSelect.addEventListener("change", (e) => {
    const fileUrl = e.target.value;
    if (!fileUrl) return;

    // Clean up previous instance if loaded
    if (book) {
      book.destroy();
    }

    // Initialize new EPUB reader
    book = ePub(fileUrl);
    rendition = book.renderTo("viewer", {
      width: "100%",
      height: "100%",
      spread: "always"
    });

    rendition.display();

    // Enable navigation controls
    prevBtn.disabled = false;
    nextBtn.disabled = false;
  });

  // Navigation handlers
  prevBtn.addEventListener("click", () => rendition && rendition.prev());
  nextBtn.addEventListener("click", () => rendition && rendition.next());

  // Keyboard navigation support
  document.addEventListener("keyup", (e) => {
    if (!rendition) return;
    if (e.key === "ArrowLeft") rendition.prev();
    if (e.key === "ArrowRight") rendition.next();
  });
</script>