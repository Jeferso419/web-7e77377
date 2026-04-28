<script>
window.requestAnimationFrame =
window.__requestAnimationFrame ||
window.requestAnimationFrame ||
window.webkitRequestAnimationFrame ||
window.mozRequestAnimationFrame ||
window.oRequestAnimationFrame ||
window.msRequestAnimationFrame ||
(function () {
  return function (callback) {
    return setTimeout(callback, 1000 / 60);
  };
})();

window.esDispositivo = (/android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i.test(
  (navigator.userAgent || navigator.vendor || window.opera).toLowerCase()
));

let cargado = false;

let iniciar = function () {
  if (cargado) return;
  cargado = true;

  const lienzo = document.getElementById('heart');
  const ctx = lienzo.getContext('2d');

  let ancho, alto, dpr;

  function ajustarCanvas() {
    dpr = window.devicePixelRatio || 1;

    ancho = innerWidth;
    alto = innerHeight;

    lienzo.width = ancho * dpr;
    lienzo.height = alto * dpr;

    lienzo.style.width = ancho + "px";
    lienzo.style.height = alto + "px";

    ctx.setTransform(1, 0, 0, 1, 0, 0); // reset
    ctx.scale(dpr, dpr);
  }

  ajustarCanvas();
  window.addEventListener('resize', ajustarCanvas);

  const aleatorio = Math.random;

  ctx.fillStyle = "black";
  ctx.fillRect(0, 0, ancho, alto);

  const posicionCorazon = (rad) => [
    Math.pow(Math.sin(rad), 3),
    -(15 * Math.cos(rad) - 5 * Math.cos(2 * rad) - 2 * Math.cos(3 * rad) - Math.cos(4 * rad))
  ];

  const escalarYTraducir = (pos, sx, sy, dx, dy) => [
    dx + pos[0] * sx,
    dy + pos[1] * sy
  ];

  let cantidadRastro = esDispositivo ? 12 : 50;
  let dr = esDispositivo ? 0.4 : 0.1;

  let puntosOrigen = [];
  for (let i = 0; i < Math.PI * 2; i += dr)
    puntosOrigen.push(escalarYTraducir(posicionCorazon(i), 210, 13, 0, 0));
  for (let i = 0; i < Math.PI * 2; i += dr)
    puntosOrigen.push(escalarYTraducir(posicionCorazon(i), 150, 9, 0, 0));
  for (let i = 0; i < Math.PI * 2; i += dr)
    puntosOrigen.push(escalarYTraducir(posicionCorazon(i), 90, 5, 0, 0));

  let cantidadPuntosCorazon = puntosOrigen.length;
  let puntosObjetivo = [];

  const pulso = (kx, ky) => {
    for (let i = 0; i < puntosOrigen.length; i++) {
      puntosObjetivo[i] = [
        kx * puntosOrigen[i][0] + ancho / 2,
        ky * puntosOrigen[i][1] + alto / 2
      ];
    }
  };

  let e = [];
  for (let i = 0; i < cantidadPuntosCorazon; i++) {
    let x = aleatorio() * ancho;
    let y = aleatorio() * alto;

    e[i] = {
      vx: 0,
      vy: 0,
      R: 2,
      velocidad: aleatorio() + 5,
      q: ~~(aleatorio() * cantidadPuntosCorazon),
      D: 2 * (i % 2) - 1,
      fuerza: 0.2 * aleatorio() + 0.7,
      f: "hsla(330," + ~~(40 * aleatorio() + 60) + "%," + ~~(60 * aleatorio() + 20) + "%,.3)",
      rastro: []
    };

    for (let k = 0; k < cantidadRastro; k++) {
      e[i].rastro[k] = { x, y };
    }
  }

  let tiempo = 0;

  function bucle() {
    let n = -Math.cos(tiempo);
    pulso((1 + n) * 0.5, (1 + n) * 0.5);

    tiempo += ((Math.sin(tiempo) < 0) ? 9 : (n > 0.8) ? 0.2 : 1) * 0.01;

    ctx.fillStyle = "rgba(0,0,0,0.1)";
    ctx.fillRect(0, 0, ancho, alto);

    for (let i = e.length; i--;) {
      let u = e[i];
      let q = puntosObjetivo[u.q];

      let dx = u.rastro[0].x - q[0];
      let dy = u.rastro[0].y - q[1];
      let longitud = Math.sqrt(dx * dx + dy * dy) || 1;

      if (longitud < 10) {
        if (aleatorio() > 0.95) {
          u.q = ~~(aleatorio() * cantidadPuntosCorazon);
        } else {
          if (aleatorio() > 0.99) u.D *= -1;
          u.q = (u.q + u.D + cantidadPuntosCorazon) % cantidadPuntosCorazon;
        }
      }

      u.vx += -dx / longitud * u.velocidad;
      u.vy += -dy / longitud * u.velocidad;

      u.rastro[0].x += u.vx;
      u.rastro[0].y += u.vy;

      u.vx *= u.fuerza;
      u.vy *= u.fuerza;

      for (let k = 0; k < u.rastro.length - 1;) {
        let T = u.rastro[k];
        let N = u.rastro[++k];
        N.x -= 0.4 * (N.x - T.x);
        N.y -= 0.4 * (N.y - T.y);
      }

      ctx.fillStyle = u.f;
      for (let k = 0; k < u.rastro.length; k++) {
        ctx.fillRect(u.rastro[k].x, u.rastro[k].y, 1, 1);
      }
    }

    ctx.fillStyle = "white";
    for (let i = 0; i < puntosObjetivo.length; i++) {
      ctx.fillRect(puntosObjetivo[i][0], puntosObjetivo[i][1], 2, 2);
    }

    requestAnimationFrame(bucle);
  }

  bucle();
};

if (document.readyState !== 'loading') iniciar();
else document.addEventListener('DOMContentLoaded', iniciar);
</script>
