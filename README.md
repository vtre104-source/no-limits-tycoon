# no-limits-tycoon
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>NO LIMITS TYCOON</title>
<style>
body { margin:0; background:#111; color:white; font-family:monospace; }
canvas { background:#222; display:block; margin:auto; }
button { font-size:14px; margin:4px; }
</style>
</head>

<body>
<canvas id="game" width="800" height="400"></canvas>
<div>
<button onclick="addPistons()">+ Pistons</button>
<button onclick="addWheels()">+ Wheels</button>
<button onclick="addTurbos()">+ Turbo</button>
<button onclick="addPSI()">+ PSI</button>
<button onclick="addRocket()">+ Rocket</button>
<button onclick="toggleMode()">Toggle Offroad</button>
</div>

<script>
// ================= CORE STATE =================
const c = document.getElementById("game");
const ctx = c.getContext("2d");

let offroad = false;

// ================= VEHICLE =================
let car = {
  mass: 600,
  structure: 0.6,
  pistons: 4,
  wheels: 4,
  drivenWheels: 2,
  turbos: 1,
  psi: 10,
  rockets: 0,
  speed: 0,
  torque: 200,
  integrity: 100,
  control: 1,
  pistonsAlive: 4
};

// ================= INPUT =================
let throttle = false;
document.body.onmousedown = () => throttle = true;
document.body.onmouseup = () => throttle = false;
document.body.onkeydown = e => {
  if (e.key === "r") igniteRockets();
};

// ================= BUILDER =================
function addPistons() {
  car.pistons++;
  car.pistonsAlive++;
  car.mass += 10;
  car.torque += 15;
}
function addWheels() {
  car.wheels++;
  car.mass += 20;
}
function addTurbos() {
  car.turbos++;
}
function addPSI() {
  car.psi += 10;
}
function addRocket() {
  car.rockets++;
  car.mass += 30;
}
function toggleMode() {
  offroad = !offroad;
}

// ================= PHYSICS =================
function update() {
  let boost = car.turbos * car.psi * 0.6;
  let engineForce = (car.torque + boost) * (car.pistonsAlive / car.pistons);

  if (throttle) {
    car.speed += engineForce / car.mass;
  }

  // drag / terrain
  car.speed -= offroad ? 0.3 : 0.1;
  if (car.speed < 0) car.speed = 0;

  // piston failure
  if (boost > 500 && Math.random() < 0.02) ejectPiston();

  // wheel loss
  if (engineForce > 2000 && Math.random() < 0.01) loseWheel();

  // integrity decay
  if (car.integrity <= 0) car.speed *= 0.95;
}

// ================= FAILURES =================
function ejectPiston() {
  if (car.pistonsAlive <= 1) return;

  car.pistonsAlive--;
  car.integrity -= 10;
  console.log("💥 PISTON EJECTED");
}

function loseWheel() {
  if (car.wheels <= 1) return;

  car.wheels--;
  car.integrity -= 15;
  car.control *= 0.7;
  console.log("🛞 WHEEL LOST");
}

// ================= ROCKET
