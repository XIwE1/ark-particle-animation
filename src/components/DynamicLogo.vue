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

/** 存储由logos生成的logoImg对象 */
const logoImgs = <LogoImg[]>reactive([]);

// 标记激活logo
let activeLogo = ref<{ name: string; src: string }>();

// 获取canvas画布
const canvas = ref<HTMLCanvasElement | null>(null);

// 获取上下文
let context = ref<CanvasRenderingContext2D | null>(null);

/** canvas实体对象 */
let particleCanvas = ref<ParticleCanvas>();

// 设置画布大小
const width = 400,
  height = 400;

// 设置粒子动画时长
const animateTime = 30;
const opacityStep = 1 / animateTime;

/** 中心影响的半径 */
const Radius = 40;
/** 排斥/吸引 力度 */
const Inten = 0.95;

/** 粒子类 */
class Particle {
  x: number; // 粒子x轴的初始位置
  y: number; // 粒子y轴的初始位置
  totalX: number; // 粒子x轴的目标位置
  totalY: number; // 粒子y轴的目标位置
  mx?: number; // 粒子x轴需要移动的距离
  my?: number; // 粒子y轴需要移动的距离
  vx?: number; // 粒子x轴移动速度
  vy?: number; // 粒子y轴移动速度
  time: number; // 粒子移动耗时
  r: number; // 粒子的半径
  color: number[]; // 粒子的颜色
  opacity: number; // 粒子的透明度
  constructor(totalX: number, totalY: number, time: number, color: number[]) {
    // 设置粒子的初始位置x、y，目标位置totalX、totalY，总耗时time
    this.x = (Math.random() * width) >> 0;
    this.y = (Math.random() * height) >> 0;
    this.totalX = totalX;
    this.totalY = totalY;
    this.time = time;
    // 设置粒子的颜色和半径
    this.r = 1.2;
    this.color = [...color];
    this.opacity = 0;
  }
  // 在画布中绘制粒子
  draw() {
    context.value!.fillStyle = `rgba(${this.color.toString()})`;
    context.value!.fillRect(this.x, this.y, this.r * 2, this.r * 2);
    context.value!.fill();
  }
  /** 更新粒子
   * @param {number} mouseX 鼠标X位置
   * @param {number} mouseY 鼠标Y位置
   */
  update(mouseX?: number, mouseY?: number) {
    // 设置粒子需要移动的距离
    this.mx = this.totalX - this.x;
    this.my = this.totalY - this.y;
    // 设置粒子移动速度
    this.vx = this.mx / this.time;
    this.vy = this.my / this.time;
    // 计算粒子与鼠标的距离
    if (mouseX && mouseY) {
      let dx = mouseX - this.x;
      let dy = mouseY - this.y;
      let distance = Math.sqrt(dx ** 2 + dy ** 2);
      // 粒子相对鼠标距离的比例 判断受到的力度比例
      let disPercent = Radius / distance;
      // 设置阈值 避免粒子受到的斥力过大
      disPercent = disPercent > 7 ? 7 : disPercent;
      // 获得夹角值 正弦值 余弦值
      let angle = Math.atan2(dy, dx);
      let cos = Math.cos(angle);
      let sin = Math.sin(angle);
      // 将力度转换为速度 并重新计算vx vy
      let repX = cos * disPercent * -Inten;
      let repY = sin * disPercent * -Inten;
      this.vx += repX;
      this.vy += repY;
    }
    this.x += this.vx;
    this.y += this.vy;
    if (this.opacity < 1) this.opacity += opacityStep;
  }
  // 切换粒子
  change(x: number, y: number, color: number[]) {
    this.totalX = x;
    this.totalY = y;
    this.color = [...color];
  }
}

/** Logo图片类 */
class LogoImg {
  src: string;
  name: string;
  particleData: Particle[]; // 用于保存筛选后的粒子
  constructor(src: string, name: string) {
    this.src = src;
    this.name = name;
    this.particleData = [];
    let img = new Image();
    img.crossOrigin = "";
    img.src = src;
    // canvas 解析数据源获取粒子数据
    img.onload = () => {
      // 获取图片像素数据
      const tmp_canvas = document.createElement("canvas"); // 创建一个空的canvas
      const tmp_ctx = tmp_canvas.getContext("2d");
      const imgW = width;
      const imgH = ~~(width * (img.height / img.width));
      tmp_canvas.width = imgW;
      tmp_canvas.height = imgH;
      tmp_ctx?.drawImage(img, 0, 0, imgW, imgH); // 将图片绘制到canvas中
      const imgData = tmp_ctx?.getImageData(0, 0, imgW, imgH).data; // 获取像素点数据
      tmp_ctx?.clearRect(0, 0, width, height);

      // 筛选像素点
      for (let y = 0; y < imgH; y += 5) {
        for (let x = 0; x < imgW; x += 5) {
          // 像素点的序号
          const index = (x + y * imgW) * 4;
          // 在数组中对应的值
          const r = imgData![index];
          const g = imgData![index + 1];
          const b = imgData![index + 2];
          const a = imgData![index + 3];
          const sum = r + g + b + a;
          // 筛选条件
          if (sum >= 100) {
            const particle = new Particle(x, y, animateTime, [r, g, b, a]);
            this.particleData.push(particle);
          }
        }
      }
    };
  }
}

// 画布类
class ParticleCanvas {
  canvasEle: HTMLCanvasElement;
  ctx: CanvasRenderingContext2D;
  width: number;
  height: number;
  ParticleArr: Particle[];
  mouseX?: number; // 鼠标X轴位置
  mouseY?: number; // 鼠标Y轴位置
  constructor(target: HTMLCanvasElement) {
    // 设置画布 获取画布上下文
    this.canvasEle = target;
    this.ctx = target.getContext("2d") as CanvasRenderingContext2D;
    this.width = target.width;
    this.height = target.height;
    this.ParticleArr = [];
    // 监听鼠标移动
    this.canvasEle.addEventListener("mousemove", (e) => {
      const { left, top } = this.canvasEle.getBoundingClientRect();
      const { clientX, clientY } = e;
      this.mouseX = clientX - left;
      this.mouseY = clientY - top;
    });
    this.canvasEle.onmouseleave = () => {
      this.mouseX = 0;
      this.mouseY = 0;
    };
  }
  // 改变图片 如果已存在图片则根据情况额外操作
  changeImg(img: LogoImg) {
    if (this.ParticleArr.length) {
      // 获取新旧两个粒子数组与它们的长度
      let newPrtArr = img.particleData;
      let newLen = newPrtArr.length;
      let arr = this.ParticleArr;
      let oldLen = arr.length;

      // 调用change修改已存在粒子
      for (let idx = 0; idx < newLen; idx++) {
        const { totalX, totalY, color } = newPrtArr[idx];
        if (arr[idx]) {
          // 找到已存在的粒子 调用change 接收新粒子的属性
          arr[idx].change(totalX, totalY, color);
        } else {
          // 新粒子数组较大 生成缺少的粒子
          arr[idx] = new Particle(totalX, totalY, animateTime, color);
        }
      }

      // 新粒子数组较小 删除多余的粒子
      if (newLen < oldLen) this.ParticleArr = arr.splice(0, newLen);
      arr = this.ParticleArr;
      let tmp_len = arr.length;
      // 随机打乱粒子最终对应的位置 使切换效果更自然
      while (tmp_len) {
        // 随机的一个粒子 与 倒序的一个粒子
        let randomIdx = ~~(Math.random() * tmp_len--);
        let randomPrt = arr[randomIdx];
        let { totalX: tx, totalY: ty, color } = randomPrt;

        // 交换目标位置与颜色
        randomPrt.totalX = arr[tmp_len].totalX;
        randomPrt.totalY = arr[tmp_len].totalY;
        randomPrt.color = arr[tmp_len].color;
        arr[tmp_len].totalX = tx;
        arr[tmp_len].totalY = ty;
        arr[tmp_len].color = color;
      }
    } else {
      this.ParticleArr = img.particleData.map(
        (item) =>
          new Particle(item.totalX, item.totalY, animateTime, item.color)
      );
    }
  }
  drawCanvas() {
    this.ctx.clearRect(0, 0, this.width, this.height);
    this.ParticleArr.forEach((particle) => {
      particle.update(this.mouseX, this.mouseY);
      particle.draw();
    });
    window.requestAnimationFrame(() => this.drawCanvas());
  }
}

// 激活logo
function clickLogo(prt_canvas: ParticleCanvas, logoItem: LogoImg) {
  prt_canvas.changeImg(logoItem);
  activeLogo.value = logoItem;
}

// 将logo数据实例化为logoImg对象
for (let item of logos) {
  logoImgs.push(new LogoImg(item.url, item.label));
}

onMounted(() => {
  if (canvas.value) {
    context.value = canvas.value.getContext("2d");
    particleCanvas.value = new ParticleCanvas(canvas.value);
    particleCanvas.value.drawCanvas();
  }
});
</script>

<template>
  <div class="canvas-container">
    <canvas ref="canvas" :width="width" :height="height"></canvas>
  </div>
  <ul class="logo-options-container">
    <li
      v-for="logoItem in logoImgs"
      :class="['logo-item', activeLogo === logoItem && 'active']"
      @click="clickLogo(particleCanvas, logoItem)"
      data-cursor="pointer"
    >
      <img :src="logoItem.src" class="item-picture" />
      <div class="item-desc">
        {{ logoItem.name }}
      </div>
    </li>
  </ul>
</template>

<style lang="scss" scoped>
.canvas-container {
  margin-top: 50px;
  display: flex;
  width: 100vw;
  height: 50vh;
  border-radius: 8px;
  background-color: rgba(0, 0, 0, 0.4);

  canvas {
    margin: auto;
  }
}

.logo-options-container {
  padding: 10px;
  margin: 0 auto;
  margin-top: 5vw;
  display: flex;
  justify-content: space-around;
  max-width: 80vw;
  height: 15vw;
  border-radius: 8px;
  // border: 1px solid #614949;
  background-color: rgba(0, 0, 0, 0.4);
  list-style-type: none;
  transition: 0.16s linear;
  user-select: none;

  .logo-item {
    padding: 15px;
    height: calc(100% - 30px);
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    border-radius: 8px;
    transition: 0.3s linear;

    .item-picture {
      height: 90%;
      background-repeat: no-repeat;
      background-size: cover;
    }

    .item-desc {
      flex: 1;
      color: #eee;
      text-transform: capitalize;
    }

    &.active {
      transition: 0.3s linear;
      background-color: rgb(116, 116, 116);
      // background-color: rgba(0, 0, 0, 0.6);
    }
  }

  box-shadow: 0px 0px 5px #614949;

  &:hover {
    transition: 0.16s linear;
  }
}
</style>
