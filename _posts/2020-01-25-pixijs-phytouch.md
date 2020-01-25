# pixi.js中使用phy-touch

[pixi.js](https://github.com/pixijs/pixi.js)是被广泛使用的2D渲染引擎，自动使用webgl或者canvas实现。而phy-touch是一个滑动检测的库，二者结合能够实现触碰交互。在[PyTouch测试使用](https://codetest.github.io/2020/01/14/phytouchtest.html)中，我们通过css3transform对节点的transform属性进行修改达到移动的目的，然后再pixi.js中，实现动画的节点应该是位置不变的，而且pixi也有自身的触碰接口，为了避免冲突，我们可以使用一个父节点包住动画的节点，父节点将触碰访问给PhyTouch，得到虚拟的移动值，再更新动画节点里面的Container或者Spirite.

## 页面结构定义
如上所述，我们需要一个节点包住动画节点，因此需要用到Vue的appendChild接口。
```vue
<template>
    <div id="canvas">
    </div>
</template>
<script lang="ts">
    import { Vue, Component } from "vue-property-decorator"
    import {AnimationControl} from "./model/AnimationControl"
    var animationControl: AnimationControl;
    @Component({ name: "App" })
    export default class App extends Vue{
        mounted(){
            animationControl = new AnimationControl(this);
        }
    }
</script>
<style scoped lang="scss">
</style>
```
在AnimiationControl中将动画节点添加到cavans节点
```typescript
    constructor(instance: Vue){
        this.instance = instance;
        this.app = new PIXI.Application({ 
			width: window.innerWidth, 
			height: window.innerWidth,       
			transparent: false, 
			resolution: 1
        });
        this.instance.$el.appendChild(this.app.view)
```
## 触碰事件响应

[回到主页](https://codetest.github.io)
