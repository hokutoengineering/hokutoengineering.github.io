---
layout: home
title: void*
---

# Games

<style>
.game-grid {
  display: grid !important;
  grid-template-columns: repeat(3, 1fr) !important;
  gap: 1.5rem !important;
  margin: 1.5rem 0 !important;
}

@media (max-width: 720px) {
  .game-grid {
    grid-template-columns: repeat(2, 1fr) !important;
  }
}

@media (max-width: 480px) {
  .game-grid {
    grid-template-columns: 1fr !important;
  }
}

.game-card {
  background-color: #23272e !important;
  border: 1px solid #30363d !important;
  border-radius: 8px !important;
  overflow: hidden !important;
  transition: transform 0.15s ease, border-color 0.15s ease !important;
}

.game-card:hover {
  border-color: #58a6ff !important;
  transform: translateY(-2px) !important;
}

.game-card img {
  width: 100% !important;
  height: 140px !important;
  object-fit: cover !important;
  display: block !important;
}

.game-card h3 {
  padding: 0.75rem 1rem !important;
  margin: 0 !important;
  font-size: 0.95rem !important;
  color: #f0f6fc !important;
  text-align: center !important;
}

.game-card a {
  text-decoration: none !important;
  display: block !important;
  height: 100% !important;
  color: inherit !important;
}
</style>

<div class="game-grid">
  <div class="game-card">
    <a href="/Experiments/games/tetris.html">
      <img src="./imgs/tetris.png" alt="Tetris preview">
      <h3>Tetris</h3>
    </a>
  </div>
  <div class="game-card">
    <a href="/Experiments/games/puyo.html">
      <img src="./imgs/puyo.png" alt="Puyo-puyo preview">
      <h3>Puyo-puyo</h3>
    </a>
  </div>
  <div class="game-card">
    <a href="/Experiments/games/tamacchi1.html">
      <img src="./imgs/tamacchi.png" alt="たまっち preview">
      <h3>たまっち</h3>
    </a>
  </div>
</div>

# Educational?

- [漢字を探せ（ウォーリーではない）](https://hokutoengineering.github.io/Experiments/games/randomKanji.html)
- [漢字ビンゴ](https://hokutoengineering.github.io/Experiments/games/kanjiBingo.html)

# More Educational?

- [MLPがどのように学んでいるかの可視化](https://hokutoengineering.github.io/Experiments/games/mlp_training.html)
- [CNNがどのように学んでいるかの可視化](https://hokutoengineering.github.io/Experiments/games/cnn_visualization.html)
