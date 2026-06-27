---
hide:
  - footer
comments: true
---

# 友链

## Why?

[FiveYellowMice](https://fiveyellowmice.com/zh/):

> 然而为什么还是放了这个友情链接页面呢？
>
> 因为这样可以让感兴趣的人通过别处的友情链接来发现我的博客的存在，同样也可以让别人的博客通过我这里的友情链接被发现。**友情链接就这样被用来传递友情，而非搜索结果排名。**

## 咱加的？

<div id="links">
<style>
.link-navigation {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
    gap: 1rem;
    max-width: 100%;
}
.card {
    width: 100%;
    max-width: 320px;
    height: 90px;
    font-size: 1rem;
    padding: 10px 20px;
    border-radius: 25px;
    transition: transform 0.15s, box-shadow 0.15s, background 0.15s;
    display: flex;
    align-items: center;
    color: #333;
    justify-self: center;
}
.card:hover {
    transform: translateY(0px) scale(1.05);
    background-color: rgba(68, 138, 255, 0.1);
    color: #040000;
}
.card a {
    border: none;
}
.card .ava {
    width: 3rem !important;
    height: 3rem !important;
    margin: 0 !important;
    margin-right: 1em !important;
    border-radius: 50%;
}
.card .card-header {
    font-style: italic;
    overflow: hidden;
    width: auto;
}
.card .card-header a {
    font-style: normal;
    font-weight: bold;
    text-decoration: none;
}
.card .card-header a:hover {
    text-decoration: none;
}
.card .card-header .info {
    font-style: normal;
    color: #706f6f;
    font-size: 14px;
    min-width: 0;
    overflow: visible;
    white-space: normal;
}
/* 小屏优化 */
@media (max-width: 768px) {
    .link-navigation {
        grid-template-columns: 1fr;
        gap: 0.8rem;
    }
    .card {
        width: 100%;
        max-width: 100%;
        height: auto;
        min-height: 80px;
    }
    .card:hover {
        background-color: rgba(68, 138, 255, 0.1);
    }
}
</style>
<div class="links-content">
<div class="link-navigation">

<div class="card">
        <img class="ava" src="https://blog.creeperspy.top/_astro/avatar.DJAsEJY-_7ifTj.avif" />
        <div class="card-header">
            <div>
                <a href="https://blog.creeperspy.top/" target="_blank">仿生猫的小窝</a>
            </div>
            <div class="info">A 13-year-old boy with few thoughts but big dreams.</div>
        </div>
</div>

</div>
</div>
</div>

## 添加咱？

如果要申请的话，请在下面的评论区提供头像，站点名称，链接和一句简介，欢迎加链喵，谢谢喵~♡

添加本站友链时，请参考以下信息：

```jsonc
{
  "title": "游离域",
  "url": "https://yulliil.moe/blog/",
  "author": "Yulliil",
  "logo": "https://yulliil.moe/blog/assets/logo.svg",
  // 考虑到本站 logo 的显示效果可能不佳，您可能会使用 avatar。
  "avatar": "https://yulliil.moe/blog/assets/yulliil.jpg",
  "desc": "今日はうまく笑えたかな？"
}
```
