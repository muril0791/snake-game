<template>
  <div class="game-container">
    <div v-if="!gameStarted" class="start-screen">
      <button @click="startGame">Iniciar Jogo</button>
    </div>
    <div v-else>
      <div ref="gameContainer" class="game-area"></div>
      <div class="score-board">Score: {{ score }}</div>
      <div v-if="gameOver" class="game-over">
        <p>Game Over!</p>
        <button @click="startGame">Jogar Novamente</button>
      </div>
    </div>
  </div>
</template>

<script>
import * as PIXI from "pixi.js";
import { onMounted, ref, onBeforeUnmount, nextTick } from "vue";

export default {
  name: "SnakeGame",
  setup() {
    const gameContainer = ref(null);
    let app, snake, food, direction, speed, snakeSize, background, obstacles;
    const gridSize = 20;
    const initialLength = 3;
    const directions = {
      ArrowUp: { x: 0, y: -1 },
      ArrowDown: { x: 0, y: 1 },
      ArrowLeft: { x: -1, y: 0 },
      ArrowRight: { x: 1, y: 0 },
      w: { x: 0, y: -1 },
      s: { x: 0, y: 1 },
      a: { x: -1, y: 0 },
      d: { x: 1, y: 0 },
    };
    const score = ref(0);
    const gameStarted = ref(false);
    const gameOver = ref(false);

    const startGame = async () => {
      gameStarted.value = true;
      gameOver.value = false;
      score.value = 0;
      await nextTick(); // Aguarda a atualização do DOM
      setupGame();
    };

    const setupGame = () => {
      try {
        if (app) {
          app.destroy(true, {
            children: true,
            texture: true,
            baseTexture: true,
          });
        }
        app = new PIXI.Application({ width: 800, height: 600 });
        gameContainer.value.appendChild(app.view);

        // Cria a cobra
        snake = [];
        direction = directions.ArrowRight;
        speed = 200;
        snakeSize = 20;

        // Define a posição inicial no centro
        const startX = Math.floor(app.view.width / 2 / snakeSize) * snakeSize;
        const startY = Math.floor(app.view.height / 2 / snakeSize) * snakeSize;

        for (let i = 0; i < initialLength; i++) {
          const part = new PIXI.Graphics();
          part.beginFill(0x00ff00);
          part.drawRect(0, 0, snakeSize, snakeSize);
          part.endFill();
          part.x = startX - i * snakeSize;
          part.y = startY;
          snake.push(part);
          app.stage.addChild(part);
        }

        // Cria a comida
        food = PIXI.Sprite.from("/rat.png");
        food.width = snakeSize * 1.5;
        food.height = snakeSize * 1.5;
        placeFood();
        app.stage.addChild(food);

        // Cria obstáculos
        obstacles = [];
        for (let i = 0; i < 5; i++) {
          obstacles.push(addObstacle());
        }

        window.addEventListener("keydown", changeDirection);
        gameLoop();
      } catch (error) {
        console.error("Erro ao configurar o jogo:", error);
      }
    };

    const placeFood = () => {
      food.x =
        Math.floor(Math.random() * (app.view.width / gridSize)) * gridSize;
      food.y =
        Math.floor(Math.random() * (app.view.height / gridSize)) * gridSize;
    };

    const changeDirection = (event) => {
      const key = event.key.toLowerCase();
      if (directions[key]) {
        direction = directions[key];
      }
    };

    const gameLoop = () => {
      setTimeout(() => {
        moveSnake();
        if (checkCollision()) {
          gameOver.value = true;
          return;
        } else {
          gameLoop();
        }
      }, speed);
    };

    const moveSnake = () => {
      const newHead = new PIXI.Graphics();
      newHead.beginFill(0x00ff00);
      newHead.drawRect(0, 0, snakeSize, snakeSize);
      newHead.endFill();
      newHead.x =
        (snake[0].x + direction.x * snakeSize + app.view.width) %
        app.view.width;
      newHead.y =
        (snake[0].y + direction.y * snakeSize + app.view.height) %
        app.view.height;

      if (newHead.x === food.x && newHead.y === food.y) {
        score.value += 10;
        placeFood();
      } else {
        const tail = snake.pop();
        app.stage.removeChild(tail);
      }

      snake.unshift(newHead);
      app.stage.addChild(newHead);
    };

    const checkCollision = () => {
      const head = snake[0];
      for (let i = 1; i < snake.length; i++) {
        if (head.x === snake[i].x && head.y === snake[i].y) {
          return true;
        }
      }
      for (const obstacle of obstacles) {
        if (head.x === obstacle.x && head.y === obstacle.y) {
          return true;
        }
      }
      return false;
    };

    const addObstacle = () => {
      const obstacle = PIXI.Sprite.from("/grass.png");
        obstacle.width = snakeSize * 2;
        obstacle.height = snakeSize * 2;
      obstacle.x =
        Math.floor(Math.random() * (app.view.width / gridSize)) * gridSize;
      obstacle.y =
        Math.floor(Math.random() * (app.view.height / gridSize)) * gridSize;
      app.stage.addChild(obstacle);
      return obstacle;
    };

    onBeforeUnmount(() => {
      if (app) {
        app.destroy(true, { children: true, texture: true, baseTexture: true });
      }
      window.removeEventListener("keydown", changeDirection);
    });

    return {
      gameContainer,
      score,
      gameStarted,
      gameOver,
      startGame,
    };
  },
};
</script>

<style>
.game-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  height: 100vh;
  overflow: hidden;
}

.start-screen {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.game-area {
  position: relative;
}

.score-board {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 20px;
  color: white;
}

.game-over {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  color: white;
  font-size: 24px;
  background-color: rgba(0, 0, 0, 0.7);
  padding: 20px;
  border-radius: 10px;
}
</style>
