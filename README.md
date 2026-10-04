# index.html
<!doctype html>
<html lang="ja">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no" />
<title>PHOTO FIGHTERS</title>

<style>
*{
  box-sizing:border-box;
}

body{
  margin:0;
  background:#111;
  color:#fff;
  font-family:system-ui,-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif;
  overflow:hidden;
}

#app{
  height:100vh;
  display:flex;
  flex-direction:column;
}

header{
  min-height:70px;
  display:flex;
  align-items:center;
  gap:12px;
  padding:10px 14px;
  background:linear-gradient(#181818,#0b0b0b);
  border-bottom:1px solid #333;
}

.title{
  font-weight:900;
  letter-spacing:1px;
  font-size:20px;
  white-space:nowrap;
}

.pick{
  display:flex;
  gap:8px;
  align-items:center;
  flex-wrap:wrap;
}

.pick label,
.btn{
  background:#fff;
  color:#111;
  padding:8px 12px;
  border-radius:10px;
  font-weight:800;
  cursor:pointer;
  border:none;
  font-size:13px;
}

input[type=file]{
  display:none;
}

#gameWrap{
  position:relative;
  flex:1;
  min-height:0;
}

canvas{
  width:100%;
  height:100%;
  display:block;
  background:#87ceeb;
  touch-action:none;
}

.hud{
  position:absolute;
  left:0;
  right:0;
  top:0;
  padding:14px;
  pointer-events:none;
}

.bars{
  display:flex;
  gap:14px;
}

.barShell{
  height:24px;
  background:#333;
  border:3px solid #fff;
  flex:1;
  border-radius:6px;
  overflow:hidden;
  box-shadow:0 2px 0 #000;
}

.bar{
  height:100%;
  background:linear-gradient(90deg,#f33,#ffb000);
}

.names{
  display:flex;
  justify-content:space-between;
  font-weight:900;
  margin-bottom:5px;
  text-shadow:0 2px 2px #000;
}

#centerMsg{
  position:absolute;
  inset:0;
  display:flex;
  align-items:center;
  justify-content:center;
  pointer-events:none;
  font-weight:1000;
  font-size:min(11vw,90px);
  text-shadow:0 5px 0 #000;
  letter-spacing:2px;
}

.controls{
  position:absolute;
  left:0;
  right:0;
  bottom:0;
  display:flex;
  justify-content:space-between;
  padding:12px;
  pointer-events:none;
}

.cluster{
  display:grid;
  grid-template-columns:64px 64px 64px;
  grid-template-rows:58px 58px;
  gap:8px;
  pointer-events:auto;
}

.cluster.right{
  grid-template-columns:72px 72px;
}

.cbtn{
  border:none;
  border-radius:16px;
  background:#ffffffd9;
  color:#111;
  font-size:22px;
  font-weight:900;
  box-shadow:0 4px 0 #555;
  touch-action:none;
  user-select:none;
  -webkit-user-select:none;
}

.cbtn:active,
.cbtn.active{
  transform:translateY(3px);
  box-shadow:0 1px 0 #555;
  background:#ffd54a;
}

.up{
  grid-column:2;
}

.left{
  grid-column:1;
  grid-row:2;
}

.down{
  grid-column:2;
  grid-row:2;
}

.rightBtn{
  grid-column:3;
  grid-row:2;
}

.attack{
  grid-column:1;
  grid-row:1 / span 2;
  background:#ff5f57;
  color:white;
  font-size:17px;
}

.jump{
  grid-column:2;
  grid-row:1 / span 2;
  font-size:17px;
}

.hint{
  position:absolute;
  right:12px;
  top:58px;
  font-size:11px;
  background:#0008;
  padding:6px 8px;
  border-radius:8px;
}

@media (max-width:700px){
  header{
    height:auto;
    min-height:94px;
    align-items:flex-start;
    flex-direction:column;
  }

  .title{
    font-size:17px;
  }

  .pick label,
  .btn{
    font-size:12px;
    padding:7px 10px;
  }

  .cluster{
    transform:scale(.9);
    transform-origin:bottom left;
  }

  .cluster.right{
    transform-origin:bottom right;
  }
}
</style>
</head>

<body>

<div id="app">

<header>
  <div class="title">🥊 PHOTO FIGHTERS</div>

  <div class="pick">
    <label>
      1P 写真を選ぶ
      <input id="p1file" type="file" accept="image/*">
    </label>

    <label>
      2P 写真を選ぶ
      <input id="p2file" type="file" accept="image/*">
    </label>

    <button class="btn" id="reset">
      RESTART
    </button>
  </div>
</header>

<div id="gameWrap">

  <canvas id="game"></canvas>

  <div class="hud">
    <div class="names">
      <span>PLAYER 1</span>
      <span>PLAYER 2</span>
    </div>

    <div class="bars">
      <div class="barShell">
        <div class="bar" id="p1bar"></div>
      </div>

      <div class="barShell">
        <div class="bar" id="p2bar"></div>
      </div>
    </div>
  </div>

  <div class="hint">
    PC: 1P A/D/W/F　2P ←/→/↑/L
  </div>

  <div id="centerMsg">
    FIGHT!
  </div>

  <div class="controls">

    <div class="cluster">
      <button class="cbtn up" data-key="KeyW">↑</button>
      <button class="cbtn left" data-key="KeyA">←</button>
      <button class="cbtn down" data-key="KeyS">↓</button>
      <button class="cbtn rightBtn" data-key="KeyD">→</button>
    </div>

    <div class="cluster right">
      <button class="cbtn attack" data-key="KeyF">
        PUNCH
      </button>

      <button class="cbtn jump" data-key="Space">
        JUMP
      </button>
    </div>

  </div>

</div>
</div>

<script>

const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

const msg = document.getElementById("centerMsg");

const p1bar = document.getElementById("p1bar");
const p2bar = document.getElementById("p2bar");

let DPR = Math.max(
  1,
  Math.min(2,devicePixelRatio || 1)
);

function fit(){

  const r =
    canvas.getBoundingClientRect();

  canvas.width =
    Math.floor(r.width * DPR);

  canvas.height =
    Math.floor(r.height * DPR);

  ctx.setTransform(
    DPR,0,0,DPR,0,0
  );
}

addEventListener(
  "resize",
  fit
);

fit();


const keys = {};

addEventListener(
  "keydown",
  e => {

    keys[e.code] = true;

    if(
      [
        "Space",
        "ArrowUp",
        "ArrowDown",
        "ArrowLeft",
        "ArrowRight"
      ].includes(e.code)
    ){
      e.preventDefault();
    }
  }
);

addEventListener(
  "keyup",
  e => {

    keys[e.code] = false;

  }
);


document
.querySelectorAll(".cbtn")
.forEach(b => {

  const k =
    b.dataset.key;

  const on = e => {

    e.preventDefault();

    keys[k] = true;

    b.classList.add("active");

  };

  const off = e => {

    e.preventDefault();

    keys[k] = false;

    b.classList.remove("active");

  };

  b.addEventListener(
    "pointerdown",
    on
  );

  b.addEventListener(
    "pointerup",
    off
  );

  b.addEventListener(
    "pointercancel",
    off
  );

  b.addEventListener(
    "pointerleave",
    off
  );

});


const floorPad = 95;

let last =
  performance.now();

let winner =
  null;


function makeFighter(
  x,
  facing,
  tint
){

  return {

    x,
    y:0,

    vx:0,
    vy:0,

    w:92,
    h:150,

    health:100,

    onGround:true,

    facing,

    attackTimer:0,

    hitFlash:0,

    img:null,

    tint

  };

}


let p1;
let p2;


function reset(){

  const W =
    canvas.getBoundingClientRect().width;

  const H =
    canvas.getBoundingClientRect().height;

  p1 =
    makeFighter(
      W * 0.25,
      1,
      "#2a7fff"
    );

  p2 =
    makeFighter(
      W * 0.75,
      -1,
      "#ff3d3d"
    );

  p1.y =
    H - floorPad - p1.h;

  p2.y =
    H - floorPad - p2.h;

  winner = null;

  msg.textContent =
    "FIGHT!";

  msg.style.opacity =
    1;

  setTimeout(
    () => {

      if(!winner){

        msg.style.opacity =
          0;

      }

    },
    700
  );

  updateBars();

}

document.getElementById(
  "reset"
).onclick =
reset;


function loadTo(
  fileInput,
  who
){

  const f =
    fileInput.files &&
    fileInput.files[0];

  if(!f)return;

  const img =
    new Image();

  img.onload =
    () => {

      who.img =
        img;

    };

  img.src =
    URL.createObjectURL(f);

}


document.getElementById(
  "p1file"
).onchange =
e =>
loadTo(
  e.target,
  p1
);


document.getElementById(
  "p2file"
).onchange =
e =>
loadTo(
  e.target,
  p2
);


function clamp(
  v,
  a,
  b
){

  return Math.max(
    a,
    Math.min(b,v)
  );

}


function rects(
  a,
  b
){

  return(
    a.x < b.x + b.w &&
    a.x + a.w > b.x &&
    a.y < b.y + b.h &&
    a.y + a.h > b.y
  );

}


function fighterInput(
  f,
  other,
  map,
  dt
){

  if(winner)return;

  const speed =
    330;

  let dir =
    (keys[map.left] ? -1 : 0)
    +
    (keys[map.right] ? 1 : 0);

  f.vx =
    dir * speed;


  if(dir){

    f.facing =
      dir > 0 ? 1 : -1;

  }


  if(
    keys[map.jump] &&
    f.onGround
  ){

    f.vy =
      -690;

    f.onGround =
      false;

    keys[map.jump] =
      false;

  }


  if(
    keys[map.attack] &&
    f.attackTimer <= 0
  ){

    f.attackTimer =
      .27;

    keys[map.attack] =
      false;

  }


  if(
    f.attackTimer > 0
  ){

    const active =
      f.attackTimer < .19 &&
      f.attackTimer > .08;


    if(
      active &&
      !f.didHit
    ){

      const reach =
        58;

      const hb = {

        x:
          f.facing > 0
          ?
          f.x + f.w - 4
          :
          f.x - reach + 4,

        y:
          f.y + 40,

        w:
          reach,

        h:
          60

      };


      if(
        rects(
          hb,
          other
        )
      ){

        other.health =
          Math.max(
            0,
            other.health - 10
          );

        other.vx =
          f.facing * 420;

        other.vy =
          -180;

        other.hitFlash =
          .16;

        f.didHit =
          true;

        updateBars();


        if(
          other.health <= 0
        ){

          endGame(
            f === p1
            ?
            "PLAYER 1"
            :
            "PLAYER 2"
          );

        }

      }

    }

  }
  else{

    f.didHit =
      false;

  }

}


function physics(
  f,
  dt
){

  const W =
    canvas.getBoundingClientRect().width;

  const H =
    canvas.getBoundingClientRect().height;

  const ground =
    H - floorPad;


  f.vy +=
    1500 * dt;

  f.x +=
    f.vx * dt;

  f.y +=
    f.vy * dt;


  if(
    f.y + f.h >= ground
  ){

    f.y =
      ground - f.h;

    f.vy =
      0;

    f.onGround =
      true;

  }


  f.x =
    clamp(
      f.x,
      0,
      W - f.w
    );


  f.attackTimer =
    Math.max(
      0,
      f.attackTimer - dt
    );


  f.hitFlash =
    Math.max(
      0,
      f.hitFlash - dt
    );

}


function resolveOverlap(
  a,
  b
){

  if(
    rects(a,b)
  ){

    const midA =
      a.x + a.w / 2;

    const midB =
      b.x + b.w / 2;


    const push =
      (a.w + b.w) / 2
      -
      Math.abs(
        midA - midB
      );


    if(
      push > 0
    ){

      const s =
        midA < midB
        ?
        -1
        :
        1;

      a.x +=
        s * push * .5;

      b.x -=
        s * push * .5;

    }

  }

}


function updateBars(){

  p1bar.style.width =
    p1.health + "%";

  p2bar.style.width =
    p2.health + "%";

}


function endGame(
  name
){

  winner =
    name;

  msg.textContent =
    name + " WINS!";

  msg.style.opacity =
    1;

}


function drawStage(
  W,
  H
){

  const sky =
    ctx.createLinearGradient(
      0,
      0,
      0,
      H
    );

  sky.addColorStop(
    0,
    "#87d4ff"
  );

  sky.addColorStop(
    .72,
    "#dff7ff"
  );

  sky.addColorStop(
    .721,
    "#8ed16a"
  );

  sky.addColorStop(
    1,
    "#5ea345"
  );


  ctx.fillStyle =
    sky;

  ctx.fillRect(
    0,
    0,
    W,
    H
  );


  ctx.fillStyle =
    "#78b95d";

  ctx.beginPath();

  ctx.moveTo(
    0,
    H-floorPad-30
  );

  for(
    let x=0;
    x<=W;
    x+=100
  ){

    ctx.lineTo(
      x,
      H-floorPad-30
      -
      Math.sin(x*.018)*22
    );

  }

  ctx.lineTo(
    W,
    H
  );

  ctx.lineTo(
    0,
    H
  );

  ctx.closePath();

  ctx.fill();


  ctx.fillStyle =
    "#d7b47b";

  ctx.fillRect(
    0,
    H-floorPad,
    W,
    floorPad
  );


  ctx.strokeStyle =
    "#b09060";

  ctx.lineWidth =
    2;


  for(
    let x=0;
    x<W;
    x+=50
  ){

    ctx.beginPath();

    ctx.moveTo(
      x,
      H-floorPad
    );

    ctx.lineTo(
      x+20,
      H
    );

    ctx.stroke();

  }

}


function drawFighter(
  f
){

  ctx.save();


  if(
    f.hitFlash > 0
  ){

    ctx.globalAlpha =
      .5;

  }


  ctx.translate(
    f.x + f.w/2,
    f.y + f.h/2
  );

  ctx.scale(
    f.facing,
    1
  );


  const bodyX =
    -f.w/2;

  const bodyY =
    -f.h/2;


  ctx.save();

  ctx.scale(
    f.facing,
    1
  );

  ctx.fillStyle =
    "#0004";

  ctx.beginPath();

  ctx.ellipse(
    0,
    f.h/2 + 8,
    58,
    13,
    0,
    0,
    Math.PI * 2
  );

  ctx.fill();

  ctx.restore();


  ctx.fillStyle =
    f.tint;

  ctx.strokeStyle =
    "#111";

  ctx.lineWidth =
    5;


  roundRect(
    ctx,
    bodyX,
    bodyY,
    f.w,
    f.h,
    20
  );

  ctx.fill();

  ctx.stroke();


  if(
    f.img
  ){

    ctx.save();


    roundRect(
      ctx,
      bodyX+6,
      bodyY+6,
      f.w-12,
      f.h-12,
      15
    );

    ctx.clip();


    const iw =
      f.img.width;

    const ih =
      f.img.height;

    const tw =
      f.w - 12;

    const th =
      f.h - 12;


    const scale =
      Math.max(
        tw/iw,
        th/ih
      );


    const dw =
      iw * scale;

    const dh =
      ih * scale;


    ctx.drawImage(
      f.img,
      bodyX+6+(tw-dw)/2,
      bodyY+6+(th-dh)/2,
      dw,
      dh
    );


    ctx.restore();

  }
  else{

    ctx.fillStyle =
      "#fff";

    ctx.font =
      "900 44px system-ui";

    ctx.textAlign =
      "center";

    ctx.textBaseline =
      "middle";

    ctx.fillText(
      "?",
      0,
      -6
    );


    ctx.font =
      "800 12px system-ui";

    ctx.fillText(
      "PHOTO",
      0,
      34
    );

  }


  ctx.strokeStyle =
    "#111";

  ctx.lineWidth =
    10;

  ctx.lineCap =
    "round";


  ctx.beginPath();

  ctx.moveTo(
    -20,
    f.h/2-4
  );

  ctx.lineTo(
    -28,
    f.h/2+34
  );

  ctx.moveTo(
    20,
    f.h/2-4
  );

  ctx.lineTo(
    28,
    f.h/2+34
  );

  ctx.stroke();


  ctx.beginPath();

  ctx.moveTo(
    -f.w/2+5,
    -10
  );

  ctx.lineTo(
    -f.w/2-28,
    16
  );

  ctx.stroke();


  ctx.beginPath();

  ctx.moveTo(
    f.w/2-4,
    -12
  );


  if(
    f.attackTimer>0 &&
    f.attackTimer<.22
  ){

    ctx.lineTo(
      f.w/2+62,
      -10
    );

  }
  else{

    ctx.lineTo(
      f.w/2+26,
      16
    );

  }

  ctx.stroke();


  ctx.fillStyle =
    "#fff";

  ctx.strokeStyle =
    "#111";

  ctx.lineWidth =
    4;


  const fx =
    (
      f.attackTimer>0 &&
      f.attackTimer<.22
    )
    ?
    f.w/2+65
    :
    f.w/2+30;


  ctx.beginPath();

  ctx.arc(
    fx,
    (
      f.attackTimer>0 &&
      f.attackTimer<.22
    )
    ?
    -10
    :
    16,
    12,
    0,
    Math.PI*2
  );

  ctx.fill();

  ctx.stroke();


  ctx.restore();

}


function roundRect(
  c,
  x,
  y,
  w,
  h,
  r
){

  r =
    Math.min(
      r,
      w/2,
      h/2
    );

  c.beginPath();

  c.moveTo(
    x+r,
    y
  );

  c.arcTo(
    x+w,
    y,
    x+w,
    y+h,
    r
  );

  c.arcTo(
    x+w,
    y+h,
    x,
    y+h,
    r
  );

  c.arcTo(
    x,
    y+h,
    x,
    y,
    r
  );

  c.arcTo(
    x,
    y,
    x+w,
    y,
    r
  );

  c.closePath();

}


function loop(
  t
){

  const dt =
    Math.min(
      .03,
      (t-last)/1000
    );

  last =
    t;


  const r =
    canvas.getBoundingClientRect();

  const W =
    r.width;

  const H =
    r.height;


  fighterInput(
    p1,
    p2,
    {
      left:"KeyA",
      right:"KeyD",
      jump:"KeyW",
      attack:"KeyF"
    },
    dt
  );


  fighterInput(
    p2,
    p1,
    {
      left:"ArrowLeft",
      right:"ArrowRight",
      jump:"ArrowUp",
      attack:"KeyL"
    },
    dt
  );


  physics(
    p1,
    dt
  );

  physics(
    p2,
    dt
  );


  resolveOverlap(
    p1,
    p2
  );


  drawStage(
    W,
    H
  );


  drawFighter(
    p1
  );

  drawFighter(
    p2
  );


  requestAnimationFrame(
    loop
  );

}


reset();

requestAnimationFrame(
  loop
);

</script>

</body>
</html>