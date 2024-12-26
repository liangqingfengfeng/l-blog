---
title: "advent to code 第六天"
date: 2024-12-26T22:25:05+08:00  
# weight: 1
# aliases: ["/first"]
tags: ["Advent of Code", "游戏"]
# author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: false
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
---

## 前言

[代码的出现](https://adventofcode.com/2024/day/6) 已经过到第六天啦，这关逐渐有意思起来了，这里记录一下我的解法，当然不是最优的解法，第一时间能想到的且最笨的解法。

## 守卫加里凡特

这天的名字叫 Guard Gallivant，要算出守卫在地图中走过的面积，地图如下：
```js
....#.....
.........#
..........
..#.......
.......#..
..........
.#..^.....
........#.
#.........
......#...
```

守卫一开始是朝着箭头的方向开始走，只要遇到 # 就往顺时针旋转 90° 的方向继续走，直到走出地图。

## 解题

矩阵里查找线路，我们很容易想到可以使用深度优先遍历，而且递归的写法更清晰明了，所以一开始我就决定用递归深度优先来解。

当遇到 # 就顺时针旋转 90°，我们可以先定义一个字典来存一下，遇到 # 要转变的下一个方向

```js
const transformPosMap = {
    'up': 'right',
    'right': 'down',
    'down': 'left',
    'left': 'up'
}
```

然后我们开始定义递归函数 dfs，我们用一个 visited 数组来记录守卫走过的路，走过的置为 1，最后我们计算一下 visited 里的 1 的数量就得出这题的答案。

```js
const dfs = (i, j, visited, matrix, rows, cols, pos) => {
    if (i >= rows || i < 0 || j >= cols || j < 0) return;
    if (visited[i][j] === 0) {
        visited[i][j] = 1;
    };

    if (pos === 'up') {
        if (matrix[i - 1] && matrix[i - 1][j] === '#') {
            return dfs(i, j + 1, visited, matrix, rows, cols, transformPosMap[pos]);
        } else {
            return dfs(i - 1, j, visited, matrix, rows, cols, pos);
        }
    } else if (pos === 'right') {
        if (matrix[i][j + 1] === '#') {
            return dfs(i + 1, j, visited, matrix, rows, cols, transformPosMap[pos]);
        } else {
            return dfs(i, j + 1, visited, matrix, rows, cols, pos);
        }
    } else if (pos === 'down') {
        if (matrix[i + 1] && matrix[i + 1][j] === '#') {
            return dfs(i, j - 1, visited, matrix, rows, cols, transformPosMap[pos]);
        } else {
            return dfs(i + 1, j, visited, matrix, rows, cols, pos);
        }
    } else if (pos === 'left') {
        if (matrix[i][j - 1] === '#') {
            return dfs(i - 1, j, visited, matrix, rows, cols, transformPosMap[pos]);
        } else {
            return dfs(i, j - 1, visited, matrix, rows, cols, pos);
        }
    }
}
```

dfs 函数写出来就简单了，我们先找到入口坐标, 记在变量 oi，oj 上，然后调用 dfs 函数，最后遍历 visited 二维数组里 1 的数量就大功告成啦

```js
for (let i = 0; i < matrix.length; i++) {
        let find = false;
        for (let j = 0; j < matrix[i].length; j++) {
            if (matrix[i][j] === '^') {
                oi = i;
                oj = j;
                find = true;
                break;
            }
        }
        if (find) break;
    }
    dfs(oi, oj, visited, matrix, rows, cols, 'up');
    let ans = 0;
    for (let i = 0; i < visited.length; i++) {
        for (let j = 0; j < visited[i].length; ++j) {
            if (visited[i][j] === 1) {
                ans++;
            }
        }
    }
    return ans;
```

大功告成过早了，示例完美运行，谜题输入爆栈了，悲催。

不过递归已经写出来了，整体思路示对的，我们把递归改为循环就可以了

```js
    let i = oi;
    let j = oj;
    let pos = 'up';
    while (i < rows && i >= 0 && j < cols && j >= 0) {
        console.log('i: ', i, 'j: ', j, 'pos: ', pos)
        if (visited[i][j] === 0) {
            visited[i][j] = 1;
        };
    
        if (pos === 'up') {
            if (matrix[i - 1] && matrix[i - 1][j] === '#') {
                j++;
                pos = transformPosMap[pos];
            } else {
                i--;
            }
        } else if (pos === 'right') {
            if (matrix[i][j + 1] === '#') {
                i++;
                pos = transformPosMap[pos];
            } else {
                j++;
            }
        } else if (pos === 'down') {
            if (matrix[i + 1] && matrix[i + 1][j] === '#') {
                j--;
                pos = transformPosMap[pos];
            } else {
                i++;
            }
        } else if (pos === 'left') {
            if (matrix[i][j - 1] === '#') {
                i--;
                pos = transformPosMap[pos];
            } else {
                j--;
            }
        }
    }
```

谜题输入一跑，这次是真的大功告成！

part two 待续。。。