<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Strixhaven Arena Pro FIX</title>

<style>
body { background:#0b0b0b; color:#eee; font-family:Arial; }
h2 { margin:5px 0; }

.container { display:flex; gap:10px; }

.zone {
  flex:1;
  background:#111;
  padding:10px;
  height:90vh;
  overflow:auto;
  border-radius:8px;
}

.card {
  display:flex;
  align-items:center;
  justify-content:space-between;
  padding:5px;
  border-bottom:1px solid #222;
}

.card img {
  width:40px;
  margin-right:10px;
}

.left {
  display:flex;
  align-items:center;
  gap:8px;
}

button {
  background:#222;
  color:white;
  border:none;
  padding:4px 8px;
  cursor:pointer;
}
</style>
</head>
<body>

<h1>🧙 Strixhaven Arena Pro (FIX)</h1>

<div class="container">

<div class="zone">
<h2>📚 Coleção</h2>
<div id="collection"></div>
</div>

<div class="zone">
<h2>🎴 Pool (suas cartas)</h2>
<div id="pool"></div>
</div>

<div class="zone">
<h2>⚔️ Deck IA</h2>
<button onclick="generateDeck()">Gerar Deck</button>
<div id="deck"></div>
</div>

</div>

<script>

let cards = [];
let stats = {};

// =====================
// CARREGAR CARTAS
// =====================
async function loadCards(){
  let res = await fetch("https://api.scryfall.com/cards/search?q=set:stx");
  let data = await res.json();

  cards = data.data.map(c=>({
    name: c.name,
    cmc: c.cmc,
    colors: c.colors,
    type: c.type_line,
    text: c.oracle_text || "",
    img: c.image_uris?.small,
    qty: 0
  }));

  render();
}

// =====================
// 17LANDS (opcional)
// =====================
async function loadStats(){
  try {
    let res = await fetch("https://www.17lands.com/card_data?expansion=STX");
    let data = await res.json();

    data.forEach(c=>{
      stats[c.name] = c.gih_wr || 0;
    });
  } catch(e){
    console.log("17Lands não carregou (ok)");
  }
}

// =====================
// RENDER COLEÇÃO
// =====================
function render(){

  document.getElementById("collection").innerHTML =
    cards.map((c,i)=>`
      <div class="card">
        <div class="left">
          <img src="${c.img}">
          <div>
            <b>${c.name}</b><br>
            <small>Qtd: ${c.qty}</small>
          </div>
        </div>

        <div>
          <button onclick="add(${i})">+</button>
          <button onclick="remove(${i})">-</button>
        </div>
      </div>
    `).join("");

  renderPool();
}

// =====================
// POOL
// =====================
function renderPool(){

  let pool = cards.filter(c=>c.qty>0);

  document.getElementById("pool").innerHTML =
    pool.map(c=>`
      <div class="card">
        <div class="left">
          <img src="${c.img}">
          <b>${c.name}</b>
        </div>
      </div>
    `).join("");
}

// =====================
// CONTROLES
// =====================
function add(i){
  if(cards[i].qty < 4) cards[i].qty++;
  render();
}

function remove(i){
  if(cards[i].qty > 0) cards[i].qty--;
  render();
}

// =====================
// IA SIMPLES (FUNCIONAL)
// =====================
function score(c){

  let s = 0;

  if(stats[c.name]) s += stats[c.name] * 100;

  s += (6 - c.cmc);

  if(c.type.includes("Creature")) s += 2;
  if(c.text.includes("destroy")) s += 4;

  return s;
}

// =====================
// GERAR DECK
// =====================
function generateDeck(){

  let pool = [];

  cards.filter(c=>c.qty>0).forEach(c=>{
    for(let i=0;i<c.qty;i++) pool.push({...c});
  });

  if(pool.length === 0){
    alert("Adicione cartas primeiro!");
    return;
  }

  pool.forEach(c=>c.s = score(c));
  pool.sort((a,b)=>b.s-a.s);

  let deck = pool.slice(0,23);

  document.getElementById("deck").innerHTML =
    deck.map(c=>`
      <div class="card">
        <div class="left">
          <img src="${c.img}">
          <b>${c.name}</b>
        </div>
      </div>
    `).join("");
}

loadCards();
loadStats();

</script>

</body>
</html>
