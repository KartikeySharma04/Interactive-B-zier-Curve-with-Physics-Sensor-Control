# Interactive-B-zier-Curve-with-Physics-Sensor-Control
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>Interactive Bézier Rope</title>
<style>
body {
  margin: 0;
  background: #0f1720;
  overflow: hidden;
}
canvas {
  display: block;
}
</style>
</head>
<body>
<canvas id="c"></canvas>

<script>
// =====================================================
// Vector Math (manual)
// =====================================================
class Vec {
  constructor(x=0,y=0){ this.x=x; this.y=y }
  copy(){ return new Vec(this.x,this.y) }
  add(v){ this.x+=v.x; this.y+=v.y; return this }
  sub(v){ this.x-=v.x; this.y-=v.y; return this }
  mul(s){ this.x*=s; this.y*=s; return this }
  len(){ return Math.hypot(this.x,this.y) }
  norm(){ let l=this.len()||1; this.x/=l; this.y/=l; return this }
  static sub(a,b){ return new Vec(a.x-b.x,a.y-b.y) }
}

// =====================================================
// Canvas Setup
// =====================================================
const canvas = document.getElementById("c");
const ctx = canvas.getContext("2d");

function resize(){
  canvas.width = innerWidth;
  canvas.height = innerHeight;
}
window.addEventListener("resize", resize);
resize();

// =====================================================
// Bézier Math (NO LIBRARIES)
// =====================================================
function bezier(p0,p1,p2,p3,t){
  const u = 1 - t;
  return new Vec(
    u*u*u*p0.x + 3*u*u*t*p1.x + 3*u*t*t*p2.x + t*t*t*p3.x,
    u*u*u*p0.y + 3*u*u*t*p1.y + 3*u*t*t*p2.y + t*t*t*p3.y
  );
}

function bezierDerivative(p0,p1,p2,p3,t){
  const u = 1 - t;
  return new Vec(
    3*u*u*(p1.x-p0.x) + 6*u*t*(p2.x-p1.x) + 3*t*t*(p3.x-p2.x),
    3*u*u*(p1.y-p0.y) + 6*u*t*(p2.y-p1.y) + 3*t*t*(p3.y-p2.y)
  );
}

// =====================================================
// Spring-Damper Physics
// =====================================================
class Spring {
  constructor(x,y){
    this.pos = new Vec(x,y);
    this.vel = new Vec();
    this.anchor = new Vec(x,y);
    this.target = new Vec(x,y);
    this.k = 80;        // stiffness
    this.damping = 16;  // increased damping (stable)
  }

  step(dt){
    // a = -k(x - target) - d*v
    let force = Vec.sub(this.pos, this.target).mul(-this.k);
    force.add(this.vel.copy().mul(-this.damping));
    this.vel.add(force.mul(dt));
    this.pos.add(this.vel.copy().mul(dt));
  }
}

// =====================================================
// Control Points
// =====================================================
const P0 = new Vec(100, innerHeight/2);          // fixed
const P3 = new Vec(innerWidth-100, innerHeight/2); // fixed

const P1 = new Spring(innerWidth*0.3, innerHeight/2);
const P2 = new Spring(innerWidth*0.7, innerHeight/2);

// =====================================================
// Mouse Input (Web interaction)
// =====================================================
let mouse = new Vec(innerWidth/2, innerHeight/2);

window.addEventListener("mousemove", e=>{
  mouse.x = e.clientX;
  mouse.y = e.clientY;

  const dx = (mouse.x - innerWidth/2) / innerWidth * 220;
  const dy = (mouse.y - innerHeight/2) / innerHeight * 220;

  P1.target = new Vec(P1.anchor.x + dx*0.6, P1.anchor.y + dy);
  P2.target = new Vec(P2.anchor.x + dx,     P2.anchor.y + dy);
});

// =====================================================
// Drawing Helpers
// =====================================================
function drawPoint(p,color){
  ctx.beginPath();
  ctx.arc(p.x,p.y,7,0,Math.PI*2);
  ctx.fillStyle = color;
  ctx.fill();
}

// =====================================================
// Main Loop (60 FPS stable)
// =====================================================
let last = performance.now();

function loop(now){
  // CLAMP dt to avoid physics explosions
  let dt = Math.min(0.016, (now - last) / 1000);
  last = now;

  P1.step(dt);
  P2.step(dt);

  ctx.clearRect(0,0,canvas.width,canvas.height);

  // Draw Bézier curve
  ctx.beginPath();
  for(let t=0;t<=1;t+=0.01){
    let p = bezier(P0,P1.pos,P2.pos,P3,t);
    t===0 ? ctx.moveTo(p.x,p.y) : ctx.lineTo(p.x,p.y);
  }
  ctx.strokeStyle = "#4dd0e1";
  ctx.lineWidth = 3;
  ctx.stroke();

  // Draw tangents
  for(let t=0;t<=1;t+=0.1){
    let p = bezier(P0,P1.pos,P2.pos,P3,t);
    let d = bezierDerivative(P0,P1.pos,P2.pos,P3,t)
              .norm()
              .mul(18); // reduced for stability
    ctx.beginPath();
    ctx.moveTo(p.x-d.x, p.y-d.y);
    ctx.lineTo(p.x+d.x, p.y+d.y);
    ctx.strokeStyle = "white";
    ctx.lineWidth = 2;
    ctx.stroke();
  }

  // Draw control points
  drawPoint(P0,"red");        // fixed endpoints
  drawPoint(P3,"red");
  drawPoint(P1.pos,"yellow"); // dynamic control points
  drawPoint(P2.pos,"yellow");

  requestAnimationFrame(loop);
}

requestAnimationFrame(loop);
</script>
</body>
</html>
