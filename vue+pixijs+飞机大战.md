# vue3+pixijs+飞机大战

### runtime-canvas

```js
import { crateRenderer } from 'vue';
import { Container, Texture, Sprite, Text } from 'pixi.js';

const renderer = createRenderer({
    createElement(type) {
        let element;
        switch (type) {
            case 'container':
                element = new Container();
                break;
            case 'sprite':
                element = new Sprite();
                break;
        }
        return element;
    },
    patchProp(el, key, preValue, nextValue) {
        switch (key) {
            case 'texture':
                el.texture = Texture.from(nextValue);
                break;
            case 'onClick':
                // 使用过程中 @click 在这里就得添加为 onClick 字眼才能给识别
                el.on('pointertap', nextValue);
                break;
            default:
                // el.x = nextValue
                // el.y = nextValue
                // 上述两个数值的简写版本
                el[key] = nextValue;
                break;
        }
    },
    // 创建一个文本
    createText(text) {
        return new Text(text);
    },
    // 必须要加，不然 vue 会报错
    nextSibling() {},
    createComment() {}
});
export function createApp(rootComponent) {
    return renderer.createApp(rootComponent);
}
```

### 视图切换

```html
<!-- App.vue -->
<template>
  <container>
    <currentPage @change-page="changePage"></currentPage>
  </container>
</template>
<script>
import StartPage from "./pages/StartPage";
import GamePage from "./pages/GamePage";
import { computed, ref } from "vue";
export default {
  setup() {
    // 通过数据响应来改变视图的页面切换
    const currentPageName = ref("GamePage");
    const currentPage = computed(() => {
      if (currentPageName.value === "StartPage") {
        return StartPage;
      } else if (currentPageName.value === "GamePage") {
        return GamePage;
      }
    });
    const changePage = (pageName) => {
      currentPageName.value = pageName
    };
    return {
      currentPage,
      changePage,
    };
  },
};
</script>
```

### 背景地图的滑动

```html
<!-- Map.vue -->
<template>
  <container>
    <sprite :texture="mapImg" :y="moveY1"> </sprite>
    <sprite :texture="mapImg" :y="moveY2"></sprite>
  </container>
</template>

<script>
import { ref } from "vue";
import mapImg from "../assets/map.jpg";
import { game } from "../game";
export default {
  setup() {
    const viewHeight = 1080;
    const moveY1 = ref(0);
    const moveY2 = ref(-viewHeight);
    const speed = 5;
      
    game.ticker.add(() => {
      moveY1.value += speed;
      moveY2.value += speed;
      // 视图重新网跳到上面补齐
      if (moveY1.value >= viewHeight) {
        moveY1.value = -viewHeight;
      }
      if (moveY2.value >= viewHeight) {
        moveY2.value = -viewHeight;
      }
    });

    return {
      mapImg,
      moveY1,
      moveY2,
    };
  },
};
</script>

```

### 碰撞检测

```js
// 矩形的碰撞检测
export function hitTestObject(objA, objB) {
  return (
    objA.x + objA.width >= objB.x &&
    objB.x + objB.width >= objA.x &&
    objA.y + objA.height >= objB.y &&
    objB.y + objB.height >= objA.y
  );
}
```

```js
// 使用
// pages/GamePage.vue
function useFighting() {
    const fighting = () => {
        for(let index = 0; index < enemyPlanes.length; index++) {
            const enemyInfo = enemyPlanes[index];
            if(hitTestObject(enemyInfo, planeInfo)) {
                // game restart
                // 跳转页面
                emit('change-page', 'EndPage');
            }
        }
        enemyPlanes.forEach((enemy, enemyIndex) => {
            bullets.forEach((bullet, bulletIndex) => {
                if(hitTestObject(enemy, bullet)) {
                    enemyPlanes.splice(enemyIndex, 1);
                    bullets.splice(bulletIndex, 1);
                }
            });
        });
    };
    // 放到其中实时监听是否碰撞
    onMounted(() => {
        game.ticker.add(fighting);
    });
    onUnmounted(() => {
        game.ticker.remove(fighting);
    })
}
useFighting();
```

### 飞机发射弹头

```js
// components/Plane.vue
export function usePlane({ onAttack }) {
    const planeInfo = reactive({
        x: 200,
        y: 400,
        width: 258,
        height: 364
    });
    const Attack = () => {
        const handleAttack = (e) => {
            if(e.code === "Space") {
                // 回调返回当前飞机的位置信息
                if(onAttack) {
                    onAttack({
                        x: planeInfo.x + 100,
                        y: planeInfo.y
                    });
                }
            }
        };
        onMounted(() => {
            window.addEventListener("keyup", handleAttack);
        });
        onUnmounted(() => {
            window.removeEventListener("keyup", handleAttack);
        })
    };
    attack();
}

```

```js
// pages/GamePage.vue
export default {
    setup(props, { emit }) {
        const { bullets, addBullet } = useBullets();
        const { planeInfo } = usePlane({
            onAttack(position) {
                // 添加子弹（给与当前飞机的位置信息）
                addBullets(position.x, position.y);
            }
        });
    }
}
```

## 源码

### pixiJS文字添加（得分）

```js
// src/pathProp.js
// line-13
if (key === "on" || key === "texture" || key === "style" || key === "text") {
    switch (key) {
        case "text":
            let text = new PIXI.Text(nextValue);
        	el.text = text._text;
        	break;
    }
}
```

```js
// src/nodeOps.js
else if(tag === "Text"){
	element = new PIXI.Text()
	element.x = 0;
    element.y = 0;
}
```

```js
// component/page/GamePage.js
	const createRichText = () => {
      return h("Text", {
        x: 5,
        y: 5,
        style: {
          fontFamily: 'Arial',
          fontSize: '36px',
          fontWeight: 'italic',
          fill: '#FCF3CF',
          stroke: '#EEC211',
          strokeThickness: '3',
          dropShadow: true,
          dropShadowColor: '#FFD633',
          dropShadowAngle: Math.PI / 6,
          dropShadowDistance: 4,
          wordWrap: true,
          wordWrapWidth: 440
        },
        text: 'goals:'+ctx.selfPlane.attackCount
      });
    }
    return h("Container", [
      h(Map),
      createRichText(),
      createSelfPlane(),
      ...ctx.selfBullets.map(createBullet),
      ...ctx.enemyPlaneBullets.map(createBullet),
      ...ctx.enemyPlanes.map(createEnemyPlane),
    ]);
```

### 子弹节流

```js
// component/Plane.js
const handleTicker = () => {
    if (isAttack) {
      // 用户单击一次只发射一颗子弹
      startTime++;
      if (startTime > ATTACK_INTERVAL) {
        emitAttack();
        startTime = 0;
      }
    }
  };


  onMounted(() => {
    // 节流，避免渲染子弹数量过多造成浏览器运行卡顿
    // 局限性太大，当节流时隙过大，用户单击不能发射子弹，想法：通过获取第一次 down 到完整的 up 时间，和 短时间进行判断是否需要节流操作
    game.ticker.add(throttle(handleTicker, 100));
  });

  onUnmounted(() => {
    game.ticker.remove(handleTicker);
  });
```

