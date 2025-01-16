<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tycoon Game</title>
  <style>
    /* CSS styles */
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      text-align: center;
    }
    #game {
      display: grid;
      grid-template-columns: repeat(6, 50px);
      grid-template-rows: repeat(10, 50px);
      gap: 2px;
      margin: 20px auto;
      width: 520px;
    }
    .tile {
      border: 1px solid #ccc;
      background-color: #f9f9f9;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      position: relative;
    }
    .tile img {
      width: 100%;
      height: 100%;
    }
    #menu {
      margin: 20px auto;
    }
  </style>
</head>
<body>
  <!-- HTML structure -->
  <div id="menu">
    <select id="buildingType">
      <option value="factory">Factory ($20)</option>
      <option value="house">House ($15)</option>
    </select>
    <button onclick="build()">Build</button>
    <button onclick="saveGame()">Save Game</button>
    <button onclick="loadGame()">Load Game</button>
    <p>Money: <span id="money">50</span></p>
  </div>
  <div id="game"></div>

  <script>
    // JavaScript logic
    const grid = document.getElementById("game");
    const moneyDisplay = document.getElementById("money");

    let money = 50;
    const tiles = [];
    const earnings = 5;

    // Initialize grid
    for (let i = 0; i < 100; i++) {
      const tile = document.createElement("div");
      tile.className = "tile";
      tile.dataset.built = "false";
      tile.dataset.level = "0";
      tile.onclick = () => selectTile(i);
      grid.appendChild(tile);
      tiles.push(tile);
    }

    let selectedTile = null;

    function selectTile(index) {
      selectedTile = tiles[index];
      if (selectedTile.dataset.built === "true") {
        alert(`This tile is built! Level: ${selectedTile.dataset.level}`);
      } else {
        alert("This tile is empty. You can build here!");
      }
    }

    function build() {
      if (!selectedTile || selectedTile.dataset.built === "true") return;

      const buildingType = document.getElementById("buildingType").value;
      const cost = buildingType === "factory" ? 20 : 15;

      if (money >= cost) {
        money -= cost;
        selectedTile.dataset.built = "true";
        selectedTile.dataset.level = "1";
        selectedTile.dataset.type = buildingType;

        // Clear existing content and add image
        selectedTile.innerHTML = "";
        const img = document.createElement("img");
        img.src = buildingType === "factory" ? "factory.png" : "house.png";
        img.style.width = "100%";
        img.style.height = "100%";
        selectedTile.appendChild(img);

        updateMoney();
      } else {
        alert("Not enough money!");
      }
    }

    function saveGame() {
      const gameState = {
        money,
        tiles: tiles.map(tile => ({
          built: tile.dataset.built,
          level: tile.dataset.level,
          type: tile.dataset.type,
        })),
      };
      localStorage.setItem("tycoonGame", JSON.stringify(gameState));
      alert("Game saved!");
    }

    function loadGame() {
      const savedState = localStorage.getItem("tycoonGame");
      if (!savedState) return;
      const { money: savedMoney, tiles: savedTiles } = JSON.parse(savedState);
      money = savedMoney;
      updateMoney();
      savedTiles.forEach((tileData, index) => {
        const tile = tiles[index];
        tile.dataset.built = tileData.built;
        tile.dataset.level = tileData.level;
        tile.dataset.type = tileData.type;
        tile.className = tileData.built === "true" ? "tile built" : "tile";
        tile.innerHTML = "";
        if (tileData.built === "true") {
          const img = document.createElement("img");
          img.src = tileData.type === "factory" ? "factory.png" : "house.png";
          tile.appendChild(img);
        }
      });
    }

    function updateMoney() {
      moneyDisplay.textContent = money;
    }

    // Earnings loop
    setInterval(() => {
      tiles.forEach(tile => {
        if (tile.dataset.built === "true") {
          money += earnings * parseInt(tile.dataset.level);
        }
      });
      updateMoney();
    }, 3000);

    // Random events
    function triggerRandomEvent() {
      const eventType = Math.random() > 0.5 ? "good" : "bad";
      if (eventType === "good") {
        const bonus = Math.floor(Math.random() * 20) + 10;
        money += bonus;
        alert(`Good Event! You received a bonus of $${bonus}.`);
      } else {
        const fine = Math.floor(Math.random() * 15) + 5;
        money -= fine;
        alert(`Bad Event! You were fined $${fine}.`);
        if (money < 0) money = 0;
      }
      updateMoney();
    }

    setInterval(triggerRandomEvent, 20000);

    // Load game on page load
    window.onload = loadGame;
  </script>
</body>
</html>
