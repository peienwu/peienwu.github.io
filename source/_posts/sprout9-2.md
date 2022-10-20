---
title: "[題解]NEOJ 187 高棕櫚觀光農場：旅行銷售員問題TSP"
date: 2021-1-10
tags: 
    - 資訊之芽
    - 題解
categories: 資芽題解
mathjax: true
---

### 高棕櫚觀光農場：旅行銷售員問題TSP
<!--more-->
[題目連結](https://neoj.sprout.tw/problem/187/)
第一次寫**TSP**(Traveling Salesman Problem)，題目敘述如下：
> 給定一系列城市和每對城市之間的距離，求解存取每一座城市一次並回到起始城市的最短迴路。

這是一個已經被證明$NP-Hard$ 的問題，暴力作法是要檢查所有路徑的情況，因為共有n個點，每一個點又連接n-1個點，繼續下去總共會有N!種情況，因此複雜度為$O(n!)$。
總共的點數共有n個，根據多邊形的邊數公式可以知道一共有$\frac{n^2-n}{2}$條邊，每一條邊都有各自的距離。
如果改用DP做，時間複雜度可以壓到時間複雜度$O(n^2\cdot2^n)$，比$O(n!)$還優秀！這一題用到的是DP優化中的<font color="#f00">狀態壓縮</font>，具體的實作細節如下：

{% note success %}

**位元運算**
在這一題需要用到位元運算，用16個bit表示每一個點有沒有被走訪過。用這樣的表示方法可以讓code更為節儉，也就是狀態壓縮的概念。

**左移運算子(<<)**
這一題會一直反覆被用到，1<<t代表把1往左移動t單位，用10進位表示就是$2^t$。每往左移動一格，數字就會變成原來的兩倍！

**狀態壓縮**
將十進位整數s以二進位表示，會得到一串01字串，假設s=10，則 $s = 1010_{(2)}$。以這題來說，這樣的字串我們可以用來表示第4和第2個點已經被拜訪，而第3跟第1個點還沒有被拜訪。
{% endnote %}

#### 定義

定義 $dp[n][s]$ 表示目前在第n點上，s為走過的點（狀態壓縮）的最短距離

#### 轉移式

$dp[n][s] = min(dp[i][s-(1<<n)]+dis[i][n])$, for all i such that s&(1<<i)!=0

這個轉移式很有意思，因為s代表了每一個點是否有被走過，當我要更新dp[n][s]時，我要確保此時的狀態s中的點n必須為1。同時，因為1<<i用二進位表示只有第i位會是1其他都是0，做and運算就看s的第i位決定結果。

對於每一個i必須確保第i點在狀態s中有被造訪，因此有了後面的條件。另外轉移式中的s-(1<<N)是把第n個點從未造訪的狀態轉移。

#### 邊界

$dp[0][1] = 0,\quad dp[i][j]= \infty$

距離的預設狀態為無限大，第一個點的初始狀態是最短距離0

#### 實作小細節

**迴圈順序**
在轉移的過程中，迴圈的第一層必須是狀態，第二層才是城市。如果顛倒過來的話，會導致前面的城市在狀態還沒有被更新的時候就已經失去了之後被更新的機會，因此城市的迴圈必須放在第二層！

**求答案**
因為我們要求的是回到原點的最短距離，因此在全部轉移完成之後，利用一個迴圈把回去原點的路的距離加上去，求得最小值。

```cpp=
#include <bits/stdc++.h>
#define N 16
#define mod 1000000007
#define FOR(i,n) for(int i=0;i<n;i++)
#define ios ios::sync_with_stdio(0)
using namespace std;
int t;

void solve(){
    int n,dis[20][20];cin>>n;
    memset(dis,0,sizeof(dis));
    
    for(int i=0;i<n-1;i++){
        for(int j=i+1;j<n;j++){
            cin>>dis[i][j];
            dis[j][i] = dis[i][j];
        }
    }
    
    int dp[N][66000],m = 1<<n;
    memset(dp,0x3f3f3f3f,sizeof(dp));
    dp[0][1] = 0;   //將0點到自己的距離設為1
    
    //O(n^2 2^n)
    for(int i=0;i<m;i++){            //i表示2^n每一種狀態
        for(int j=0;j<n;j++){        //j表示城市
            if(!(i&(1<<j)))continue; //j城市要是被造訪過的狀態
            for(int k=0;k<n;k++){    //可以到的所有點中的狀態
                if(i & 1<<k){        //確保轉移過去的有被造訪過
                    dp[j][i] = min(dp[j][i],dp[k][i-(1<<j)]+dis[j][k]);
                }
            }
        }
    }
    int ans = INT_MAX;
    for(int i=0;i<n;i++)ans = min(ans,dp[i][m-1]+dis[i][0]);
    cout<<ans<<endl;
}

int main(){
    ios;
    cin>>t;
    while(t--){
        solve();
    }
}
```