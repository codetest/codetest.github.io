# 测试使用PhyTouch
PhyTouch是由腾讯开发的一个触碰滑动管理库。具体可以参见 [PhyTouch GitHub](https://github.com/AlloyTeam/PhyTouch)
## 缘由
在研究一些H5动画案例的时候，PhyTouch被用上了，而且体验还是挺不错的。因此就测试使用一下。这个测试使用是基于vue+typescript的（typescript是大势所趋，就用这个了）。

## 调研
我是参照[PhyTouch Simple](http://alloyteam.github.io/PhyTouch/example/simple.html)写的一个demo，做一个在页面中部滚动的列表。
![PhyTouch simple](/images/PhyTouchTest/PhyTouch-Simple.png)

可以通过查看上面的源码来看看页面结构。一个wrapper包住scroller,里面是一个列表。scroller有一个transform属性，但overflow-y没有scroll属性, wrapper则是overflow:hidden。
![Sourc1](/images/PhyTouchTest/Sourc1.png)

当我们滑动页面的时候，再次看页面变化。scroller的transform值发生变化。而scroller也没有y轴的scroll属性，因此可以推断是通过检测滑动变化更新transofrm属性达到滑动的效果。
![Source2](/images/PhyTouchTest/Source2.png)

我们再看看[PhyTouch Simple](http://alloyteam.github.io/PhyTouch/example/simple.html) 的js部分。
```js
    <script>
        var target = document.querySelector("#scroller");
        //给element注入transform属性
        Transform(target,true);

        var at = new PhyTouch({
            touch:"#wrapper",//反馈触摸的dom
            vertical: true,//不必需，默认是true代表监听竖直方向touch
            target: target, //运动的对象
            property: "translateY",  //被滚动的属性
            sensitivity: 1,//不必需,触摸区域的灵敏度，默认值为1，可以为负数
            factor: 1,//不必需,默认值是1代表touch区域的1px的对应target.y的1
            min: window.innerHeight - 45 - 48 - 2000, //不必需,滚动属性的最小值
            max: 0, //不必需,滚动属性的最大值
            step: 40,
            animationEnd: function (value) {
                //console.log(value);
            },
            pressMove: function (evt,value) {
                //console.log(evt.deltaX + "_" + evt.deltaY + "__" + value);
            }
        })

        document.addEventListener("touchmove", function (evt) {
            evt.preventDefault();
        }, false);

    </script>
```
在Transform函数里面，可以看到对target使用observe进行属性监控。在observe的回调里面，通过matric计算更新style的transform属性。因此只要通过函数更新target的translateX, translateY等属性，就触发回调更新style的transform。
```js
    function Transform(obj, notPerspective) {
        if(obj.___mixCSS3Transform) return;
        var observeProps = ["translateX", "translateY", "translateZ", "scaleX", "scaleY", "scaleZ", "rotateX", "rotateY", "rotateZ", "skewX", "skewY", "originX", "originY", "originZ"],
            objIsElement = isElement(obj);
        if (!notPerspective) {
            observeProps.push("perspective");
        }
        obj.___mixCSS3Transform = true
        observe(
            obj,
            observeProps,
            function () {
                var mtx = obj.matrix3d.identity().appendTransform(obj.translateX, obj.translateY, obj.translateZ, obj.scaleX, obj.scaleY, obj.scaleZ, obj.rotateX, obj.rotateY, obj.rotateZ, obj.skewX, obj.skewY, obj.originX, obj.originY, obj.originZ);
                var transform = (notPerspective ? "" : "perspective(" + obj.perspective + "px) ") + "matrix3d(" + Array.prototype.slice.call(mtx.elements).join(",") + ")";
                if (objIsElement) {
                    obj.style.transform = obj.style.msTransform = obj.style.OTransform = obj.style.MozTransform = obj.style.webkitTransform = transform;
                } else {
                    obj.transform = transform;
                }
            });

        obj.matrix3d = new Matrix3D();
        if (!notPerspective) {
            obj.perspective = 500;
        }
        obj.scaleX = obj.scaleY = obj.scaleZ = 1;
        //由于image自带了x\y\z，所有加上translate前缀
        obj.translateX = obj.translateY = obj.translateZ = obj.rotateX = obj.rotateY = obj.rotateZ = obj.skewX = obj.skewY = obj.originX = obj.originY = obj.originZ = 0;
    }
```

在PhyTouch的index.js里面有以下的对target的属性更新。
```js
        _move: function (evt) {
            if (this.isTouchStart) {
                var len = evt.touches.length,
                    currentX = evt.touches[0].pageX,
                    currentY = evt.touches[0].pageY;

                if (this._firstTouchMove && this.lockDirection) {
                    var dDis = Math.abs(currentX - this.x1) - Math.abs(currentY - this.y1);
                    if (dDis > 0 && this.vertical) {
                        this._preventMove = true;
                    } else if (dDis < 0 && !this.vertical) {
                        this._preventMove = true;
                    }
                    this._firstTouchMove = false;
                }
                if(!this._preventMove) {
                    var d = (this.vertical ? currentY - this.preY : currentX - this.preX) * this.sensitivity;
                    var f = this.moveFactor;
                    if (this.isAtMax() && (this.reverse ? -d : d) > 0) {
                        f = this.outFactor;
                    } else if (this.isAtMin() && (this.reverse ? -d : d) < 0) {
                        f = this.outFactor;
                    }
                    d *= f;
                    this.preX = currentX;
                    this.preY = currentY;
                    if (!this.fixed) {
                        var detalD = this.reverse ? -d : d;
                        this.target[this.property] += detalD;
```

因此简单理解就是，通过注入transform属性， PhyTouch在检测运动变化的时候，更新transform属性。在上面的代码中由几个常量值，window.innerHeight - 45 - 48 - 2000，可以通过下面的截图看到其中的意义。2000就是滚动部分的高度，45 48就是对应的top和bottom。当scroller滚动到底部的时候transform的值就是window.innerHeight - 45 - 48 - 2000
![Source4](/images/PhyTouchTest/Source4.png)

## 实现
因此基于上面的分析使用下面的代码来做demo。css3transform进行实质的target css style更新。
```vue
<template>
    <div style="overflow:hidden;">
        <div style="width:100%;font-size:40px;line-height:48px;color:green;">
            滚动条展示
        </div>
        <div :style="styleOpt" id="listwrapper" class="listwrapper">
            <div class="list" id="list">
                <div v-for="(item,index) in list" :key="index" class="item">
                    {{item}}
                </div>
            </div>
        </div>
        <div style="width:100%;font-size:40px;line-height:48px;border-bottom: solid 1px;color:green;">
            底部条展示
        </div>
    </div>
</template>
<script lang="ts">
    import { Vue, Component } from "vue-property-decorator"
    const PhyTouch = require("phy-touch")
    const Transform = require("css3transform")

    @Component({ name: "App" })
    export default class App extends Vue{
        list: string[] = []
        styleOpt: any = { "height": "0" }
        bindTouch() {
            var target = <HTMLElement>document.getElementById("list")
            Transform(target, true)
            var phy = new PhyTouch({
                touch:"#listwrapper",//反馈触摸的dom
                vertical: true,//不必需，默认是true代表监听竖直方向touch
                target: target, //运动的对象
                property: "translateY",  //被滚动的属性
                sensitivity: 1,//不必需,触摸区域的灵敏度，默认值为1，可以为负数
                factor: 1,//不必需,默认值是1代表touch区域的1px的对应target.y的1
                min: window.innerHeight - 48 - 48 - target.scrollHeight, //不必需,滚动属性的最小值
                max: 0, //不必需,滚动属性的最大值
                step: 40
            })
        }

        mounted() {
            for (var i = 0; i < 25; ++i) {
                this.list.push("row" + i.toString())
            }

            this.styleOpt["height"] = (window.innerHeight - 96).toString() + "px"
            document.body.addEventListener("touchmove", function (evt) {
                evt.preventDefault();
            }, false);
            Vue.nextTick(() => {
                this.bindTouch()
            })
        }
    }
</script>
<style scoped lang="scss">
    .listwrapper {
        background:#ccc;
        overflow:hidden;
    }

    .list {
        width:100%;
    }

    .item {
        line-height: 40px;
        font-size:32px;
        border-bottom-width:1px;
        border-bottom-color:black;
        border-bottom-style:solid;
    }
</style>
```

效果如下
![DemoPages](/images/PhyTouchTest/Source5.png)

## repo
完整的demo repo可以参见[PhyTouchTest](https://github.com/codetest/PhyTouchTest)

[回到主页](https://codetest.github.io/)
