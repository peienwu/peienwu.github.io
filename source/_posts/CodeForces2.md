---
title: 2021.2.5 Codeforces Round #699 (Div.2)
date: 2021-2-5
tags: 
    - CF
    - 競賽
categories:
	- C++基礎主題
	- CF

mathjax: true
---

* Rating change:1197-><font color="#0f0">1201<font color="#f00">(+4)<font color="#000">
* Problem solved: 2

終於上綠了！！這次只有加四（加的分數越來越少，表示要更強才能加更多分），統計一下總共打了六場才升到 rating 1200。

<!--more-->

## Problem 1

### 題序：[點此](https://codeforces.com/contest/1481/problem/A)

![](https://i.imgur.com/TwBSik8.png)

### 解題想法

看完題目花了我大概五分鐘（太慢！），理解題意後發現蠻簡單的，只要分別統計 *U,D,R,L* 的數量即可，判斷  *U,D,R,L*的數量足不足夠到指定的目標。

```cpp=
#include <iostream>
#include <string>
using namespace std;
int t;
 
int main(){
    cin>>t;
    while(t--){
        string s;
        int x,y;cin>>x>>y;
        cin>>s;
        
        int arr[4] = {0};
        long int len = s.length();
        for(int i=0;i<len;i++){
            if(s[i]=='U')arr[0]++;
            else if(s[i]=='D')arr[1]++;
            else if(s[i]=='R')arr[2]++;
            else if(s[i]=='L')arr[3]++;
        }
        if(x>=0 && x<=arr[2] && y>=0 && y<=arr[0]){
            cout<<"YES"<<endl;
        }
        else if(x>=0 && x<=arr[2] && y<=0 && abs(y)<=arr[1]){
            cout<<"YES"<<endl;
        }
        else if(x<=0 && abs(x)<=arr[3]&& y>=0 && y<=arr[0]){
            cout<<"YES"<<endl;
        }
        else if(x<=0 && abs(x)<=arr[3]&& y<=0 && abs(y)<=arr[1]){
            cout<<"YES"<<endl;
        }
        else cout<<"NO"<<endl;
    }
}
```

結果我還是吃了wa，因為沒有 **<=** 寫成 **<**，改完之後（改太快沒有注意到）結果CE，每一次上傳都要自己先編譯過啊，才不會犯這種低級錯誤！

### 解題紀錄

這一題用了30分鐘才寫出來，同學用了大概17分鐘就解出來了。這大概是我第一次體會到 coding 的速度跟不上腦袋想的速度（力不從心），明明已經想到要怎麼解，卻寫得很慢。所以看題目的時間＋手速很慢＝得分很少。
寫程式的速度需要時間慢慢培養，首先要做的就是加快打字的速度！
除此之外，還要避免犯下低級錯誤，務必要在上傳之前先檢查一下!

## Problem 2

### 題序：[點此](https://codeforces.com/contest/1481/problem/B)

![](https://i.imgur.com/VtQIJx3.png)

### 解題想法

直覺想到就是模擬第**1**顆球到**k-1**顆球，就可以推得第k顆球的位置

```cpp=
#include <iostream>
using namespace std;
int t;
 
int main(){
    cin>>t;
    while(t--){
        int n,k;cin>>n>>k;
        int arr[n];
        for(int i=0;i<n;i++)cin>>arr[i];
        
        while(--k){
            int ind = 0;
            while(arr[ind]>=arr[ind+1] && ind<n-1)ind++;
            if(ind!=n-1)arr[ind]++;
            else break;
        }
        int i=0;
        while(arr[i]>=arr[i+1] && i<n-1)i++;
        if(i!=n-1)cout<<i+1<<endl;
        else cout<<-1<<endl;
    }
}
```

### 解題紀錄

寫到一半看到k的的大小 **10^9^**，還以為**O(n)**的複雜度會tle，不過我寫完丟上去測，過了！（我還跟同學說這個問題，結果他就去想有沒有其他O(n)以下的解法，後來他就沒有寫出來）不過花了28分鐘寫出來之後，看了範圍限制想了想，有一個關鍵的條件：
$$1≤𝑛≤100, 1≤ℎ𝑖≤100$$
**the sum of 𝑛 over all test cases does not exceed 100.**
也就代表卡在中間的球數不會超過$(𝑛−1)⋅(100−1)$個，也就是說，最多從第$(𝑛−1)⋅(100−1)+1$ 個開始，接下來每一顆球都會一路望下滑，不會卡在中間，這時候迴圈就可以break了。看來 **10^9^**只是拿來嚇人的，正常模擬是不會tle的（某同學丟了3次還是wa）。

## 心得

![](https://i.imgur.com/xaEBkHZ.png)
這是目前的rating，昨天+4 之後終於變成綠色了！
![](https://i.imgur.com/cgOkxrJ.png)
![](https://i.imgur.com/5BtUq4I.png)
這個是目標
{% note default%}
希望接下來可以順利參加資訊之芽，好好讓自已變強，學一些dp還有其他的演算法，把學長的講義看完然後學起來，在高二的時候可以打比賽！
{% endnote %}
