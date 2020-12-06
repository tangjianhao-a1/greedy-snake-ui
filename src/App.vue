<template>
  <div id="app">
    <div
      style="border: 1px solid black; height: fit-content; width: fit-content;"
    >
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
    //定时刷新页面，500ms一次
    setInterval(this.refreshView, 500);
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
          x: 10,
          y: 10,
        },
        {
          x: 10,
          y: 11,
        },
        {
          x: 11,
          y: 11,
        },
      ],
      /**
       * 苹果的坐标
       */
      applePoint: {
        x: 15,
        y: 14,
      },
      /**
       * 棋盘的大小
       */
      checkerboardSize: {
        width: 20,
        height: 20,
      },
    };
  },
  methods: {
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
  },
};
</script>

<style>
.zone {
  width: 30px;
  height: 30px;
  border: 1px solid black;
  display: table-cell;
}
.empty_zone {
  background-color: white;
}

.snake_header_zone {
  background-color: green;
}

.snake_body_zone {
  background-color: black;
}

.apple_zone {
  background-color: red;
}
</style>
