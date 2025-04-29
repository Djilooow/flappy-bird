# flappy-bird<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Flappy Bird</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: sans-serif; display: flex; flex-direction: column; align-items: center; background: #70c5ce; }
    canvas { background: #4ec0ca; border: 2px solid #000; margin-top: 20px; }
    h1 { margin-top: 20px; color: #fff; }
  </style>
</head>
<body>
  <h1>Flappy Bird</h1>
  <canvas id="gameCanvas" width="320" height="480"></canvas>

  <audio id="flapSound" src="https://www.soundjay.com/button/beep-07.wav"></audio>
  <audio id="hitSound" src="https://www.soundjay.com/button/beep-10.wav"></audio>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    const flapSound = document.getElementById('flapSound');
    const hitSound = document.getElementById('hitSound');

    let frames = 0;
    const DEGREE = Math.PI / 180;

    const state = {
      current: 0,
      getReady: 0,
      game: 1,
      over: 2
    };

    let score = 0;
    let best = localStorage.getItem("best") || 0;

    document.addEventListener('keydown', function (e) {
      if (e.code === 'Space') {
        switch (state.current) {
          case state.getReady:
            state.current = state.game;
            break;
          case state.game:
            bird.flap();
            flapSound.play();
            break;
          case state.over:
            pipes.reset();
            bird.reset();
            score = 0;
            state.current = state.getReady;
            break;
        }
      }
    });

    const bird = {
      x: 50,
      y: 150,
      w: 34,
      h: 26,
      gravity: 0.25,
      jump: 4.6,
      speed: 0,
      rotation: 0,
      draw() {
        ctx.save();
        ctx.translate(this.x, this.y);
        ctx.rotate(this.rotation);
        ctx.fillStyle = 'yellow';
        ctx.fillRect(-this.w / 2, -this.h / 2, this.w, this.h);
        ctx.restore();
      },
      flap() {
        this.speed = -this.jump;
      },
      update() {
        if (state.current === state.getReady) {
          this.y = 150;
          this.rotation = 0;
        } else {
          this.speed += this.gravity;
          this.y += this.speed;
          if (this.speed >= this.jump) {
            this.rotation = 90 * DEGREE;
          } else {
            this.rotation = -25 * DEGREE;
          }
        }

        if (this.y + this.h / 2 >= canvas.height) {
          state.current = state.over;
          hitSound.play();
        }
      },
      reset() {
        this.speed = 0;
        this.y = 150;
      }
    };

    const pipes = {
      position: [],
      w: 50,
      h: 300,
      gap: 100,
      dx: 2,
      draw() {
        this.position.forEach(p => {
          ctx.fillStyle = 'green';
          ctx.fillRect(p.x, p.y, this.w, this.h);
          ctx.fillRect(p.x, p.y + this.h + this.gap, this.w, this.h);
        });
      },
      update() {
        if (state.current !== state.game) return;

        if (frames % 100 === 0) {
          this.position.push({
            x: canvas.width,
            y: -Math.floor(Math.random() * this.h)
          });
        }

        this.position.forEach((p, i) => {
          p.x -= this.dx;

          if (p.x + this.w <= 0) {
            this.position.splice(i, 1);
            score++;
            best = Math.max(best, score);
            localStorage.setItem("best", best);
          }

          if (bird.x + bird.w / 2 > p.x && bird.x - bird.w / 2 < p.x + this.w &&
              (bird.y - bird.h / 2 < p.y + this.h || bird.y + bird.h / 2 > p.y + this.h + this.gap)) {
            state.current = state.over;
            hitSound.play();
          }
        });
      },
      reset() {
        this.position = [];
      }
    };

    function draw() {
      ctx.fillStyle = '#70c5ce';
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      pipes.draw();
      bird.draw();

      ctx.fillStyle = '#fff';
      ctx.font = '20px Arial';
      if (state.current === state.game) {
        ctx.fillText("Score : " + score, 10, 25);
      } else if (state.current === state.getReady) {
        ctx.fillText("Appuie sur ESPACE pour commencer", 20, canvas.height / 2);
      } else if (state.current === state.over) {
        ctx.fillText("Game Over ! Score: " + score, 50, canvas.height / 2 - 20);
        ctx.fillText("Meilleur score: " + best, 70, canvas.height / 2 + 10);
        ctx.fillText("Appuie sur ESPACE pour recommencer", 10, canvas.height / 2 + 40);
      }
    }

    function update() {
      bird.update();
      pipes.update();
    }

    function loop() {
      update();
      draw();
      frames++;
      requestAnimationFrame(loop);
    }

    loop();
  </script>
</body>
</html>
