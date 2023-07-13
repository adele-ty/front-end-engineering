以下情况都需要用到相交检测：  
- 图片懒加载——当图片滚动到可见时才进行加载  
- 内容无限滚动——也就是用户滚动到接近内容底部时直接加载更多，而无需用户操作翻页，给用户一种网页可以无限滚动的错觉  
- 检测广告的曝光情况——为了计算广告收益，需要知道广告元素的曝光情况  
- 在用户看见某个区域时执行任务或播放动画  

# 使用Element.getBoundingClientRect()
```html
<body>
    <div class="item" index="0">0</div>
    <div class="item" index="1">1</div>
    <div class="item" index="2">2</div>
</body>
<script>
    const items = Array.from((document.getElementsByClassName("item")));

    function isInViewport(item) {
      const notBelow = item.getBoundingClientRect().top <= window.innerHeight ? true : false;
      const notAbove = item.getBoundingClientRect().bottom >= 0 ? true : false;
      if (notAbove && notBelow) return true;
    }
    
    const scrollEvent = function () {
      items.forEach((item) => {
        if (isInViewport(item)) {
          console.log("第" + item.getAttribute("index") + "个盒子出现在视口中");
        }
      });
    };

    window.addEventListener("scroll", scrollEvent);
    window.addEventListener("load", scrollEvent);
</script>
```
getBoundingClientRect()会触发页面回流，并且需要监听 scroll 事件，然后调用目标元素的 getBoundingClientRect 方法获取元素位置，判断其是否出现在视口之内，scroll 触发非常密集，导致计算量很大，性能不佳。

# Intersection Observer
IntersectionObserver 提供了一种异步观察目标元素与根元素交叉状态的方法，不随着目标元素的滚动同步触发。

## 使用 Intersection Observer 实现虚拟列表
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      * {
        margin: 0;
        padding: 0;
      }
      .container {
        text-align: center;
      }
      .container > li {
        margin: 5px auto;
        width: 80%;
        height: 120px;
        outline: 1px solid red;
        list-style-type: none;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <li class="item" index="0">0</li>
      <li class="item" index="1">1</li>
      <li class="item" index="2">2</li>
      <li class="item" index="3">3</li>
      <li class="item" index="4">4</li>
    </div>
    <script>
      const container = document.getElementsByClassName("container")[0];
      const itemHeight = 120, visibleCount = Math.ceil(window.innerHeight / itemHeight);
      let startIndex = 0, endIndex = startIndex + visibleCount;

      const addItems = function () {
        startIndex = endIndex, endIndex += visibleCount;
        const willAddItems = document.createDocumentFragment();
        for (let index = startIndex; index < endIndex; index++) {
          const li = document.createElement("li");
          li.setAttribute("class", "item");
          li.setAttribute("index", index);
          li.innerHTML = index;
          io.observe(li);
          willAddItems.appendChild(li);
        }
        container.appendChild(willAddItems);
      };

      const removeItems = function () {
        let count = visibleCount;
        while (count--) {
          const item = container.firstElementChild;
          io.unobserve(item);
          container.removeChild(item);
        }
      };

      var io = new IntersectionObserver((entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            // 获取当前进入可视区域列表项的索引
            const CurIndex = +entry.target.getAttribute("index");

            // 此时列表中只有初始时的数据，无法滚动，无法触发监视器来添加数据，因此直接添加新的items
            if (CurIndex === endIndex - 1 && window.pageYOffset <= 50) 
              addItems();

            // 列表滚动，当前进入可视区的元素是列表中的最后一条数据
            // 滚出可视区的元素的个数为visibleCount - 1到visibleCount之间
            // 将(visibleCount - 1) * itemHeight + 50作为条件而不是visibleCount * itemHeight
            // 因为若将后者作为条件，列表中的最后一个元素已完整的进入了可视区，可视区就无法再滚动，也就无法触发监视器
            if (
              CurIndex === endIndex - 1 &&
              window.pageYOffset >= (visibleCount - 1) * itemHeight + 50
            ) {
              removeItems();
              addItems();
            }
          }
        });
      });

      const list = [...document.querySelectorAll(".item")];
      list.forEach((li) => io.observe(li));
    </script>
  </body>
</html>
```
此虚拟列表中始终只有10条列表项，只能往下滚动，因为向下滚动时，会将顶部滚出可视区的内容删除，因此无法往上滚，往上滚和往下滚的逻辑一样，就不再重复写了

参考[图片懒加载](https://github.com/amandakelake/blog/issues/46)  
参考[IntersectionObserver API 使用教程 by阮一峰](https://www.ruanyifeng.com/blog/2016/11/intersectionobserver_api.html)  
参考[聊聊前端开发中的长列表](https://zhuanlan.zhihu.com/p/26022258) 
