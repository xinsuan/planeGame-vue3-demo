# 打飞机
![](https://github.com/xinsuan/planeGame-vue3-demo/blob/main/planeGameImages/%E6%B8%B8%E6%88%8F%E7%A4%BA%E6%84%8F%E5%9B%BE1.jpg)

## 启动

1. 先启动服务

```shell
yarn start
```

2. 访问 http://localhost:5000/dist/

## 开发

```shell
yarn dev
```

## 构建
```shel
yarn build
```

## tasking

- [x] 地图可滚动
  - [x] 逻辑实现
  - [x] 素材
- [x] 开始页面
- [x] 结束页面
- [ ] 战斗
  - [ ] 敌机
    - [x] 1秒创建一个敌机
    - [x] 从上往下移动
    - [x] 移动的方向随机变换
        - [x] 不允许移动超过地图边界
    - [x] 可以发射炮弹
      - [x] 超出屏幕过，销毁炮弹
    - [x] 被击中 3 次就要爆炸掉
      - [ ] 销毁之前播放爆炸动画
  - [ ] 我方战机
    - [x] 碰到敌机子弹的话会直接死亡跳转到游戏结束页面
    - [x] 我方子弹碰到地方子弹的话，双方子弹都消失

- [ ] 优化
  - [x] 飞机移动不流畅
  - [x] 刚进战斗时，飞机应该自己飞出来
- [ ] 抽离所有的参数配置到 options 内
