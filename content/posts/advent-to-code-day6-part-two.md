---
title: "advent to code 第六天 Part Two"
date: 2025-01-15T16:56:16+08:00  
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

继续来闯关，第二小问还卡了我挺长时间，少了一个小判断。。。

## 题目

```
....#.....
....+---+#
....|...|.
..#.|...|.
....|..#|.
....|...|.
.#.O^---+.
........#.
#.........
......#...
```

这一问是要在地图上放置障碍物，使得守卫进入循环，避免发现我们，比如下面在(6,3)处放置一个障碍物，就会导致守卫循环得走一条路，需要我们找出有多少个位置放上障碍物会导致守卫进入循环。


## 思路

以前还真没有做过如何判断矩阵中循环路径，一开始没什么思路，后面想到，守卫走过的步数是小于整个矩阵的大小的，如果守卫走的步数巨大，那不就说明守卫进入了循环吗？

根据这个思路我们很快就可以改好第一版代码

## 解法

我们只需在 while循环中加一个变量来记录守卫走过的步数，当超过一定的数值就退出循环，我这里设置的是矩阵大小的 10 倍。

```javascript
function checked(matrix, oi, oj) {
    const rows = matrix.length;
    const cols = matrix[0].length;
    let i = oi;
    let j = oj;
    let pos = 'up';
    let step = 0;
    while (i < rows && i >= 0 && j < cols && j >= 0) {
        step++;
        if (step > cols * rows * 10) {
            return 1;
        }
    
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
    return 0;
}
```

其实我们只需要在守卫走过的路径上放障碍物，可以节省一些判断，因为在守卫没走过的地方放障碍物不会影响到守卫的路径。

```javascript
let ans = 0;
for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; ++j) {
        if (visited[i][j] === 1 && matrix[i][j] === '.') {
            count++;
            matrix[i][j] = '#';
            if (checked(matrix, oi, oj)) {
                ans++;
            }
            matrix[i][j] = '.';
        }
    }
}
```

很遗憾，得出的答案是 1825，比正确答案要高，原因是我们少判断了一种情况


<img src="/images/exp1.png" alt="" style="width: 300px; height: auto;">

例如上面这种情况，往红色箭头的地方放一个障碍物，当守卫往上走，遇到障碍物的时候，会右转，但是右转也是障碍物，这时就需要再右转 90°，缺少了这个判断。只需要多判断一次就可以了，因为连续右转两次就是 180° 转弯，转回来的路了。

我们改进一下点，多加一个判断，四个方向都要加

```javascript
function checked(matrix, oi, oj) {
    const rows = matrix.length;
    const cols = matrix[0].length;
    let i = oi;
    let j = oj;
    let pos = 'up';
    let step = 0;
    while (i < rows && i >= 0 && j < cols && j >= 0) {
        step++;
        if (step > cols * rows * 10) {
            return 1;
        }
        if (pos === 'up') {
            if (matrix[i - 1] && matrix[i - 1][j] === '#') {
                if (matrix[i][j + 1] === '#') {
                    i++;
                    pos = revertPosMap[pos];
                } else {
                    j++;
                    pos = transformPosMap[pos];
                }
            } else {
                i--;
            }
        } else if (pos === 'right') {
            if (matrix[i][j + 1] === '#') {
                if (matrix[i + 1] && matrix[i + 1][j] === '#') {
                    j--;
                    pos = revertPosMap[pos];
                } else {
                    i++;
                    pos = transformPosMap[pos];
                }
            } else {
                j++;
            }
        } else if (pos === 'down') {
            if (matrix[i + 1] && matrix[i + 1][j] === '#') {
                if (matrix[i][j - 1] === '#') {
                    i--;
                    pos = revertPosMap[pos];
                } else {
                    j--;
                    pos = transformPosMap[pos];
                }
            } else {
                i++;
            }
        } else if (pos === 'left') {
            if (matrix[i][j - 1] === '#') {
                if (matrix[i - 1] && matrix[i - 1][j] === '#') {
                    j++;
                    pos = revertPosMap[pos];
                } else {
                    i--;
                    pos = transformPosMap[pos];
                }
            } else {
                j--;
            }
        } 
    }
    return 0;
}
```

完美解决

## 后记

真的想了挺多办法的，卡住的时候挺无奈，没想到是少了一个判断，新的一年已经到来，没有总结的习惯，也没有计划的习惯，看看要不要做个计划吧