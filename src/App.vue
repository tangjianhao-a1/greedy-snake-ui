<template>
  <div id="app">
    <!-- 游戏按钮  -->

    <div style="text-align: center; height: 30px; margin-top: 50px">
      <button v-if="!starting" @click="startGame">开始游戏</button>
      <button v-else @click="stopGame">暂停游戏</button>
      <button @click="refresh">重新开始</button>
    </div>

    <!-- 棋盘盒子 -->

    <div
      style="
        border: 1px solid black;
        height: fit-content;
        width: fit-content;
        margin: 0px auto;
      "
    >
      <!-- 横纵格子 -->

      <div v-for="y of checkerboardSize.height" :key="y + refreshTimes">
        <span
          v-for="x of checkerboardSize.width"
          :key="x + refreshTimes"
          :class="['zone', convertZoneTypeToClassName(getZoneType(x, y))]"
        >
        </span>
      </div>
    </div>
  </div>
</template>

<script>
/**
 * 空白区域类型
 */
const EMPTY_ZONE = Symbol("空白区域");

/**
 * 蛇头区域类型
 */
const SNAKE_HEADER_ZONE = Symbol("蛇头区域");

/**
 * 蛇体区域类型
 */
const SNAKE_BODY_ZONE = Symbol("蛇体区域");

/**
 * 苹果区域类型
 */
const APPLE_ZONE = Symbol("苹果区域");

/**
 * 方向上
 */
const UP_DERECTION = Symbol("方向上");

/**
 * 方向下
 */
const DOWN_DERECTION = Symbol("方向下");

/**
 * 方向左
 */
const LEFT_DERECTION = Symbol("方向左");

/**
 * 方向右
 */
const RIGHT_DERECTION = Symbol("方向右");

export default {
  mounted() {
    //绑定键盘事件
    document.onkeydown = this.onKeyDown;
  },
  data() {
    return {
      /**
       * 蛇的运动方向，一共有4个: 上、下、左、右
       */
      moveDirection: UP_DERECTION,
      /**
       * 刷新次数
       */
      refreshTimes: 0,
      /**
       * 蛇身体的坐标
       */
      snakeBodyPointList: [
        {
          x: 3,
          y: 3,
        },
        {
          x: 3,
          y: 4,
        },
        {
          x: 3,
          y: 5,
        },
      ],
      /**
       * 苹果的坐标
       */
      applePoint: {
        x: 1,
        y: 1,
      },
      /**
       * 棋盘的大小
       */
      checkerboardSize: {
        width: 30,
        height: 30,
      },
      /**
       * 定时器
       */
      timer: {},
      /**
       * 按钮控制
       */
      starting: false,
    };
  },
  methods: {
    /**
     * 开始游戏
     */
    startGame() {
      this.starting = true;
      this.timer = setInterval(() => {
        this.refreshView();
        this.add_snack();
        this.commit_suicide();
        this.collision();
      }, 500);
    },
    /**
     * 暂停游戏
     */
    stopGame() {
      clearInterval(this.timer);
      this.starting = false;
    },
    /**
     * 计算蛇身第二节在蛇头的位置
     */
    getSnakeSecondBodyDerection() {
      if (this.snakeBodyPointList[1].y != this.snakeBodyPointList[0].y) {
        if (this.snakeBodyPointList[1].y > this.snakeBodyPointList[0].y) {
          return DOWN_DERECTION;
        } else {
          return UP_DERECTION;
        }
      } else {
        if (this.snakeBodyPointList[1].x > this.snakeBodyPointList[0].x) {
          return RIGHT_DERECTION;
        } else {
          return LEFT_DERECTION;
        }
      }
    },
    /**
     * 键盘事件
     */
    onKeyDown({ key }) {
      let snakeSecondBodyDerection = this.getSnakeSecondBodyDerection();
      //处理键盘事件，改变蛇的运动方向
      if (key == "ArrowUp") {
        //蛇的运动方向不能和蛇的第二节身体所在方向相同
        if (snakeSecondBodyDerection != UP_DERECTION) {
          this.moveDirection = UP_DERECTION;
        }
      }
      if (key == "ArrowDown") {
        if (snakeSecondBodyDerection != DOWN_DERECTION) {
          this.moveDirection = DOWN_DERECTION;
        }
      }
      if (key == "ArrowLeft") {
        if (snakeSecondBodyDerection != LEFT_DERECTION) {
          this.moveDirection = LEFT_DERECTION;
        }
      }
      if (key == "ArrowRight") {
        if (snakeSecondBodyDerection != RIGHT_DERECTION) {
          this.moveDirection = RIGHT_DERECTION;
        }
      }
    },
    /**
     * 刷新界面，计算下一帧画面(蛇运动一次)
     */
    refreshView() {
      this.refreshTimes = this.refreshTimes++;
      //蛇的第n节身子坐标 = 蛇的第n-1节身子坐标
      //......
      //蛇的第三节身子坐标 = 蛇的第二节身子坐标
      //蛇的第二节身子坐标 = 蛇的第一节身子（蛇头）坐标
      for (let i = this.snakeBodyPointList.length - 1; i > 0; i--) {
        this.snakeBodyPointList[i] = { ...this.snakeBodyPointList[i - 1] };
      }
      //蛇头移动
      if (this.moveDirection == UP_DERECTION) {
        this.snakeBodyPointList[0].y--;
      } else if (this.moveDirection == DOWN_DERECTION) {
        this.snakeBodyPointList[0].y++;
      } else if (this.moveDirection == LEFT_DERECTION) {
        this.snakeBodyPointList[0].x--;
      } else if (this.moveDirection == RIGHT_DERECTION) {
        this.snakeBodyPointList[0].x++;
      }
    },

    /**
     * 获取坐标 x,y所在位置的空间类型
     * @param x 横坐标
     * @param y 纵坐标
     */
    getZoneType(x, y) {
      //x,y坐标和苹果坐标相等，则坐标类型为苹果
      if (x == this.applePoint.x && y == this.applePoint.y) {
        return APPLE_ZONE;
      }

      for (let i = 0; i < this.snakeBodyPointList.length; i++) {
        let snakeBodyPoint = this.snakeBodyPointList[i];
        //x,y坐标和蛇坐标相等，则坐标类型为蛇

        if (snakeBodyPoint.x == x && snakeBodyPoint.y == y) {
          //如果是第一节，则类型为蛇头，否则为蛇身
          if (i == 0) {
            return SNAKE_HEADER_ZONE;
          }
          return SNAKE_BODY_ZONE;
        }
      }
      //都不是，则为空白区域
      return EMPTY_ZONE;
    },
    /**
     * 将空间类型 转换为 类名
     */
    convertZoneTypeToClassName(type) {
      if (type == APPLE_ZONE) {
        return "apple_zone";
      }

      if (type == SNAKE_HEADER_ZONE) {
        return "snake_header_zone";
      }

      if (type == SNAKE_BODY_ZONE) {
        return "snake_body_zone";
      }
      return "empty_zone";
    },
    /**
     * 食物随机刷新，蛇身加长
     */
    add_snack() {
      //判断食物和蛇头是否重叠

      let isApple =
        this.snakeBodyPointList[0].x == this.applePoint.x &&
        this.snakeBodyPointList[0].y == this.applePoint.y;

      if (isApple) {
       
       /* 刷新食物 */
        this.change_APPle();

        //接触食物后蛇身增加

        let snack_body = {
          x: this.snakeBodyPointList[0].x,
          y: this.snakeBodyPointList[0].y,
        };
        this.snakeBodyPointList.unshift(snack_body);
      }
    },

    /**
     * 蛇头触碰到自身，游戏结束
     */
    commit_suicide() {
      let snack_head = {
        x: this.snakeBodyPointList[0].x,
        y: this.snakeBodyPointList[0].y,
      };
      this.snakeBodyPointList.some((item, index) => {
        if (snack_head.x === item.x && snack_head.y === item.y && index > 2) {
          clearInterval(this.timer);
          alert("自杀了！哥！！！！！");
          this.refresh();
        }
      });
    },

    /**
     * 随机食物刷新点
     */
    change_APPle() {
      //随机生成食物x，y坐标  并赋值
      this.applePoint.x = this.random(1, this.checkerboardSize.width);
      this.applePoint.y = this.random(1, this.checkerboardSize.height);

      /**
       * 如果食物刷新到蛇身上
       */
      let isSome = this.snakeBodyPointList.some(
        (value) =>
          this.applePoint.x === value.x && this.applePoint.y === value.y
      );

      //重新刷新
      if (isSome) this.change_APPle();
    },

    /**
     * Math对象函数封装
     */
    random(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min;
    },
    /**
     * 定义重新刷新页面
     */
    refresh() {
      location.reload();
    },
    /**
     * 撞墙游戏结束
     */
    collision() {
      let end =
        this.snakeBodyPointList[0].x <= 0 ||
        this.snakeBodyPointList[0].x > this.checkerboardSize.width ||
        this.snakeBodyPointList[0].y <= 0 ||
        this.snakeBodyPointList[0].y > this.checkerboardSize.height;
      if (end) {
        clearInterval(this.timer);
        alert("眼睛拿来干嘛的！撞墙额");
        this.refresh();
      }
    },
  },
};
</script>

<style>
/* 每个小格子样式 */

.zone {
  width: 20px;
  height: 20px;
  display: table-cell;
}

/* 空白地样式 */

.empty_zone {
  background-color: white;
}

/* 蛇头样式*/

.snake_header_zone {
  background-color: green;
}

/* 生身样式 */

.snake_body_zone {
  background-color: black;
}

/* 食物样式 */

.apple_zone {
  background-color: red;
  border-radius: 10px;
}

/* 按钮样式 */

button {
  outline: none;
  height: 30px;
  margin-left: 5px;
}
</style>
