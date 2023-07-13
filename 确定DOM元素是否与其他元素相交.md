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
```html
<body>
    <div class="container">
      <div class="item">1</div>
      <div class="item">2</div>
      <div class="item">3</div>
    </div>
    <script>
      var io = new IntersectionObserver(
        (entries) => {
          entries.forEach((item) => {
            if (item.isIntersecting) {
              console.log(item.target.innerText);
            }
          });
        },
        { threshold: 1 }
      );
      const divArr = [...document.querySelectorAll(".item")];
      divArr.forEach((div) => io.observe(div));
    </script>
</body>
```
参考[图片懒加载](https://github.com/amandakelake/blog/issues/46)  
参考[IntersectionObserver API 使用教程 by阮一峰](https://www.ruanyifeng.com/blog/2016/11/intersectionobserver_api.html)
