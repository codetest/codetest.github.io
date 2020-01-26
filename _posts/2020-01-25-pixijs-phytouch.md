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
需要对id为cavans的div进行触碰绑定，
```typescript
        this.touch = new PhyTouch({
            touch:"#canvas",
            vertical: false,
            property: "translateX",
            value: this.app.stage.x,
            step: 1,
            max: 0,
            min: this.minWidth,
        })
```
这里需要注意的是需要定义好value这个位置的初始值，比如前面需要绑定触碰，根据设计需要到某个位置解除绑定显示一些动画，然后再继续绑定触碰。但是在PhyTouch的库里面没有提供解绑定的接口，可以看这个[issue](https://github.com/AlloyTeam/PhyTouch/issues/63)(到现在还没有merge，真是。。。)因此就直接基于PhyTouch现有代码进行修改，添加解绑定接口。具体可以参看[PhyTouchExtenstion.js](https://github.com/codetest/pixijs-phytouch/blob/master/src/model/PhyTouchExtend.js#L135-140)。在触碰绑定的时候主要看两个接口change和animationEnd，根据回调得到的value更新app.stage.x达到移动的效果(享受丝滑的效果)。在实际测试过程中发现手机上可能不能显示部分图片，这个可能与图片大小和格式有关，但是另外的回事了。

具体可以参考[Pixi-PhyTouch](https://github.com/codetest/pixijs-phytouch)。这个例子里面先移动到第一个图的中部位置，然后停止接受触碰，5秒后重新接受触碰。下面是效果图。
![Demo](/images/PixiJs-PhyTouch/Demo.png)


[回到主页](https://codetest.github.io)
