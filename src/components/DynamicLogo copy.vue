<script setup lang="ts">
import { ref, reactive, onMounted } from "vue";

// 准备logo数据
const logos = reactive([
  { label: "kazimierz", url: "/src/assets/logo_kazimierz.png" },
  { label: "rhine", url: "/src/assets/logo_rhine.png" },
  { label: "rhodes", url: "/src/assets/logo_rhodes.png" },
  { label: "victoria", url: "/src/assets/logo_victoria.png" },
  { label: "yan", url: "/src/assets/logo_yan.png" },
]);
// 标记激活logo
let activeLogo = ref<{ label: string; url: string }>();

// 获取canvas画布
const canvasDOM = ref<HTMLCanvasElement | null>(null);
// 获取上下文
let context = ref<CanvasRenderingContext2D | null>(null);
// 设置画布大小
const width = 600,
  height = 600;
let time1: any;

class ball {
  x: number; // 粒子x轴的初始位置
  y: number; // 粒子y轴的初始位置
  dx: number; // 粒子x轴的目标位置
  dy: number; // 粒子y轴的目标位置
  mx: number; // 粒子x轴需要移动的距离
  my: number; // 粒子y轴需要移动的距离
  time: number; // 粒子移动耗时
  r: number; // 粒子的半径
  color: string; // 粒子的颜色
  constructor(dx: number, dy: number, time: number = 80) {
    // 设置粒子的初始位置x、y，目标位置dx、dy，总耗时time
    this.x = Math.random() * width;
    this.y = Math.random() * height;
    this.dx = dx;
    this.dy = dy;
    this.time = time;
    // 设置粒子需要移动的距离
    this.mx = this.dx - this.x;
    this.my = this.dy - this.y;
    // 设置粒子的颜色和半径
    this.r = 1.5;
    this.color = Math.random() > 0.99 ? "grey" : "#fff";
  }
  // 在画布中绘制粒子
  draw() {
    this.x += this.mx / 80;
    this.y += this.my / 80;
    drawCircle(this.x, this.y, this.r, this.color);
  }
}

function drawCircle(x: number, y: number, r: number, color: string) {
  context.value!.beginPath();
  context.value!.fillStyle = color;
  // context.value!.arc(x, y, r, (Math.PI / 180) * 0, (Math.PI / 180) * 360);
  context.value!.fillRect(x, y, r, r);
  context.value!.fill();
}

// 激活logo
function clickLogo(item: any) {
  let image = new Image();
  image.src = item.url;
  image.onload = () => {
    getImgData(image);
    animateBall(image);
  };

  activeLogo.value = item;
}
// 开始动画
function start(a: any, b: any, t: number) {
  setTimeout(() => {
    relateTow(a, b);
    animateBall(a);
  }, 4000 * t);
}
//连接两个数组
function relateTow(a: any, b: any) {
  for (let i = 0; i < a.ballArr.length; i++) {
    const v = Math.random() * b.ballArr.length;
    a.ballArr[i].x = b.ballArr[v].dx;
    a.ballArr[i].initialX = a.ballArr[i].dx - a.ballArr[i].x;
    a.ballArr[i].y = b.ballArr[v].dy;
    a.ballArr[i].initialY = a.ballArr[i].dy - a.ballArr[i].y;
  }
}
// 设置动画
function animateBall(img: any) {
  let count = 1;
  // window.requestAnimationFrame()
  clearInterval(time1);
  time1 = setInterval(function () {
    context.value!.clearRect(0, 0, width, height);
    for (let i = 0; i < img.ballArr.length; i++) {
      img.ballArr[i].draw(count);
    }
    if (count === 80) {
      clearInterval(time1);
    }
    count++;
  }, 16);
}
// canvas 获取粒子位置数据
function getImgData(img: any) {
  context.value?.clearRect(0, 0, width, height); // 清空画布
  context.value?.drawImage(img, 0, 0, width, height); // 将图片绘制到canvas中
  const imgData = context.value?.getImageData(0, 0, width, height); // 获取像素点数据

  img.ballArr = []; // 用于保存像素点
  context.value!.clearRect(0, 0, width, height);
  // 筛选像素点
  for (let y = 0; y < height; y += 6) {
    for (let x = 0; x < width; x += 5) {
      // 像素点的序号
      const index = (x + y * width) * 4;
      // 帅选条件为透明度
      const a = imgData!.data[index + 3];

      if (a > 150) {
        // 符合条件条件的坐标绘制粒子
        const bal = new ball(x, y);
        img.ballArr.push(bal);
      }
    }
  }
}
onMounted(() => {
  context.value = canvasDOM.value!.getContext("2d") as CanvasRenderingContext2D;
});
</script>

<template>
  <div class="canvas-container">
    <canvas ref="canvasDOM" width="600" height="600"></canvas>
  </div>
  <ul class="logo-options-container">
    <li
      v-for="logoItem in logos"
      :class="['logo-item', activeLogo === logoItem && 'active']"
      @click="clickLogo(logoItem)"
    >
      <img :src="logoItem.url" class="item-picture" />
      <div class="item-desc">
        {{ logoItem.label }}
      </div>
    </li>
  </ul>
</template>

<style lang="scss" scoped>
.canvas-container {
  /* margin: auto; */
  width: 600px;
  height: 600px;
  border-radius: 8px;
  background-color: rgba(0, 0, 0, 0.4);
}
.logo-options-container {
  padding: 10px;
  margin: 0;
  margin-top: 5vw;
  display: flex;
  justify-content: space-around;
  max-width: 60vw;
  height: 10vw;
  border-radius: 8px;
  // border: 1px solid #614949;
  background-color: rgba(0, 0, 0, 0.4);
  list-style-type: none;
  transition: 0.16s linear;
  user-select: none;
  .logo-item {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    margin: 0 15px;
    padding: 15px;
    border-radius: 8px;
    cursor: pointer;
    transition: 0.3s linear;
    .item-picture {
      max-width: 8vw;
      max-height: 8vw;
      background-repeat: no-repeat;
      background-size: cover;
    }
    .item-desc {
      color: #eee;
      text-transform: capitalize;
    }
    &.active {
      transition: 0.3s linear;
      background-color: rgba(0, 0, 0, 0.6);
    }
  }
  &:hover {
    transition: 0.16s linear;
    box-shadow: 0px 0px 5px #614949;
  }
}
</style>
