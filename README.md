<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Batman – Arkham Escape V2</title>
<style>
body { font-family: Arial, sans-serif; background:#111; color:#eee; display:flex; justify-content:center; padding:20px; }
#game { max-width:600px; width:100%; text-align:center; background:#1c1c1c; padding:20px; border-radius:10px; border:2px solid #444; }
h1 { color:#f39c12; }
button { padding:10px 20px; margin:5px; font-size:16px; cursor:pointer; border-radius:5px; border:none; }
#log { margin-top:20px; background:#222; border:1px solid #555; padding:10px; height:200px; overflow-y:auto; text-align:left; }
.avatar { font-size:50px; margin:10px; }
.stats { margin:10px 0; }
.hpbar { width:100%; height:14px; background:#333; border-radius:6px; overflow:hidden; margin-top:4px; }
.hpfill { height:100%; transition:width .3s; background:linear-gradient(90deg,#2ecc71,#1abc9c); }
.enemyfill { background:linear-gradient(90deg,#ff6b6b,#ff4d4d); }
.inventory { display:flex; flex-wrap:wrap; justify-content:center; gap:5px; margin-top:5px; }
.item { background:#222; padding:4px 8px; border-radius:5px; font-size:14px; border:1px solid #444; }
</style>
</head>
<body>
<div id="game">
<h1>🦇 Batman – Arkham Escape</h1>
<div id="scene"></div>
<div id="stats"></div>
<div id="actions"></div>
<div id="log"></div>
</div>

<script>
// --- État du joueur ---
let batman = { pv:40, max:40, objets:[], hasEvade:false, batarang:false };
let encounters = 0; // nb d'épreuves réussies
let currentEnemy = null;
let combatEnCours = false;

// --- Utilitaires ---
function log(msg){ let l=document.getElementById('log'); l.innerHTML+="<div>"+msg+"</div>"; l.scrollTop=l.scrollHeight; }
function rnd(min,max){ return Math.floor(Math.random()*(max-min+1))+min; }
function majStats(){
  let s=document.getElementById('stats');
  let html=`<div class="avatar">🦇</div>
  <div class="stats">PV Batman: ${batman.pv}/${batman.max}<div class="hpbar"><div class="hpfill" style="width:${batman.pv/batman.max*100}%"></div></div></div>`;
  if(batman.objets.length>0){
    html+=`<div>Objets:<div class="inventory">`;
    batman.objets.forEach((it,i)=>{ html+=`<div class="item">${it}</div>`; });
    html+=`</div></div>`;
  }
  s.innerHTML=html;
}
function majScene(html){ document.getElementById('scene').innerHTML = html; }
function majActions(html){ document.getElementById('actions').innerHTML = html; }

// --- Début du jeu ---
function startGame(){
  batman={ pv:40, max:40, objets:[], hasEvade:false, batarang:false };
  encounters=0;
  combatEnCours=false;
  majStats();
  log("🦇 Batman s'éveille dans Arkham !");
  choixPorte();
}

// --- Choix de sortie ---
function choixPorte(){
  majScene("<p>Choisis une sortie :</p>");
  majActions(`
    <button onclick="rencontre('gauche')">🚪 Porte gauche</button>
    <button onclick="rencontre('droite')">🚪 Porte droite</button>
    <button onclick="rencontre('ascenseur')">⬆️ Ascenseur</button>
  `);
}

// --- Déroulement des rencontres ---
function rencontre(dir){
  if(dir==="gauche"){ currentEnemy={nom:"Joker", pv:20, max:20, dmg:[2,5], emoji:"🤡", type:"combat"}; }
  else if(dir==="droite"){ currentEnemy={nom:"Bane", pv:25, max:25, dmg:[3,7], emoji:"💪", type:"combat"}; }
  else { currentEnemy={nom:"L'Homme-Mystère", type:"riddle"}; }
  
  if(currentEnemy.type==="combat"){ debutCombat(); }
  else { enigmeRiddler(); }
}

// --- Combat ---
function debutCombat(){
  combatEnCours=true;
  majScene(`<div class="avatar">${currentEnemy.emoji}</div><p>${currentEnemy.nom} vous fait face !</p>`);
  majActions(`
    <button onclick="attaque('rapide')">⚡ Attaque rapide</button>
    <button onclick="attaque('lourde')">💥 Attaque lourde</button>
    <button onclick="esquive()">🤸 Esquive</button>
  `);
  majStats();
  log(`💥 Combat contre ${currentEnemy.nom} !`);
}

function attaque(type){
  if(!combatEnCours) return;
  let dmg=0;
  if(type==="rapide") dmg=rnd(3,5);
  if(type==="lourde") dmg=rnd(5,9);
  if(batman.batarang){ dmg=Math.round(dmg*1.5); batman.batarang=false; log("🛠️ Batarang utilisé : dégâts augmentés !"); }
  currentEnemy.pv-=dmg;
  log(`🦇 Batman inflige ${dmg} PV à ${currentEnemy.nom}.`);
  if(currentEnemy.pv<=0){ victoire(); return; }
  riposte();
}

function esquive(){
  log("🦇 Batman tente d'esquiver...");
  if(Math.random()<0.5){ log("❌ Échec ! L'ennemi attaque."); riposte(); }
  else { log("✅ Esquive réussie !"); }
}

function riposte(){
  let dmg=rnd(currentEnemy.dmg[0], currentEnemy.dmg[1]);
  if(batman.hasEvade){ dmg=0; batman.hasEvade=false; log("🛡️ Esquive ou fumigène bloque l'attaque !"); }
  batman.pv-=dmg;
  log(`${currentEnemy.emoji} ${currentEnemy.nom} inflige ${dmg} PV à Batman !`);
  majStats();
  if(batman.pv<=0){ defaite(); return; }
}

// --- Énigme Homme-Mystère ---
function enigmeRiddler(){
  majScene(`<div class="avatar">❓</div><p>L’Homme-Mystère : "Je peux être brisé sans être touché, qui suis-je ?"</p>`);
  majActions(`
    <button onclick="reponseEnigme('secret')">Un secret</button>
    <button onclick="reponseEnigme('verre')">Un verre</button>
    <button onclick="reponseEnigme('coeur')">Un cœur</button>
  `);
}

function reponseEnigme(rep){
  if(rep==="secret"){ log("✅ Bonne réponse !"); victoire(); }
  else { batman.pv-=5; log("❌ Mauvaise réponse : -5 PV"); majStats(); if(batman.pv<=0) defaite(); else victoire(); }
}

// --- Fin combat / rencontre ---
function victoire(){
  combatEnCours=false;
  encounters++;
  // récompense aléatoire
  const items=["Fumigène","Batarang","Trousse"];
  const item=items[rnd(0,items.length-1)];
  batman.objets.push(item);
  log(`🎁 Vous obtenez un objet : ${item}`);
  majStats();
  if(encounters>=2){ majScene("<p>🏆 Batman atteint le toit et s'échappe !</p>"); majActions(`<button onclick="startGame()">🔄 Rejouer</button>`); return; }
  majScene("<p>✅ Épreuve terminée ! Choisis la prochaine sortie.</p>");
  choixPorte();
}

function defaite(){
  combatEnCours=false;
  majScene("<p>☠️ Batman est vaincu et reste enfermé dans Arkham...</p>");
  majActions(`<button onclick="startGame()">🔄 Rejouer</button>`);
}

// --- Lancement ---
startGame();
</script>
</body>
</html>
