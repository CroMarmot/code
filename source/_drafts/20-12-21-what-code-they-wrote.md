---
title: 前人留下的生产代码
date: 2020-12-21 11:20:14
tags: code
category: [code,frontend,backend,golang,js,c++]
---

# 总

看这些代码你可能觉得是故意“编造”出来的代码，初学者才会写的代码。

然而这些代码，是在一个一个生产代码中找到的，他们均来自于技术组的核心成员，没有一个是实习生写的，提交历史短的3个月，长的也是一两年。推荐`git blame`命令是真的好用。

# 易读性

## 背景

`A*` 寻路, 原始代码也不是C++。下面用C++非`A*`的简单bfs寻路 示例

## 重现示例代码

去掉了一些错误传入的判断示例

```c++
#include <bits/stdc++.h>
using namespace std;

vector<pair<int,int> > get_path(const vector<vector<int> >& maze,const pair<int,int>& st_pos, const pair<int,int>& end_pos ) {
  // 去掉了边界校验
  if(st_pos == end_pos)return {};
  if(maze[st_pos.first][st_pos.second] == 1 || maze[end_pos.first][end_pos.second] == 1 )return {};
  vector<pair<int,int> > bfs;
  vector<vector<int> > distance(maze.size(),vector<int>(maze[0].size(),0));
  vector<vector<pair<int,int> > > fa(maze.size(),vector<pair<int,int>>(maze[0].size(),{0,0}));
  distance[st_pos.first][st_pos.second] = 1;
  bfs.push_back(st_pos);
  int itr = 0;
  while(itr < bfs.size()){
    auto head = bfs[itr];
    if(head == end_pos){
      vector<pair<int,int> > path;
      auto ptr = head;
      path.push_back(ptr);
      while(distance[ptr.first][ptr.second] > 1){
        ptr = fa[ptr.first][ptr.second];
        path.push_back(ptr);
      }
      reverse(path.begin(),path.end());
      return path;
    }
    auto [i,j] = head;
    {
      if(j+1 < maze[0].size()){
        if(!(distance[i][j+1] > 0 || maze[i][j+1] == 1)){
          fa[i][j+1] = head;
          distance[i][j+1] = distance[i][j] + 1;
          bfs.push_back({i,j+1});
        }
      }
    }
    {
      if(j-1 >= 0){
        if(!(distance[i][j-1] > 0 || maze[i][j-1] == 1)){
          fa[i][j-1] = head;
          distance[i][j-1] = distance[i][j] + 1;
          bfs.push_back({i,j-1});
        }
      }
    }
    {
      if(i+1 < maze.size()){
        if(!(distance[i+1][j] > 0 || maze[i+1][j] == 1)){
        fa[i+1][j] = head;
        distance[i+1][j] = distance[i][j] + 1;
        bfs.push_back({i+1,j});
        }
      }
    }
    {
      if(i-1 >= 0){
        if(!(distance[i-1][j] > 0 || maze[i-1][j] == 1)){
          fa[i-1][j] = head;
          distance[i-1][j] = distance[i][j] + 1;
          bfs.push_back({i-1,j});
        }
      }
    }
    itr++;
  }
  return {};
}

int main(){
  vector<vector<int>>maze = {
    {0,1,0,0,0},
    {0,1,0,1,0},
    {0,1,0,1,0},
    {0,1,0,1,0},
    {0,1,0,1,0},
    {0,1,0,1,0},
    {0,1,0,1,0},
    {0,0,0,1,0},
  };
  pair<int,int> st_pos = {0,0};
  pair<int,int> end_pos = {7,4};
  auto path = get_path(maze,st_pos,end_pos);
  for(auto item:path){
    printf("(%d %d)\n",item.first,item.second);
  }
  return 0;
}
```

我仿佛回忆起了还没学过 for和数组的使用技巧时，曾经用VB写俄罗斯方块判断下落的一个又一个if

看着4块代码，我不知道如果有一天改成8邻会变什么样，或者用了六边形六邻会变怎么样。

## 改造

其中4块部分，其实就是当前位置的4邻块，以及边界判断，阻碍判断

```c++
const int K = 4;
int deri[K] = {0,0,-1,1};
int derj[K] = {-1,1,0,0};
auto [i,j] = head;
for(int k=0;k<K;k++){
  int newi = i + deri[k];
  int newj = j + derj[k];
  if(newi < 0 || newj <0 || newi >= maze.size() || newj >= maze[0].size())continue;
  if(distance[newi][newj] > 0 || maze[newi][newj] == 1)continue;
  fa[newi][newj] = head;
  distance[newi][newj] = distance[i][j] + 1;
  bfs.push_back({newi,newj});
}
```

## 总结

还好这部分代码本身只有易读性的问题，根据也似乎不会有改成8邻的需求，再加上原始代码还融合了很多其它条件加到判断，整个复用性几乎没有。如果编译器不够聪明，甚至可以说手动优化了编译后的判断次数，:-) 开心就好。

总体来说，见到有重复可提取部分还是要有提取的习惯。

# 坐标

## 背景

不知道什么原因没有使用struct表示x,y，而是采用编码解码的形式来记录

## 重现示例代码

```go
package main

import (
  "fmt"
)

func point(x int, y int) int {
  return x*100+y;
}

func xy(p int) (int,int) {
  return p/100,p%100;
}

func main() {
  a,b := xy(point(3,4))
  fmt.Printf("%d %d\n",a,b)
}
```

## 改造

这一段看似没有任何问题，的确在我们的使用中也的确 坐标范围都在`[0,10]`

然而实习生写代码，在计算两点经过的直线上的点时，计算错了，计算出了超出界限的坐标，有的坐标为负数，这段代码却从来没有报错，持续放置了4个多月，多轮测试后，才发现有问题。

改造方法：增加了输出日志报错，不让已有的程序崩掉（没有throw Error），报错一个解决一个。

## 总结

很明显多加判断，就多加了运行时的执行代码数量，尤其是在这种本身就短，复用又高的代码中，耗时比例会相对明显。

感觉能用struct还是struct吧。没实际测过加了判断影响多少耗时，但加了判断，确定了调用者错误调用的位置，所以还是每层都加校验的减少错误代价 来换 运行时间代价？

# 锁开闭

TODO

# if逻辑的内外嵌套问题

TODO

# 分层逻辑

嘲讽逻辑，两个不同层级都在处理，两次取反，非幂等 TODO

# 数组合并

## 背景

有A方提供完整的列表和所有元素相关的信息，B方记录要展示的id

## 重现代码


```c++
#include <bits/stdc++.h>
using namespace std;

struct DemoStruct{
  int data;
  int id;
};

template<typename T>
vector<T> item_list(vector<T>info, vector<int> id ){
  vector<T> result ;
  for(int i = 0;i<info.size();i++){
    for(int j = 0;j < id.size();j++){
      if(info[i].id == id[j]){
        result.push_back(info[i]);
        break;
      }
    }
  }
  return result;
}

int main(){
  vector<DemoStruct>info = {{100,0},{101,1},{102,2},{103,3},{104,4},{105,5}};
  vector<int>id = {1,4,7};
  auto l = item_list(info,id);
  for(auto item:l){
    printf("%d\n",item.data);
  }
  return 0;
}
```

## 改造

用随手可即的js写一段类似的性能测试

```js
function includesPerf(N){
  const start = performance.now();
  const arr0 = [];
  for(let i =0;i<N;i++){
    arr0.push(i);
  }
  let ans = 0;
  for(let j = 0;j<N;j++){
    ans+= arr0.includes(j);
  }
  console.log(ans);
  console.log(performance.now()-start);
}


function setPerf(N){
  const start = performance.now();
  const arr0 = {};
  for(let i =0;i<N;i++){
    arr0[i] = 1;
  }
  let ans = 0;
  for(let j = 0;j<N;j++){
    ans+= arr0[j] == 1;
  }
  console.log(ans);
  console.log(performance.now()-start)
}
let N = 100000;
includesPerf(N);
setPerf(N);
```

或者稍微有点复杂度了解，就知道`N^2` 和`N log(N)`的区别, (假定 set 是log N 性能)

# 全部选择标识

## 背景

有一个全部选择的按钮


```js
let all_checked = false;

button.onClick(()=>{
  all_checked = true;
});
```

# resultful API

`http://mappcenter.mpaas.com/webapi/mappcenter/mds/nebula/changeTaskStatus?.....` 方法是GET

# 微信小程序QrCode

扫描成功的errMsg里返回的是`scanCode:ok`

## 修复

如果没有记错，在学编程前几节课就说"如果一个变量没有被赋值，它就保持原来的值"。

这问题在，取消全选的时候，并没有修改`all_checked`为false，而这个问题竟然上线了几个月，才被发现。

## 总结

不过即使是生产，只要用户对应的数据量还不够大，所有用户调用这个查询的总频率还不够高，它就是“合理”的代码。

你们真的做了code review吗，有多少正常运行，但是还没到这种“瓶颈”的代码在隐藏着，有这么些代码的存在，像上面，那样坐标加的错误判断所带来的性能损失又算什么呢？记住code review，不是“bug代码/已有代码”，而是“code review 发现的bug代码/ 所有 bug 代码”。

有时我在想，面试那么多动态规划，了解了select/epoll，却在这种代码上写成这样，那何必呢。

对于用户来说，真的是能跑就行，Done is better than perfect，并且就再也不优化的现状，写好代码，真的有意义吗，还是要学会多累积“名词”学习，这些细节，并不会带来绩效，而那些“名词”会。即使做出来的东西，需要频繁的手动修复数据，甚至能产生多的工作量，看起来在加班。

# 参考

历史项目中所见到的生产代码

# 总

尽量持续更新

感觉面试和工作内容总结，组员评价应该构成环。

面试中我尽量去掉那种1分钟内，就搜索一下就出来答案的问题，也去掉了那些写一次封装以后就再也不用关心的问题。

尽量从历史代码中出现的问题，抽出一个相对复杂的任务，让面试者自己去拆分设计。

面试算法和设计思想，是一种偷懒大多时候有效的方法，有时变成背诵题目和面经，我没在实际工作中用到dp，但却见到了if层级写不好的，该用else if 条件判断而省略成了else的

~~原来标题想叫《那些问“复杂”面试题的面试官写了些什么生产代码》~~
