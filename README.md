<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Ultimate Game Hub</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

<style>
body {
    margin: 0;
    background: black;
    color: white;
    font-family: 'Poppins', sans-serif;
    overflow-x: hidden;
    display: flex;
    flex-direction: column;
    align-items: center;
}

canvas {
    position: fixed;
    top: 0;
    left: 0;
    z-index: -1;
}

header {
    text-align: center;
    font-size: 50px;
    padding: 20px;
    text-shadow: 0 0 10px cyan;
}

nav {
    display: flex;
    justify-content: center;
    gap: 20px;
    margin-bottom: 30px;
}

nav button {
    padding: 15px 25px;
    font-size: 18px;
    font-weight: bold;
    border: none;
    border-radius: 12px;
    background: #00ffff;
    color: black;
    cursor: pointer;
    transition: transform 0.2s, background 0.2s;
}

nav button:hover {
    transform: scale(1.05);
    background: #00cccc;
}

/* Container for all games to sit below Nav */
#game-container {
    width: 100%;
    display: flex;
    justify-content: center;
    padding-bottom: 50px;
}

.gamePopup {
    background: rgba(0, 0, 0, 0.85);
    border: 2px solid cyan;
    padding: 30px;
    border-radius: 20px;
    display: none; /* Hidden by default */
    text-align: center;
    box-shadow: 0 0 20px rgba(0, 255, 255, 0.3);
    min-width: 320px;
}

.show {
    display: block; /* Visible when active */
}

.board {
    display: grid;
    grid-template-columns: repeat(3, 100px);
    gap: 10px;
    justify-content: center;
    margin: 20px 0;
}

.cell {
    height: 100px;
    background: #111;
    border: 1px solid #333;
    font-size: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
}

.cell:hover { background: #222; }

.memoryGrid {
    display: grid;
    grid-template-columns: repeat(4, 80px);
    gap: 10px;
    justify-content: center;
    margin-top: 20px;
}

.card {
    height: 80px;
    background: #111;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 30px;
    cursor: pointer;
    border: 1px solid #333;
}

.timer {
    font-size: 40px;
    color: #00ff9f;
    font-family: monospace;
    margin: 10px 0;
}

.light {
    width: 50px;
    height: 50px;
    border-radius: 50%;
    background: #400;
    margin: 8px;
    display: inline-block;
    border: 2px solid #200;
}

.light.on { background: red; box-shadow: 0 0 15px red; }
.light.go { background: #00ff9f; box-shadow: 0 0 15px #00ff9f; }

select, button.reset-btn {
    padding: 8px;
    border-radius: 5px;
    margin-top: 10px;
    cursor: pointer;
}
</style>
</head>

<body>

<canvas id="bg"></canvas>

<header>🔥  Game Hub</header>

<nav>
    <button onclick="openGame('tic')">Tic Tac Toe</button>
    <button onclick="openGame('memory')">Memory</button>
    <button onclick="openGame('reaction')">Reaction</button>
</nav>

<div id="game-container">
    <div id="tic" class="gamePopup">
        <h2>Tic Tac Toe</h2>
        <select id="mode">
            <option value="cpu">User vs AI</option>
            <option value="pvp">User vs User</option>
        </select>
        <div class="board" id="board"></div>
        <p id="status"></p>
        <button class="reset-btn" onclick="resetGame()">Reset Game</button>
    </div>

    <div id="memory" class="gamePopup">
        <h2>Memory Game</h2>
        <div class="timer" id="memTimer">0.000s</div>
        <div class="memoryGrid" id="memGrid"></div>
        <button class="reset-btn" onclick="initMemory()">New Game</button>
    </div>

    <div id="reaction" class="gamePopup">
        <h2>Reaction Test</h2>
        <p>Wait for Green, then Click Anywhere!</p>
        <div id="lights"></div>
        <div class="timer" id="timer">0.000s</div>
        <button class="reset-btn" onclick="startReaction()">Start Test</button>
    </div>
</div>

<script>
function openGame(id){
    document.querySelectorAll('.gamePopup').forEach(g=>g.classList.remove('show'));
    document.getElementById(id).classList.add('show');
}

/* 🎉 CONFETTI */
function confetti(){
    for(let i=0;i<80;i++){
        let d=document.createElement('div');
        d.style.position='fixed';
        d.style.width='6px';
        d.style.height='6px';
        d.style.background=`hsl(${Math.random()*360},100%,50%)`;
        d.style.top='50%'; d.style.left='50%';
        d.style.zIndex='100';
        document.body.appendChild(d);
        let x=(Math.random()-0.5)*800;
        let y=(Math.random()-0.5)*800;
        d.animate([{transform:'translate(0,0)'},{transform:`translate(${x}px,${y}px)`}],{duration:1000});
        setTimeout(()=>d.remove(),1000);
    }
}

/* TIC TAC TOE */
let boardArr=["","","","","","","","",""];
let current="X", gameOver=false;
const wins=[[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]];
const board = document.getElementById("board");
const status = document.getElementById("status");
const mode = document.getElementById("mode");

function draw(){
    board.innerHTML="";
    boardArr.forEach((v,i)=>{
        let d=document.createElement("div");
        d.className="cell";
        d.innerText=v;
        d.onclick=()=>move(i);
        board.appendChild(d);
    });
}

function move(i){
    if(boardArr[i]||gameOver) return;
    if(mode.value==="cpu" && current==="O") return;

    boardArr[i]=current;
    draw();
    if(checkWin()) return;

    if(mode.value==="cpu"){
        current="O";
        setTimeout(aiMove,200);
    }else current=current==="X"?"O":"X";
}

function aiMove(){
    let best=-Infinity,movePos;
    for(let i=0;i<9;i++){
        if(!boardArr[i]){
            boardArr[i]="O";
            let score=minimax(boardArr,false);
            boardArr[i]="";
            if(score>best){best=score;movePos=i;}
        }
    }
    boardArr[movePos]="O";
    draw();
    if(checkWin()) return;
    current="X";
}

function minimax(b,isMax){
    let res=evaluate(b);
    if(res!==null) return res;

    if(isMax){
        let best=-Infinity;
        for(let i=0;i<9;i++){
            if(!b[i]){
                b[i]="O";
                best=Math.max(best,minimax(b,false));
                b[i]="";
            }
        }
        return best;
    }else{
        let best=Infinity;
        for(let i=0;i<9;i++){
            if(!b[i]){
                b[i]="X";
                best=Math.min(best,minimax(b,true));
                b[i]="";
            }
        }
        return best;
    }
}

function evaluate(b){
    for(let w of wins){
        let[a,c,d]=w;
        if(b[a]&&b[a]==b[c]&&b[a]==b[d]){
            return b[a]=="O"?10:-10;
        }
    }
    if(b.every(x=>x)) return 0;
    return null;
}

function checkWin(){
    for(let w of wins){
        let[a,b,c]=w;
        if(boardArr[a]&&boardArr[a]==boardArr[b]&&boardArr[a]==boardArr[c]){
            status.innerText=boardArr[a]+" Wins!";
            confetti();
            gameOver=true;
            return true;
        }
    }
    if(boardArr.every(x=>x)){
        status.innerText="Draw!";
        gameOver=true;
        return true;
    }
    return false;
}

function resetGame(){
    boardArr=["","","","","","","","",""];
    current="X"; gameOver=false; status.innerText="";
    draw();
}
draw();

/* MEMORY */
let memSymbols=["🍎","🍌","🍇","🍒","🍍","🥝","🍉","🍑"];
let memArr=[],first=null,lock=false;
let timerInterval,startTime,started=false;
const memTimer = document.getElementById("memTimer");
const memGrid = document.getElementById("memGrid");

function initMemory(){
    clearInterval(timerInterval);
    memTimer.innerText="0.000s";
    started=false;
    memGrid.innerHTML="";
    memArr=[...memSymbols,...memSymbols].sort(()=>Math.random()-0.5);
    first=null; lock=false;

    memArr.forEach(sym=>{
        let c=document.createElement("div");
        c.className="card";
        c.dataset.val=sym;
        c.onclick=()=>flip(c);
        memGrid.appendChild(c);
    });
}

function flip(c){
    if(lock||c.innerText) return;

    if(!started){
        started=true;
        startTime=performance.now();
        timerInterval=setInterval(()=>{
            let t=(performance.now()-startTime)/1000;
            memTimer.innerText=t.toFixed(3)+"s";
        },50);
    }

    c.innerText=c.dataset.val;

    if(!first) first=c;
    else{
        if(first.dataset.val===c.dataset.val){
            first=null;
            if([...memGrid.children].every(x=>x.innerText)){
                clearInterval(timerInterval);
                confetti();
            }
        }else{
            lock=true;
            setTimeout(()=>{
                c.innerText=""; first.innerText="";
                first=null; lock=false;
            },600);
        }
    }
}
initMemory();

/* REACTION */
let lightsDiv=document.getElementById("lights"),lights=[];
const reactionTimerDisplay = document.getElementById("timer");

function createLights(){
    lightsDiv.innerHTML="";
    lights=[];
    for(let i=0;i<5;i++){
        let l=document.createElement("div");
        l.className="light";
        lightsDiv.appendChild(l);
        lights.push(l);
    }
}
createLights();

let reactionStart, reactionInterval;

function startReaction(){
    createLights();
    reactionTimerDisplay.innerText="0.000s";

    let delay=0;
    lights.forEach((l,i)=>{
        setTimeout(()=>l.classList.add("on"),delay+=700);
    });

    setTimeout(()=>{
        lights.forEach(l=>l.classList.remove("on"));
        lights.forEach(l=>l.classList.add("go"));

        reactionStart=performance.now();

        reactionInterval=setInterval(()=>{
            let t=(performance.now()-reactionStart)/1000;
            reactionTimerDisplay.innerText=t.toFixed(3)+"s";
        },10);

        const stopFunc = ()=>{
            clearInterval(reactionInterval);
            window.removeEventListener('mousedown', stopFunc);
        };
        window.addEventListener('mousedown', stopFunc);

    },delay+Math.random()*1500);
}

/* BACKGROUND */
const scene=new THREE.Scene();
const camera=new THREE.PerspectiveCamera(75,window.innerWidth/window.innerHeight,0.1,1000);
const renderer=new THREE.WebGLRenderer({canvas:bg, antialias: true});
renderer.setSize(window.innerWidth,window.innerHeight);
camera.position.z=8;

let rings=[];
for(let i=0;i<6;i++){
    let geo=new THREE.TorusGeometry(2+i*0.5,0.05,16,100);
    let mat=new THREE.MeshBasicMaterial({color:0x00ffff,wireframe:true});
    let mesh=new THREE.Mesh(geo,mat);
    scene.add(mesh);
    rings.push(mesh);
}

let pts=[];
for(let x=-10;x<10;x++){
    for(let y=-10;y<10;y++){
        pts.push(x,y,Math.sin(x*y));
    }
}
const g=new THREE.BufferGeometry();
g.setAttribute('position',new THREE.Float32BufferAttribute(pts,3));
const p=new THREE.Points(g,new THREE.PointsMaterial({size:0.1, color: 0x00ffff}));
scene.add(p);

function animate(){
    requestAnimationFrame(animate);
    rings.forEach((r,i)=>{
        r.rotation.x+=0.002+i*0.001;
        r.rotation.y+=0.003+i*0.001;
    });
    p.rotation.y+=0.002;
    renderer.render(scene,camera);
}
animate();

window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
});
</script>

</body>
</html>
