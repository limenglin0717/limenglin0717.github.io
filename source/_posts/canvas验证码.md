---
title: canvas验证码
date: 2017-01-25 14:45:15
tags: canvas
categories: canvas
---
一份简单的前端canvas来生成的验证码

index.html

``` 
<canvas id="canvas" width="120" height="40"></canvas>
<a href="#" id="changeImg">看不清，换一张</a>
```


demo.js
```
/**生成一个随机数**/
function randomNum(min, max) {
    return Math.floor(Math.random() * (max - min) + min);
}
/**生成一个随机色**/
function randomColor(min, max) {
    var r = randomNum(min, max);
    var g = randomNum(min, max);
    var b = randomNum(min, max);
    return "rgb(" + r + "," + g + "," + b + ")";
}
drawPic();
document.getElementById("changeImg").onclick = function(e) {
    e.preventDefault();
    drawPic();
}

/**绘制验证码图片**/
function drawPic() {
    var canvas = document.getElementById("canvas");
    var width = canvas.width;
    var height = canvas.height;
    var ctx = canvas.getContext('2d');
    ctx.textBaseline = 'bottom';

    /**绘制背景色**/
    ctx.fillStyle = randomColor(180, 240); //颜色若太深可能导致看不清
    ctx.fillRect(0, 0, width, height);
    /**绘制文字**/
    var str = 'abcdefghijklmnopqrstuvwxyz123456789'; //看需求可以增减随机出现的内容
    var yzmNum = "";//最后出来的四位验证码
    for(var i = 0; i < 4; i++) {
        var txt = str[randomNum(0, str.length)];
        yzmNum += txt;
        ctx.fillStyle = randomColor(50, 160); //随机生成字体颜色
        ctx.font = randomNum(25, 40) + 'px SimHei'; //随机生成字体大小
        var x = 10 + i * 25;
        var y = randomNum(30, 40);
        var deg = randomNum(-30, 30);
        //修改坐标原点和旋转角度
        ctx.translate(x, y);
        ctx.rotate(deg * Math.PI / 180);
        ctx.fillText(txt, 0, 0);
        //恢复坐标原点和旋转角度
        ctx.rotate(-deg * Math.PI / 180);
        ctx.translate(-x, -y);
    }
    console.log(yzmNum);
        /**绘制干扰线**/
    for(var i = 0; i < 8; i++) {
        ctx.strokeStyle = randomColor(40, 180);
        ctx.beginPath();
        ctx.moveTo(randomNum(0, width), randomNum(0, height));
        ctx.lineTo(randomNum(0, width), randomNum(0, height));
        ctx.stroke();
    }
    /**绘制干扰点**/
    for(var i = 0; i < 100; i++) {
        ctx.fillStyle = randomColor(0, 255);
        ctx.beginPath();
        ctx.arc(randomNum(0, width), randomNum(0, height), 1, 0, 2 * Math.PI);
        ctx.fill();
    }
    return yzmNum;
}   
var yzmNum = drawPic();

```

源码很容易看懂的，自己有什么需求可以在里面相应的更改，备注很明确。
Ps:如果觉得干扰太严重了，可以把字体大小和旋转角度的范围相应的更改，或者把干扰线的条数减少点。