# 测试使用PhyTouch
PhyTouch是由腾讯开发的一个触碰滑动管理库。具体可以参见https://github.com/AlloyTeam/PhyTouch
## 缘由
在研究一些H5动画案例的时候，PhyTouch被用上了，而且体验还是挺不错的。因此就测试使用一下。这个测试使用是基于vue+typescript的（typescript是大势所趋，就用这个了）。

## 调研
我是参照http://alloyteam.github.io/PhyTouch/example/simple.html 写的一个demo，做一个在页面中部滚动的列表。
![PhyTouch simple](images/PhyTouchTest/PhyTouch-Simple.png)

可以通过查看上面的源码来看看页面结构。一个wrapper包住scroller,里面是一个列表。scroller有一个transform属性，但overflow-y没有scroll属性。
![Sourc1](images/PhyTouchTest/Sourc1.png)

当我们滑动页面的时候，再次看页面变化。scroller的transform值发生变化。而scroller也没有y轴的scroll属性，因此可以推断是通过检测滑动变化更新transofrm属性达到滑动的效果。
![Source2](images/PhyTouchTest/Source2.png)

我们再看看https://github.com/AlloyTeam/PhyTouch/blob/master/example/simple.html 的js部分。
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
因此通过注入transform属性， PhyTouch在检测运动变化的时候，更新transform属性。在上面的代码中由几个常量值，window.innerHeight - 45 - 48 - 2000，它们的意义如下，
