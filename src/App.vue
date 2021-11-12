<template>
  <div class="main">
    <div class="canvas-container">
      <canvas class="canvas" height="600" width="800"></canvas>
    </div>
    <div class="ctrl-container">
      <h1>Cohen-Sutherland algorithm</h1>
      <h3>Frame params</h3>
      <div class="input-wrapper">
        <div class="label">x</div>
        <input type="number" id="x" min="0" max="600" value="100" />
      </div>
      <div class="input-wrapper">
        <div class="label">y</div>
        <input type="number" id="y" min="0" max="600" value="100" />
      </div>
      <div class="input-wrapper">
        <div class="label">width</div>
        <input type="number" id="width" min="0" max="600" value="600" />
      </div>
      <div class="input-wrapper">
        <div class="label">height</div>
        <input type="number" id="height" min="0" max="600" value="400" />
      </div>
      <h3>Number of lines</h3>
      <div class="input-wrapper">
        <div class="label">count</div>
        <select id="select">
          <option value="1">1</option>
          <option value="2">2</option>
          <option value="3">3</option>
          <option value="4">4</option>
          <option selected value="5">5</option>
          <option value="6">6</option>
          <option value="7">7</option>
          <option value="8">8</option>
          <option value="9">9</option>
          <option value="10">10</option>
          <option value="15">15</option>
        </select>
      </div>
      <button class="draw-btn" @click="redraw" :disabled="btnLock">draw</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  components: {},
  data() {
    return {
      left: 1,
      right: 2,
      bot: 4,
      top: 8,

      invalid_input: false,
      timeout1: null,
      timeout2: null,
      btnLock: false,
    };
  },
  mounted() {
    const canvas = document.querySelector(".canvas");
    const ctx = canvas.getContext("2d");
    ctx.translate(0, canvas.height);
    ctx.scale(1, -1);
  },
  methods: {
    redraw: async function () {
      this.btnLock = true;
      if (this.timeout1 != null) {
        clearTimeout(this.timeout1);
      }
      if (this.timeout2 != null) {
        clearTimeout(this.timeout2);
      }
      const canvas = document.querySelector(".canvas");
      const ctx = canvas.getContext("2d");
      const count = document.getElementById("select").value;
      let rect = this.getRect();

      let lines = new Array();
      let completeLines = new Array();

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.imageSmoothingEnabled = true;
      ctx.translate(0.5, 0.5);

      ctx.beginPath();
      ctx.strokeStyle = "#ccc";
      ctx.lineWidth = 1;

      for (var x = 0.5; x < 800; x += 20) {
        ctx.moveTo(x, 0);
        ctx.lineTo(x, 600);
      }

      for (var y = 0.5; y < 600; y += 20) {
        ctx.moveTo(0, y);
        ctx.lineTo(800, y);
      }

      ctx.stroke();

      ctx.closePath();

      ctx.strokeStyle = "#7a7a7a";
      ctx.lineWidth = 2;

      for (let i = 0; i < count; i++) {
        let arr = this.getRandomCoords();
        lines.push({ x: arr[0], y: arr[1] });
        lines.push({ x: arr[2], y: arr[3] });

        let a = { x: arr[0], y: arr[1] };
        let b = { x: arr[2], y: arr[3] };

        completeLines.push({ a, b });

        let points = this.calcWaypoints(lines);
        let t = 1;

        console.log(points, t);
        ctx.beginPath();
        await this.animate(ctx, t, points);
        ctx.closePath();

        lines = [];
      }

      setTimeout(() => {
        ctx.strokeStyle = "#000";
        lines.push({ x: rect.x_min, y: rect.y_min });
        lines.push({ x: rect.x_max, y: rect.y_min });
        lines.push({ x: rect.x_max, y: rect.y_max });
        lines.push({ x: rect.x_min, y: rect.y_max });
        lines.push({ x: rect.x_min, y: rect.y_min });

        let points = this.calcWaypoints(lines);
        let t = 1;

        ctx.beginPath();
        this.animate(ctx, t, points);
        ctx.closePath();

        lines = [];
      }, 2000);

      new Promise((resolve) => {
        setTimeout(() => {
          for (let i = 0; i < count; i++) {
            console.log(completeLines);
            let res = this.cohen_sutherland(
              rect,
              completeLines[i].a,
              completeLines[i].b
            );
            if (res === -1) {
              continue;
            }
            (completeLines[i].a = res.a),
              (completeLines[i].b = res.b),
              lines.push(completeLines[i].a);
            lines.push(completeLines[i].b);

            let points = this.calcWaypoints(lines);
            let t = 1;

            ctx.beginPath();
            ctx.strokeStyle = "red";
            this.animate(ctx, t, points);
            ctx.closePath();

            lines = [];
          }
          completeLines = [];
          resolve();
        }, 4200);
      }).then(() => {
        setTimeout(() => {
          this.btnLock = false;
        }, 1000);
      });
    },
    calcWaypoints(vertices) {
      let waypoints = [];
      for (let i = 1; i < vertices.length; i++) {
        let pt0 = vertices[i - 1];
        let pt1 = vertices[i];
        let dx = pt1.x - pt0.x;
        let dy = pt1.y - pt0.y;
        for (let j = 0; j <= 100; j++) {
          let x = pt0.x + (dx * j) / 100;
          let y = pt0.y + (dy * j) / 100;
          waypoints.push({ x: x, y: y });
        }
      }
      return waypoints;
    },
    animate(ctx, t, points) {
      let interval = setInterval(function () {
        if (t >= points.length) clearInterval(interval);
        else {
          ctx.moveTo(points[t - 1].x, points[t - 1].y);
          ctx.lineTo(points[t].x, points[t].y);
          ctx.stroke();
          t++;
        }
      }, 5);
    },
    getRandomCoords() {
      let numReserve = [];
      let counter = 1;
      while (numReserve.length < 4) {
        let randomNumber;
        if (counter % 2) randomNumber = Math.ceil(Math.random() * 800);
        else randomNumber = Math.ceil(Math.random() * 600);

        let found = false;
        for (let i = 0; i < numReserve.length; i++) {
          if (numReserve[i] === randomNumber) {
            found = true;
            break;
          }
        }
        if (!found) {
          numReserve[numReserve.length] = randomNumber;
        }
        counter++;
      }
      return numReserve;
    },
    getRect() {
      let x = document.getElementById("x").value;
      let y = document.getElementById("y").value;
      let width = document.getElementById("width").value;
      let height = document.getElementById("height").value;
      x = Number(x);
      y = Number(y);
      width = Number(width);
      height = Number(height);
      return { x_min: x, x_max: x + width, y_min: y, y_max: y + height };
    },
    getCode(r, p) {
      return (
        (p.x < r.x_min ? this.left : 0) +
        (p.x > r.x_max ? this.right : 0) +
        (p.y < r.y_min ? this.bot : 0) +
        (p.y > r.y_max ? this.top : 0)
      );
    },
    cohen_sutherland(r, a, b) {
      let code_a = this.getCode(r, a);
      let code_b = this.getCode(r, b);
      let code, c;

      while (code_a | code_b) {
        if (code_a & code_b) return -1;

        if (code_a) {
          code = code_a;
          c = a;
        } else {
          code = code_b;
          c = b;
        }

        if (code & this.left) {
          c.y += ((a.y - b.y) * (r.x_min - c.x)) / (a.x - b.x);
          c.x = r.x_min;
        } else if (code & this.right) {
          c.y += ((a.y - b.y) * (r.x_max - c.x)) / (a.x - b.x);
          c.x = r.x_max;
        } else if (code & this.bot) {
          c.x += ((a.x - b.x) * (r.y_min - c.y)) / (a.y - b.y);
          c.y = r.y_min;
        } else if (code & this.top) {
          c.x += ((a.x - b.x) * (r.y_max - c.y)) / (a.y - b.y);
          c.y = r.y_max;
        }

        if (code == code_a) {
          a = c;
          code_a = this.getCode(r, a);
        } else {
          b = c;
          code_b = this.getCode(r, b);
        }
      }

      return { a, b };
    },
  },
};
</script>

<style lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Titillium+Web&display=swap");

* {
  font-family: "Titillium Web", sans-serif;
  color: rgb(54, 54, 54);
}

html {
  margin: 0;
  padding: 0;
  height: 100%;
}
body {
  background: url("./assets/mesh2.jpg");
  background-size: cover;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  max-height: 100%;
  height: 100%;
  width: 100%;
}

.main {
  display: flex;
  width: 100%;
  height: 100vh;
  align-items: center;
  .canvas-container {
    width: 60%;
    display: flex;
    justify-content: center;

    .canvas {
      border-radius: 15px;
      box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.397);
      background: rgb(245, 245, 245);
    }
  }
  .ctrl-container {
    h1 {
      width: 300px;
      text-align: center;
      margin: 0;
    }
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-evenly;
    height: 600px;
    width: 400px;

    h3 {
      margin-bottom: -5px;
      margin-top: -5px;
    }

    .input-wrapper {
      font-size: 1.2em;
      height: 30px;
      display: grid;
      grid-template-columns: 100px 100px;
      margin: -15px 0 0 0;
      input {
        border-radius: 7px;
        padding: 5px;
        box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.247);
        border: none;
        &:focus {
          outline: none;
        }
      }
      select {
        border-radius: 7px;
        padding: 5px;
        outline: none;
        box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.247);
        border: none;
        &:focus {
          outline: none;
        }
      }
    }
    .draw-btn {
      margin-top: 10px;
      width: 200px;
      height: 40px;
      outline: none;
      border: none;
      border-radius: 10px;
      box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.247);
      background: rgb(252, 252, 252);
      cursor: pointer;
      transition: all 0.3s ease-in-out;
      font-size: 1.2em;

      &:hover {
        background: rgb(230, 230, 230);
      }
    }
  }
}
</style>
