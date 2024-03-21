<template>
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      <title>Drawing App</title>
      <link rel="stylesheet" href="../assets/styles/style.css" />
      <title>Drawing application</title>
    </head>
    <body>
      <div id="app">
        <h2>{{ message }}</h2>
        <h3 class="tool">Choose Pen Color</h3>
        <div class="color-picker">
          <div
            v-for="color in colors"
            :key="color"
            class="color-box"
            :style="{ backgroundColor: color }"
            @click="changeColor(color, 'brush')"
          ></div>
        </div>

        <div class="brush-size-picker">
          <div
            class="size-box"
            v-for="brush in brushes"
            :key="brush.id"
            @click="changeSize(brush)"
            :class="{ 'size-box_selected': brush === brush_selected }"
          >
            {{ brush }}
          </div>
        </div>

        <div class="brush-type-picker">
          <button @click="selectBrushType('normal')">Normal Brush</button>
          <button @click="selectBrushType('triple')">Triple Brush</button>
          <button @click="selectBrushType('spray')">Spray Brush</button>
        </div>

        <canvas
          @mousedown="startPainting"
          @mouseup="finishedPainting"
          @mouseleave="finishedPainting"
          @mousemove="draw"
          id="canvas"
        ></canvas>

        <h3 class="tool">Choose Background Color</h3>
        <div class="color-picker">
          <div
            v-for="color in colors"
            :key="color"
            class="color-box"
            :style="{ backgroundColor: color }"
            @click="changeColor(color, 'background')"
          ></div>
        </div>
        <div class="button-box">
          <button
            class="eraser"
            :class="{ active: eraserMode }"
            @click="toggleEraserMode"
          >
            {{ eraserMode ? "Disable Eraser" : "Enable Eraser" }}
          </button>
          <button class="clear-button" @click.prevent="clearCanvas">
            Clear Canvas
          </button>
          <button class="save-button" @click.prevent="saveButton">
            Save picture
          </button>
        </div>
      </div>
    </body>
  </html>
</template>

<script>
import { ref, onMounted } from "vue";
export default {
  setup() {
    const message = ref("Hello! You can draw here");
    const painting = ref(false);
    const canvas = ref(null);
    const ctx = ref(null);
    const brush_color = ref("#000000");
    const background_color = ref("#FFFFFF");
    const colors = ref([
      "#CCFF66",
      "#66FF00",
      "#66FFCC",
      "#99FFFF",
      "#0033FF",
      "#6666FF",
      "#CC66FF",
      "#FFCCFF",
      "#FF6699",
      "#FF0033",
      "#FF6633",
      "#FFFF00",
      "#663300",
      "#CCCCCC",
      "#666666",
      "#000000",
      "#FFFFFF",
    ]);
    const brushes = ref([1, 2, 3, 5, 8, 10, 15, 25, 35, 50, 75, 100, 125]);
    const brush_selected = ref(1);
    const selectedBrushType = ref("normal");
    const eraserMode = ref(false);

    const changeColor = (color, type) => {
      if (type === "brush") {
        brush_color.value = color; // Установка цвета кисти
        ctx.value.strokeStyle = color; // Установка цвета кисти для режима "normal"
        eraserMode.value = false;
      } else if (type === "background") {
        canvas.value.style.backgroundColor = color;
      }
    };

    const changeSize = (brush) => {
      ctx.value.lineWidth = brush;
      brush_selected.value = brush;
    };

    const selectBrushType = (type) => {
      selectedBrushType.value = type;
    };

    const clearCanvas = () => {
      ctx.value.clearRect(0, 0, canvas.value.width, canvas.value.height);
    };

    const saveButton = () => {
      const tempCanvas = document.createElement("canvas");
      const tempCtx = tempCanvas.getContext("2d");
      tempCanvas.width = canvas.value.width;
      tempCanvas.height = canvas.value.height;
      tempCtx.fillStyle = canvas.value.style.backgroundColor;
      tempCtx.fillRect(0, 0, tempCanvas.width, tempCanvas.height);
      tempCtx.drawImage(canvas.value, 0, 0);
      const data = tempCanvas.toDataURL("image/png");
      const a = document.createElement("a");
      a.href = data;
      a.download = "picture.png";
      a.click();
    };

    const startPainting = (e) => {
      painting.value = true;
      draw(e);
    };

    const finishedPainting = () => {
      painting.value = false;
      ctx.value.beginPath();
    };

    const draw = (e) => {
      if (!painting.value) return;

      ctx.value.lineCap = "round";

      const rect = canvas.value.getBoundingClientRect();
      const offsetX = rect.left;
      const offsetY = rect.top;

      const x = e.clientX - offsetX;
      const y = e.clientY - offsetY;

      switch (selectedBrushType.value) {
        case "triple":
          ctx.value.fillStyle = brush_color.value; // Установка цвета кисти для режима "triple"
          // Рисование с тройной кистью
          const mainPointRadius = brush_selected.value; // Радиус основной точки
          const sidePointRadius = brush_selected.value / 2; // Радиус боковых точек
          if (eraserMode.value) {
            ctx.value.fillStyle = canvas.value.style.backgroundColor;
          }
          // Основная точка
          ctx.value.beginPath();
          ctx.value.arc(x, y, mainPointRadius, 0, 2 * Math.PI);
          ctx.value.fill();
          ctx.value.closePath();

          // Боковые точки
          const sideDistance = brush_selected.value * 20; // Расстояние до боковых точек
          ctx.value.beginPath();
          ctx.value.arc(x - sideDistance, y, sidePointRadius, 0, 2 * Math.PI);
          ctx.value.fill();
          ctx.value.closePath();

          ctx.value.beginPath();
          ctx.value.arc(x + sideDistance, y, sidePointRadius, 0, 2 * Math.PI);
          ctx.value.fill();
          ctx.value.closePath();
          break;
        case "spray":
          ctx.value.fillStyle = brush_color.value;
          ctx.value.lineWidth = brush_selected.value;
          if (eraserMode.value) {
            ctx.value.fillStyle = canvas.value.style.backgroundColor;
          }
          for (let i = 0; i < 50; i++) {
            const randomX =
              x +
              Math.random() * brush_selected.value * 10 -
              brush_selected.value;
            const randomY =
              y +
              Math.random() * brush_selected.value * 10 -
              brush_selected.value;
            ctx.value.fillRect(
              randomX,
              randomY,
              brush_selected.value / 5,
              brush_selected.value / 5
            );
          }
          break;
        default:
          ctx.value.strokeStyle = brush_color.value;
          ctx.value.lineWidth = brush_selected.value;
          if (eraserMode.value) {
            ctx.value.strokeStyle = canvas.value.style.backgroundColor;
          }
          ctx.value.lineTo(x, y);
          ctx.value.stroke();
          break;
      }

      ctx.value.beginPath();
      ctx.value.moveTo(x, y);
    };

    const toggleEraserMode = () => {
      eraserMode.value = !eraserMode.value;
    };

    onMounted(() => {
      canvas.value = document.getElementById("canvas");
      ctx.value = canvas.value.getContext("2d");

      ctx.value.strokeStyle = colors.value[0];

      canvas.value.style.backgroundColor = "#FFFFFF";

      canvas.value.height = window.innerHeight * 0.6;
      canvas.value.width = window.innerWidth * 0.7;
    });

    return {
      message,
      painting,
      canvas,
      ctx,
      colors,
      brushes,
      brush_selected,
      brush_color,
      background_color,
      eraserMode,
      changeColor,
      toggleEraserMode,
      changeSize,
      clearCanvas,
      saveButton,
      startPainting,
      finishedPainting,
      draw,
      selectedBrushType,
      selectBrushType,
    };
  },
};
</script>
