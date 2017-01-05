---
title: 网页组件制作之CSS3+JS 实现散列画廊特效
date: 2016-11-22 14:41:03
categories: coding
tags:
  - HTML
  - CSS
  - JavaScript
---

**在该笔记中将学到列表展示中散列画廊特效的方法与展示效果**

本系列的是参考[慕课网 Lyn](http://www.imooc.com/u/104593/courses?sort=publish)老师的视频，仅供个人查阅使用，具体讲解推荐参考视频。

显示效果如下：

![](http://ofjjubwp5.bkt.clouddn.com/image/png/img70.PNG)

其中，点击屏幕下部的按钮，可以实现居中照片的切换。点击屏幕中的照片，也可以控制照片正反面的切换动画。

<!--more-->

## VCD 分解

1.`View`：视觉，是 HTML + CSS，基本界面模版
2. `Controller`：控制，JavaScript，内容处理，事件处理
3. `Data`：数据，data.js，非必须，助于理解

### 视觉部分

![](http://ofjjubwp5.bkt.clouddn.com/image/png/img66.PNG)

### 数据部分

**有一个 data.js 来处理所有的海报图片链接、标题、描述**

### 控制部分

控制海报的批处理，分配海报的位置，设置控制条的动作，海报的翻面控制。

![](http://ofjjubwp5.bkt.clouddn.com/image/png/img68.PNG)

**使用 `-webkit-perspective:800px` 设置子元素对 3D 效果的支持。这里设置的是子元素距离视图的位置**

**使用 `-webkit-backface-visibility:hidden` 设置元素不面向屏幕的时候进行隐藏**

**使用 `-webkit-transform-style:preserve-3d` 设置支持子元素的 3D 效果**

** `-webkit-transform: rotateY(0deg)` 定义元素旋转的角度**

**使用 `transition` 设置过渡动画**

### 计算周围海报的位置范围

为了不让周围的海报随机显示的时候，不要太超出边界，或者不要太遮挡中间的海报，所以计算海报随机位置的时候，需要给出一个范围

![](http://ofjjubwp5.bkt.clouddn.com/image/png/img69.PNG)

	<html>
	    <head>
	        <meta charset="utf-8">
	        <title>海报画廊</title>
	        <style type="text/css">
	            *{padding: 0; margin: 0;font-size: 14px;}
	            body{border: 1px solid rebeccapurple;}
	            .wrap{width: 800px;height: 400px;margin: 100px auto;border: 1px solid rebeccapurple;overflow: hidden;
	            perspective: 800px;-webkit-perspective:800px;background-color: #666}
	            .photo{width: 160px;height: 190px;position: absolute;box-shadow: 0 0 1px rgba(0, 0, 0, 0.01);cursor: pointer;transition: all 0.5s;}
	            .photo .side-front{padding: 10px;position: absolute;}
	            .photo .side-front img{width: 140px;height: 150px;display: block;margin: 0 auto;border: 1px solid #aaa;}
	            .photo .side-front p{display: block;height: 30px;line-height: 30px;text-align: center;color: #333;font-size: 12px}
	            .photo .side-back{padding: 10px;position: absolute;width: 140px;height: 170px;overflow: hidden;background-color: #ddd;}
	            .photo .side-back p{color: #666;font-size: 12px;line-height: 1.5em;}
	            .photo-center{top: 50%; left: 50%;margin: -80px 0 0 -95px;-webkit-transform: rotateY(0deg);z-index: 1;}
	            /*设置图片旋转的效果*/
	            .photo-wrap{-webkit-transform-style: preserve-3d;-webkit-backface-visibility: hidden;width: 100%;height: 100%;transition: all 0.6s;border: 1px solid #aaa;}
	            .photo-wrap .side-front{-webkit-transform: rotateY(0deg);}
	            .photo-wrap .side-back{-webkit-transform: rotateY(180deg);}
	            .photo-f .photo-wrap{-webkit-transform: rotateY(0deg);}
	            .photo-b .photo-wrap{-webkit-transform: rotateY(180deg);}
	            .nav{width: 100%;height: 26px;line-height: 26px;position: absolute; bottom: 36px;text-align: center;}
	            .nav-i{display: inline-block;background-color: #333;width: 26px;height: 26px;border-radius: 13px;margin:0 10px;cursor: pointer;transform: scale(0.5);}
	            .nav-current{transform: scale(0.8);}
	        </style>
	        <script>
	            function turnPage(elem){
	            let className = elem.className;
	            if(/photo-f/.test(className)){
	                className = className.replace("photo-f", "photo-b");
	            } else {
	                className = className.replace("photo-b", "photo-f");
	            }
	            elem.className = className;
	        }
	
	        function randomInt(min, max){
	            let diff = max-min+1;
	            return Math.ceil(Math.random()*diff + min - 1);
	        }
	
	        function randomArray(arr){
	            let rArr = [];
	            do{
	                let index = randomInt(0, arr.length-1);
	                rArr.push(arr.splice(index,1).pop());
	            }while(arr.length>0)
	            return rArr;
	        }
	
	        function range(){
	            let range = { left:{x:[], y:[]}, right:{x:[], y:[]}};
	
	            let wrap = {
	                width:document.getElementById("wrap").clientWidth,
	                height:document.getElementById("wrap").clientHeight
	            }; 
	            let photo = {
	                width:document.getElementsByClassName("photo")[0].clientWidth,
	                height:document.getElementsByClassName("photo")[0].clientHeight
	            };
	
	            range.left.x.push(0-photo.width/2);
	            range.left.x.push(wrap.width/2-photo.width);
	            range.left.y.push(0-photo.height/2);
	            range.left.y.push(wrap.height-photo.height/2);
	
	            range.right.x.push(wrap.width/2+photo.width/2);
	            range.right.x.push(wrap.width-photo.width/2);
	            range.right.y.push(0-photo.height/2);
	            range.right.y.push(wrap.height-photo.height/2);
	            return range;
	        }
	
	        //根据index 选择对应的 photo，index的值从1-data.length
	        function setCenter(index){
	            let photos = document.getElementsByClassName("photo");
	            let photo = photos[index-1];
	            for(let i=0;i<photos.length;i++){
	                photos[i].className = photos[i].className.replace(" photo-center", "");
	                photos[i].style.left = "";
	                photos[i].style.top = "";
	                photos[i].style['-webkit-transform'] = "rotateY(0deg)";
	                photos[i].className = photos[i].className.replace(" photo-b", " photo-f");
	            }
	            photo.className += " photo-center";
	        }
	
	        function setLeftRight(index){ 
	
	            let r= range();
	            let _photos = document.getElementsByClassName("photo");
	            let photos = [];
	            for (let i=0; i<_photos.length; i++){
	                if(i!== (index-1))
	                photos.push(_photos[i]);
	                _photos[i].className = _photos[i].className.replace(" photo-b", " photo-f");
	            }
	            photos = randomArray(photos);
	            let photoLeft = photos.splice(Math.ceil(photos.length/2), Math.ceil(photos.length/2));
	            let photoRight = photos;
	            for(let i=0; i<photoLeft.length; i++){
	                let photo = photoLeft[i];
	                photo.style.left = randomInt(r.left.x[0], r.left.x[1]) + "px";
	                photo.style.top = randomInt(r.left.y[0], r.left.y[1]) + "px";
	                photo.style['-webkit-transform'] = 'rotate('+randomInt(-60, 60) + 'deg)';
	            }
	            for(let i=0; i<photoRight.length; i++){
	                let photo = photoRight[i];
	                photo.style.left = randomInt(r.right.x[0], r.right.x[1]) + "px";
	                photo.style.top = randomInt(r.right.y[0], r.right.y[1]) + "px";
	                photo.style['-webkit-transform'] = 'rotate('+randomInt(-60, 60) + 'deg)';
	            }
	        }
	
	
	
	
	        let data = [
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/1.jpg",title : "照片1",desc : "照片1的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/2.jpg",title : "照片2",desc : "照片2的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/3.jpg",title : "照片3",desc : "照片3的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/4.jpg",title : "照片4",desc : "照片4的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/5.jpg",title : "照片5",desc : "照片5的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/6.jpg",title : "照片6",desc : "照片6的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/7.jpg",title : "照片7",desc : "照片7的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/8.jpg",title : "照片8",desc : "照片8的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/9.jpg",title : "照片9",desc : "照片9的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/10.jpg",title : "照片10",desc : "照片10的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/11.jpg",title : "照片11",desc : "照片11的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/12.jpg",title : "照片12",desc : "照片12的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/13.jpg",title : "照片13",desc : "照片13的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/14.jpg",title : "照片14",desc : "照片14的描述"},
	            {img : "http://ofjjubwp5.bkt.clouddn.com/image/jpg/15.jpg",title : "照片15",desc : "照片15的描述"}
	        ];
	
	        window.onload = function(){
	
	            //批处理生成照片
	            let template = document.getElementById("photos").innerHTML;
	            let navs = document.getElementById("navs");
	            let navTemplate = navs.innerHTML;
	            let htmlColl = [];
	            let navColl = [];
	            for(let i=0;i<data.length;i++){
	                let html = template.replace("{{index}}", i+1).replace("{{img}}", data[i].img).replace("{{title}}", data[i].title).replace("{{desc}}", data[i].desc);
	                htmlColl.push(html);
	                let nav = navTemplate.replace("{{index}}", i+1);
	                navColl.push(nav);
	            }
	            document.getElementById("photos").innerHTML = htmlColl.join("");
	            navs.innerHTML = navColl.join("");
	            let navIs = document.getElementsByClassName("nav-i");
	            for(let i=0; i<navIs.length; i++){
	                navIs[i].onclick = function(){
	                    let photos = document.getElementsByClassName("photo");
	                    if(/nav-current/.test(this.className)){
	                        let className = photos[i].className;
	                        if(/photo-f/.test(className)){
	                            photos[i].className = className.replace(" photo-f", " photo-b");
	                        } else{
	                            photos[i].className = className.replace(" photo-b", " photo-f");
	                        }
	                    } else{
	                        let This = this;
	                        let navIs = document.getElementsByClassName("nav-i");
	                        for(let j=0;j<navIs.length;j++){
	                            navIs[j].className = navIs[j].className.replace(" nav-current", "");
	                        }
	                        This.className += " nav-current";
	                        setCenter(i+1); 
	                        setLeftRight(i+1);
	                        
	                    }
	                    
	                }
	            }
	
	            //在开始的时候，随机选取一张照片放在中间区域
	            let c = randomInt(1, 8);
	            setCenter(c);
	            setLeftRight(c);
	        }
	        </script>
	    </head>
	    <body>
	        <div class="wrap" id="wrap">
	            <!-- photo 负责平移和旋转-->  
	            <div id="photos">
	                <div class="photo photo-f" onclick="turnPage(this)" id="photo-{{index}}">
	                    <!-- photo-wrap 负责照片的翻转-->
	                    <div class="photo-wrap">
	                        <div class="side-front">
	                            <img src="{{img}}" alt="">
	                            <p>{{title}}</p>
	                        </div>
	                        <div class="side-back">
	                            <p>{{desc}}</p>
	                        </div>                                   
	                    </div>
	                </div>
	            </div>          
	            <div class="nav" id="navs">
	                <i class="nav-i" id="nav-{{index}}"></i>
	            </div>
	        </div>
	    </body>
	</html>
