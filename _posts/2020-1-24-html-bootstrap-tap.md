---
layout: post
title: Bootstrap之选项卡
categories: [前端]
tags: [Bootstrap]
date: 2020-1-24 13:48:07
comments: true
---

#### 概述

Bootstrap选项卡主要有两部分内容组成：
1. 选项卡菜单组件，对应的是Bootstrap的nav-tabs
2. 可以切换选项卡面板组件，在Bootstrap通常tab-pane来表示

在Bootstrap框架中选项卡nav-tabs已带有样式，而对于面板内容tab-pane都是隐藏的，只有当前面板内容才是显示的

1、选项卡导航链接中要设置 data-toggle="tab"

2、并且设置 data-target="对应内容面板的选择符(一般是ID)"；如果是链接的话，还可以通过 href="对应内容面板的选择符(一般是ID)"，主要作用是用户点击的时候能找到该选择符所对应的面板内容 tab-pane。

3、面板内容统一放在 tab-content 容器中，而且每个内容面板 tab-pane 都需要设置一个独立的选择符（最好是ID）与选项卡中的 data-target 或 href 的值匹配

参考链接：https://www.cnblogs.com/xiaohuochai/p/7144028.html

#### 参考代码


```html
<div class="card ">
        <div class="card-header ui-sortable-handle border-0">
            <h3 class="card-title">
              用户趋势
            </h3>
            <div class="card-tools">
              <ul class="nav nav-pills ml-auto">
                <li class="nav-item">
                  <a class="nav-link active" href="#new-users" data-toggle="tab">新增用户</a>
                </li>
                <li class="nav-item">
                  <a class="nav-link" href="#active-users" data-toggle="tab">日活用户</a>
                </li>
                <li class="nav-item">
                <a class="nav-link" href="#all-users" data-toggle="tab">累计用户</a>
                </li>
              </ul>
            </div>
        </div>
        <div class="card-body">
            <div class=" tab-content p-0">
                <div class="chart tab-pane fade show active" id="new-users">
                    新增用户
                </div>
                <div class="chart tab-pane fade" id="active-users">
                    日活用户
                </div>
                <div class="chart tab-pane fade" id="all-users">
                    累计用户
                </div>
            </div>
        </div>
    </div>
```

【渐入效果】
　　为了让面板的隐藏与显示在切换的过程效果更流畅，可以在面板中添加类名 fade，让其产生渐入的效果。在添加 fade 样式时，最初的默认显示的内容面板一定要加上 in 类名，不然用户无法看到其内容。









 





















