---
title: item高度动态变化的RecyclerView 
date: 2022-11-20 11:02:44
categories: Android
---

## 首先来看一下效果图

<div style="text-align: center; display: flex; justify-content: space-around;">
    <div>
        <img src="/images/5125944-77e025a2d814f062.gif" width="256" height="256">
        <div>linearLayoutManager</div>
    </div>
    <div>
        <img src="/images/5125944-a03770304bb1a3f1.gif" width="256" height="256">
        <div>gridLayoutManager</div>
    </div>
</div>
<br>


首先可以确定的是每个item肯定是有最大高度和最小高度的，所以我们先要给定item的目标高度（targetHeight）和初始高度（defaultHeight）。
</br>
就以LinearLayoutManager和targetHeight大于defaultHeight为例，都是第一排的item往上滑动的时候item高度是减小的，但是第二排的item的高度是增加的，很明显的能看出当我们滑动的时候，列表滑动是很流畅的，不难看出第一排item高度的减小和第二排item高度的增加是守恒的，也就是说这两个高度相加是一直等于targetHeight+defaultHeight的，所有我们只需要确定了第一个item的高度那么第二个item的高度就等于targetHeight+defaultHeight-第一个item的高度。
</br>
那么第一个item的高度要怎么确定呢，理论上这个top绝对值的的最大值是defaultheight,因为当第一个item快要完全划出去的时候我们通过getTop获取到的高度就应该是defaultHeight，而且第一个item就应该根据top的变化来动态的改变自己的高度，而top绝对值变化的范围就是在targetHeight和defaultHeight之间的，那么根据高度的变化就等于(top \* 1f / defaultHeight) * (targetHeight - defaultHeight)，再通过和targetHeight求和就得到了第一个item的高度了，可以看出第一个item高度的范围，当top为0的时候是targetHeight，当top为-targetHeight的时候（不难看出第一个item的top值一直是小于等于0的）是defaultHeight。

```
View childAt0 = getChildAt(0);
ViewGroup.LayoutParams layoutParams0 = childAt0.getLayoutParams();
int top = childAt0.getTop();
top = top <= -defaultHeight ? -defaultHeight : top;
layoutParams0.height = (int) (targetHeight + (top * 1f / defaultHeight) * (targetHeight - defaultHeight));
childAt0.requestLayout();
```

那么第二个item的高度确定的方法上面也说了，就是等于总高度减去第一个item的高度。

```
View childAt1 = getChildAt(1);
ViewGroup.LayoutParams layoutParams1 = childAt1.getLayoutParams();
layoutParams1.height = defaultHeight + targetHeight - getChildAt(0).getLayoutParams().height;
childAt1.requestLayout();
```

接下来的item高度就好确定了，都是defaultHeight。

```
View childAt = getChildAt(i);
ViewGroup.LayoutParams layoutParams = childAt.getLayoutParams();
layoutParams.height = defaultHeight;
childAt.requestLayout();
```

## 还存在的问题

如果用户滑动速度比较慢，那么top的绝对值一直是小于等于defaultHeight的，但是当用户快速滑动的时候top的绝对值就会出现比defaultHeight大的情况，这个bug目前还不知道是怎么产生的（如果有大佬知道，希望大佬给点意见）,所以目前先在第一个item确定高度的地方加上了

```
top = top <= -defaultHeight ? -defaultHeight : top;
```

这个判断避免出现top绝对值大于defaultHeight 的情况。

<a href="https://github.com/MonkZl/FreeStyleRecyclerViewDemo">github源代码链接</a>
