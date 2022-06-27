<template>
  <div>
    <canvas id="canvas1" width="800" height="500"></canvas>
    <GiftModal :couponR="couponR" :couponRN="couponRN" />
    <ListCoupon :couponR="couponR" :couponArray="couponArray" />
  </div>
</template>

<script lang="ts">
import { onMounted, Ref, ref } from "vue";
import GiftModal from "./GifModal.vue";
import ListCoupon from "./ListCoupon.vue";
export default {
  name: "Home",
  components: { GiftModal, ListCoupon },

  setup() {
    onMounted(() => {
      draw();
    });

    const couponR: Ref<string | undefined> = ref("");
    const couponRN: Ref<number | undefined> = ref();
    const couponArray = ref([]);

    // Canvas setup
    const draw = () => {
      const canvas: any = document.getElementById("canvas1");
      // @ts-ignore
      const ctx = canvas.getContext("2d");

      canvas.width = windowWH().w - 100;
      canvas.height = windowWH().h - 100;

      let gameFrame = 0;
      let is_dragging: boolean;
      let current_fish_index: any = null;
      let startX: number;
      let startY: number;
      let numberOfPlays: number = 3;

      ctx.font = "50px Georgia";
      // @ts-ignore
      let canvasPosition = canvas.getBoundingClientRect();

      // Xac dinh vi tri chuot
      let is_mouse_in_shape = function (x: number, y: number, enemy: any) {
        let shape_left = enemy.x - enemy.radius * 1.75;
        let shape_right = shape_left + enemy.radius * 2.5;
        let shape_top = enemy.y - enemy.radius * 1.75;
        let shape_bottom = enemy.y + enemy.radius * 1.75;

        if (
          x > shape_left &&
          x < shape_right &&
          y > shape_top &&
          y < shape_bottom
        ) {
          return true;
        } else {
          return false;
        }
      };

      // @ts-ignore
      let mouse_down = function (event) {
        event.preventDefault();
        // @ts-ignore
        startX = parseInt(event.clientX - canvasPosition.left);
        // @ts-ignore
        startY = parseInt(event.clientY - canvasPosition.top);

        let index = 0;

        for (let enemy of enemiesArray) {
          if (is_mouse_in_shape(startX, startY, enemy)) {
            current_fish_index = index;
            is_dragging = true;
            return;
          } else {
          }
          index++;
        }
      };

      // @ts-ignore
      let mouse_up = function (event) {
        if (!is_dragging) {
          return;
        }
        event.preventDefault();
        is_dragging = false;
      };

      // @ts-ignore
      let mouse_out = function (event) {
        if (!is_dragging) {
          return;
        }
        event.preventDefault();
        is_dragging = false;
      };

      // @ts-ignore
      let mouse_move = async (event) => {
        if (!is_dragging) {
          return;
        } else {
          event.preventDefault();
          // @ts-ignore
          let mouseX = parseInt(event.clientX - canvasPosition.left);
          // @ts-ignore
          let mouseY = parseInt(event.clientY - canvasPosition.top);

          let dx = mouseX - startX;
          let dy = mouseY - startY;

          let current_fish = enemiesArray[current_fish_index];

          current_fish.x += dx;
          current_fish.y += dy;

          startX = mouseX;
          startY = mouseY;
        }
      };

      canvas.onmousedown = mouse_down;
      canvas.onmouseup = mouse_up;
      canvas.onmouseout = mouse_out;
      canvas.onmousemove = mouse_move;

      // Chest
      const chestImage = new Image();
      chestImage.src = "/chest/RED-OPEN.png";

      class Chest {
        x: number;
        y: number;
        distance: number | undefined;
        radius: number;

        constructor() {
          this.x = canvas.width * 0.5;
          this.y = canvas.height * 0.65;
          this.distance;
          this.radius = 30;
        }
        draw(): void {
          ctx.fillStyle = "transparent";
          ctx.beginPath();
          ctx.arc(
            canvas.width * 0.52,
            canvas.height * 0.9,
            this.radius,
            0,
            Math.PI * 2
          );
          ctx.fill();
          ctx.closePath();
          ctx.drawImage(
            chestImage,
            canvas.width * 0.45,

            this.y,
            canvas.width * 0.15,
            canvas.width * 0.15
          );
        }
      }

      const chest = new Chest();

      // Bubbles

      interface BubbleType {
        x: number;
        y: number;
        radius: number;
        speed: number;
        distance: number | undefined;
        counted: boolean;
        update: () => void;
        draw: () => void;
      }

      const bubblesArray: BubbleType[] = [];

      const bubbleImage = new Image();
      bubbleImage.src = "/bubble-pop/bubble_pop_frame_01.png";
      class Bubble {
        x: number;
        y: number;
        radius: number;
        speed: number;
        distance: number | undefined;
        counted: boolean;

        constructor() {
          this.x = Math.random() * canvas.width;
          this.y = canvas.height + 100;
          this.radius = 50;
          this.speed = Math.random() * 5 + 1;
          this.distance;
          this.counted = false;
        }
        update() {
          this.y -= this.speed;
        }
        draw() {
          ctx.fillStyle = "transparent";
          ctx.beginPath();

          ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
          ctx.fill();
          ctx.closePath();
          ctx.stroke();
          ctx.drawImage(
            bubbleImage,
            this.x - 65,
            this.y - 65,
            this.radius * 2.6,
            this.radius * 2.6
          );
        }
      }

      function handleBubbles() {
        if (gameFrame % 50 == 0) {
          bubblesArray.push(new Bubble());
        }
        for (let i = 0; i < bubblesArray.length; i++) {
          bubblesArray[i].update();
          bubblesArray[i].draw();
        }
      }

      // Enemies
      const enemyImage = new Image();
      enemyImage.src = "/enemy/__red_cartoon_fish_01_swim.png";

      function randomInteger(min: number, max: number) {
        return Math.floor(Math.random() * (max - min + 1)) + min;
      }

      interface EnemyType {
        x: number;
        y: number;
        radius: number;
        speed: number;
        frame: number;
        frameX: number;
        frameY: number;
        spriteWidth: number;
        spriteHeight: number;
        distance: number | undefined;
        draw: () => void;
        update: () => void;
      }

      class Enemy {
        x: number;
        y: number;
        radius: number;
        speed: number;
        frame: number;
        frameX: number;
        frameY: number;
        spriteWidth: number;
        spriteHeight: number;
        distance: number | undefined;

        constructor() {
          this.x = canvas.width + 200;
          this.y = randomInteger(45, canvas.height * 0.45);
          this.radius = 60;
          this.speed = Math.random() * 2 + 2;
          this.frame = 0;
          this.frameX = 0;
          this.frameY = 0;
          this.spriteWidth = 418;
          this.spriteHeight = 397;
          this.distance;
        }
        draw() {
          ctx.fillStyle = "transparent";
          ctx.beginPath();
          ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
          ctx.fill();
          ctx.drawImage(
            enemyImage,
            this.frameX * this.spriteWidth,
            this.frameY * this.spriteHeight,
            this.spriteWidth,
            this.spriteHeight,
            this.x - 60,
            this.y - 70,
            this.spriteWidth / 3,
            this.spriteWidth / 3
          );
        }

        update() {
          this.x -= this.speed;
          if (this.x < 0 - this.radius * 2) {
            this.x = canvas.width + 200;
            this.y = randomInteger(90, 220);
            this.speed = Math.random() * 2 + 2;
          }
          if (gameFrame % 5 == 0) {
            this.frame++;

            if (this.frame >= 12) this.frame = 0;

            if (this.frame == 3 || this.frame == 7 || this.frame == 11) {
              this.frameX = 0;
            } else {
              this.frameX++;
            }

            if (this.frame < 3) this.frameY = 0;
            else if (this.frame < 7) this.frameY = 1;
            else if (this.frame < 11) this.frame = 2;
            else this.frameY = 0;
          }

          const dx = this.x - chest.x;
          const dy = this.y - chest.y;

          this.distance = Math.sqrt(dx * dx + dy * dy);
        }
      }

      function handleCatchFish() {
        if (gameFrame % 60 == 0 && gameFrame < 250) {
          enemiesArray.push(new Enemy());
        }
        for (let i = 0; i < enemiesArray.length; i++) {
          enemiesArray[i].update();

          enemiesArray[i].draw();

          if (
            // @ts-ignore
            enemiesArray[i].distance <
            enemiesArray[i].radius + chest.radius
          ) {
            enemiesArray.splice(i, 1);
            is_dragging = false;
            handleGameOver();
          }
        }
      }

      let enemiesArray: EnemyType[] = [];

      function handleGameOver() {
        ctx.fillStyle = "black";
        OpenBootstrapPopup();
        numberOfPlays--;
      }

      function OpenBootstrapPopup() {
        couponR.value = (Math.random() + 1).toString(36).substring(7);
        couponRN.value = getRandomArbitrary(10, 50);
        // @ts-ignore
        couponArray.value.push(couponR.value);
        // @ts-ignore
        $("#myModal").modal("show");
      }

      // Animation Loop
      function animate() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        handleBubbles();
        chest.draw();
        handleCatchFish();
        gameFrame++;
        if (numberOfPlays != 0) {
          requestAnimationFrame(animate);
        }
      }
      animate();

      function getRandomArbitrary(min: number, max: number) {
        min = Math.ceil(min);
        max = Math.floor(max);
        return Math.floor(Math.random() * (max - min + 1)) + min;
      }

      function windowWH() {
        return {
          w: window.innerWidth,
          h: window.innerHeight,
        };
      }
    };

    return {
      couponR,
      couponRN,
      couponArray,
    };
  },
};
</script>

<style>
#canvas1 {
  padding: 0px;
  margin: auto 0px;
  border: 4px solid black;
  background: linear-gradient(to bottom, lightblue, rgb(0, 14, 139));
}
</style>
