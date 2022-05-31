---
title: 資芽9-13週：圖論、隨機、根號算法
date: 2021-5-8
tags: 
    - 資芽筆記
    - 動態規劃
    - DP
categories: 資訊之芽筆記
mathjax: true
#password: sprout2022
---

# 資芽第九週：動態規劃（三）

## 上課內容

首先是影片的重點整理，主題是DP優化：

DP優化

- 矩陣快速冪優化
  - 費氏數列：O(n)->O(logN)
  - DC
  - 把很大的n做矩陣快速冪降到logN
<!--more-->
- 狀態壓縮
  - TSP旅行銷售員問題
  - 權重最小的漢米頓迴路
  - 暴力做：n個點、n-1點、...、1個點
  - 暴力複雜度：O(N!)
  - 使用位元紀錄
  - DP[n][s]用s的二進位第i位表示是否去過該城市
  - 複雜度：轉移$O(N)\times O(N\times 2^N)=O(n^2\times 2^n)$

- 資料結構優化（單調隊列優化）
  - 開一個新的dpMAX，O(n^2)->O(n)
  - 不大於K，用heap來O(nlogn)維護遞增的固定區間最大值
  - 方法三：用Deque實作，O(n)
  - I<j，dp[i]>dp[j] 具有單調性（單調遞減）

- 円円開店
  - dp[n] = val[n]+max(dp[i]-c(n-i)),n-k≤i≤n-1
  - O(NK)->O(N)使用單調隊列優化
  - 令t[I] = dp[i]+c*i

- 有限背包問題優化
  - 分推湊出每一種重量
  - 如何分推：二進位 ${1,2,4,8,...,2^p,q}$，使 $2^{p+1}-1$ 不大於k[i]的最大整數
  - 原O(NWK) -> O(NW*logK)
  - 利用單調隊列優化：O(NW)!!!

## 上機作業

### 円円賣漢堡

[題目連結](https://neoj.sprout.tw/problem/188/)
首先在做任何優化之前，都必須先列出定義與轉移式。

#### 定義

定義$dp[i]$為在第1到i個點開店，且第i個點有開店時的最大盈利

#### 轉移式

$dp[i] = val[i]+max(dp[j]-c(i-j)),i-k≤j≤i-1$

#### 邊界

$dp[i] = 0, 1≤i≤n$

做完以上的定義與轉移式之後，可以得到複雜度：$O(NK)$就可以得到以下的結果：
![](https://i.imgur.com/Z4OaqJJ.png)
試著利用**單調隊列優化**，把複雜度降到$O(N)$
利用單調隊列可以確保所有的數字都只會被push與pop一次，因此總複雜度為$O(n)$
![](https://i.imgur.com/hn2Px6s.png)
差一個K時間就差很多！

具體的作法如下，因為我們要維護的單調隊列的dp後面有一些東西，因此我們要做一些調整，把有i的提出來（不然當i不一樣的時候就很難處理）

$$\begin{split}dp[i] &= val[i]+max(dp[j]-c(i-j)),i-k≤j≤i-1\\&=val[i]-ci+max(dp[j]+cj)\end{split}$$

這時候令 $t[j] = dp[j]+cj$，可以把加上 $cj$ 想像成跟頭的距離（跟當前的 $-ci$ 合在一起就是我們要的距離），則 $max(t[j]),i-k≤j≤i-1$ 就可以使用單調隊列優化！

{% note info %}
**單調隊列優化**

單調隊列優化名稱的由來是因為維護的容器具有單調性（在這裡用到的是單調遞減），這一次嘗試的是用deque<pair<int,int>>來實作，第一維是裝t[i]，第二維是裝i。以下是具體的實作步驟：

1. 將單調隊列中<font color="#f00">過時的</font>（也就是距離大於K的）pop出來，讓容器中的元素都是可以被取到的狀態
2. 更新 $dp[n]$ 的值，利用容器最前面的元素（就是最大的by單調性）來更新。要注意到，元素最前面固然是最大，但也可能會比什麼都不取還差，要特別注意！
3. 從容器的後面更新，把小於接下來要放入的值的元素都pop出來，以維持單調遞減的特性！
4. 將當前的 $t[n]$ 給push進去！結束！

如果要維護dp[n]的單調性，會保證若 $i<j$，則 $dp[i]>dp[j]$；因為如果$dp[i]≤dp[j]$，那第i個永遠不會被取到（因為要求的區間是不斷往後的，若一個數字在前面又比後面的數字小，永遠就不會被取到！）
{% endnote %}

以下是程式碼的部分，其中第21到24行為單調隊列優化的部分：

```cpp=
#include <bits/stdc++.h>
#define int ll
#define ll long long
#define mod 1000000007
#define FOR(i,n) for(int i=0;i<n;i++)
#define ios ios::sync_with_stdio(0)
using namespace std;
int t;

void solve(){
    int n,k,c,val[100005];cin>>n>>k>>c;
    for(int i=1;i<=n;i++)cin>>val[i];
    
    int dp[100005];
    memset(dp,0,sizeof(dp));
    deque<pair<int,int>> deq;
    
    deq.push_back(make_pair(val[1]+c,1));    //這個小細節差點沒有注意到，要加上c
    dp[1] = val[1];
    for(int i=2;i<=n;i++){
        while(!deq.empty() && deq.front().second < i-k)deq.pop_front();
        dp[i] = val[i]-c*i+max(c*i,deq.front().first);
        while(!deq.empty() && deq.back().first <= dp[i]+c*i)deq.pop_back();
        deq.push_back(make_pair(dp[i]+c*i,i));
    }
    int ans = 0;
    for(int i=1;i<=n;i++)ans = max(ans,dp[i]);
    cout<<ans<<endl;
}

signed main(){
    ios;
    cin>>t;
    while(t--){
        solve();
    }
}
```

### 高棕櫚觀光農場：旅行銷售員問題TSP

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

### 憤怒的小鳥

[題目連結](https://neoj.sprout.tw/problem/255/)
[Submission](https://neoj.sprout.tw/challenge/177536/)
玩憤怒鳥遊戲的概念，給定二維平面上n個點代表n個豬（皆在第一象限），這時候可以從原點(0,0)的位置發射軌跡為拋物線的小鳥，打到豬豬後並不會改變飛行的軌跡，題目想要問最少要發射幾個小鳥才能射下所有的豬豬？

> 範例測資
> 7
> 1 3
> 2 4
> 3 3
> 2 6
> 4 8
> 1 1
> 1 2

如果畫在圖上就長這樣：
![](https://i.imgur.com/cCEHouc.png)

如果把拋物線畫出來就像這樣：
![](https://i.imgur.com/u63WTC6.png)

因為$n\le24$，所以可以知道這一題可能只能把所有情況都掃一遍（~~才不是因為剛好講狀態壓縮XD~~）

**作法1：** 時間複雜度$O(n^2\cdot 2^n)$
我們可以利用一個二維陣列g[i][j]表示經過i豬和j豬與原點所形成的拋物線所能經過的所有點的狀態。這樣我們可以利用$O(n^3)$的時間完成預處理，知道一條由i豬和j豬形成的拋物線能經過的點的狀態（用二進位表示），在dp轉移的時候則需枚舉所有的i和j，所需時間是$n^2$。
轉移式的話一開始是寫用推的：$dp[s|g[i][j]] = min(dp[s]+1)$，比較直覺一點。

**作法2：** 時間複雜度$O(n\cdot 2^n)$
當我們在做狀態s的時候，如果是上面的方法就是$n^2$掃過所有的點的組合，其實我們可以只要找到s其中一位第k位是1的，因為那一位所對應到的豬豬一定會經過，所以就可以針對那一位做轉移，如此一來跟k同在一個拋物線的最多只會有n個，複雜度就降到了$O(n\cdot 2^n)$。這裡可以搭配一個vector<int>line[N]裝每一位有經過的可能的狀態。

#### 定義

$dp[s]$為狀態為s的時候的最小拋物線數量

#### 轉移式

$dp[s]=min(dp[s\&(\sim mask)]+1)$

這個轉移式挺有趣的，我們知道如果要從狀態p得到狀態s，對於s是0的bit，p也應該是0；對於s是1的bit，若mask是1，則p的bit要是0，反之則是1。

```flow
st=>start: for each bit p in s
sub1=>subroutine: My Subroutine
cond=>condition: if p=1
e=>operation: set dp[i] to 0
io=>condition: if mask[i]=1
ss=>operation: set dp[i] to 1
sp1=>operation: set dp[i] to 0
sp2=>operation: set dp[i] to 1

st->cond->io
cond(yes)->io->e
cond(no)->e

io(yes)->sp1
io(no)->sp2
```

這個轉移式可以有另外一個表示的方式，可以達到同樣的效果：
$dp[s]=min(dp[s\&(i\oplus mask)]+1)$

#### 邊界

$for\ each\ dp[i]=0$

以下程式碼是以$O(n^3)$的預處理，$O(n\cdot 2^n)$的轉移，因此總時間複雜度是$O(2^n)$，在時限內是可以過的！
![](https://i.imgur.com/j8uRDfj.png)

> [time=Thu, Sep 16, 2021 3:03 PM]更新

結果呢，有人問了我這一題，我回去看我的code，結果發現明明應該是 $O(n\cdot 2^n)$ 轉移，但每一個line_mask中在worst case卻有 $n^2$ 個狀態，這樣子下來一樣還是 $O(n^2\cdot 2^n)$ 沒有進步。發現問題在於，當全部的點都在同一條拋物線上的時候，會有重複的狀態被push很多次的問題，所以只要用visit去紀錄這種狀態是否出現過，讓最多被推入一次就好!

![](https://i.imgur.com/V2ubK6i.png)

```cpp=
#include <bits/stdc++.h>
#define long long ll
#define N 24
#define mod 1000000007
#define FOR(i,n) for(int i=0;i<n;i++)
using namespace std;
int t,n,x[N],y[N];

void solve(){
    vector<int>line_mask[N];
    scanf("%d",&n);
    for(int i = 0;i < n;i++)scanf("%d %d",&x[i],&y[i]);
    int m = 1<<n,dp[m];
    bool visit[m];memset(visit,0,sizeof(visit));
    //calculate all lines O(n^3)
    for(int i=0;i<n;i++){
        line_mask[i].push_back(1<<i);       //有可能此點孤立一人，要確保也在裡面
        for(int j=i+1;j<n;j++){
            int x1 = x[i],y1 = y[i];
            int x2 = x[j],y2 = y[j];
            if(x1==x2)continue;

            double a = (double)((x1*y2)-(x2*y1))/(x1*x2*(x2-x1));
            double b = (double)(y1-a*x1*x1)/x1;
            if(a >= 0)continue;

            int mask1 = 1<<i|1<<j;
            for(int k = 0;k < n;k++){
                if(fabs((double)a*x[k]*x[k]+b*x[k]-y[k])<=1e-6)mask1|=(1<<k);
            }
            if(visit[mask1])continue;
            for(int k = 0;k < n;k++){
                if((mask1>>k) & 1)line_mask[k].push_back(mask1);
            }
            visit[mask1] = 1;
            //過k點的所有經過的點的狀態
        }
    }
    memset(dp,0x3f3f3f,sizeof(dp));
    dp[0] = 0;

    //O(n 2^n轉移)
    for(int s = 1;s < m;s++){
        int cur = __lg(s),len = line_mask[cur].size();
        for(int i = 0;i < len;i++){
            int mk = line_mask[cur][i];
            dp[s] = min(dp[s], dp[(s&(~mk))] + 1);
            if(dp[s] == 1)break;
        }
    }
    printf("%d\n",dp[m-1]);
}

int main(){
    scanf("%d",&t);
    while(t--){
        solve();
    }
}
```

### 硬幣問題

[題目連結](https://neoj.sprout.tw/problem/159/)

有點訝異，原本想說把code丟上去測一下，看看能過前面幾筆測資，結果卻直接全部過了！我也沒有做什麼特別的DP優化，丟上去卻直接Accept了！這題跟原本的硬幣問題有些許的不同，在於他有限制每一種硬幣的數量，面額為 $C_i$ 的硬幣共有 $K_i$ 個，也就是要問你能否用這些固定數量的硬幣湊出特定的面額。

我們可以討論不同的面額的餘數，時間複雜度分析估計約為：$O(N\times C_i\times \frac{M}{C_i})=O(NM)$，照理來說因為要執行t次，算下來有可能會TLE，不過這一題沒有卡就是了。

```cpp=
#include <bits/stdc++.h>
#define N 105
#define mod 1000000007
#define FOR(i,n) for(int i=0;i<n;i++)
#define ios ios::sync_with_stdio(0)
using namespace std;
int t;

void solve(){
    int n,m,val[N],num[N];cin>>n>>m;
    for(int i=0;i<n;i++)cin>>val[i]>>num[i];
    
    bool dp[20005];
    memset(dp,0,sizeof(dp));
    dp[0] = 1;
    
    for(int i=0;i<n;i++){
        for(int k=0;k<val[i];k++){
            int left = num[i];
            for(int j=k+val[i];j<=m;j+=val[i]){
                if(dp[j]!=0)left = num[i];
                else if(left>0 && dp[j-val[i]]){
                    left--;
                    dp[j] = 1;
                }
            }
        }
    }
    if(dp[m]==1)cout<<"Yes"<<endl;
    else cout<<"No"<<endl;
}

int main(){
    ios;
    cin>>t;
    while(t--){
        solve();
    }
}
```

### 烏龜疊疊樂---易

[題目連結](https://neoj.sprout.tw/problem/185/)
這題簡單來說，是給定一個數列，要嘗試分配成好幾堆，每一堆的數量不超過k個。其中第i堆的總和乘上(i-1)，算出來的總和要最小。由題目敘述可以知道，每一堆到後面乘上的數字是越來越大，也就是被加上的次數會越來越多，如果我們將數列倒過來dp會比正面做來得容易維護。

{% note success %}

**轉移式**
dp[i] = max(dp[j]+pref[j]),for i-k≤j≤i-1，pref是維護已經反序過後的數列之前綴和。以範測為例，當k=2時，dp[i]可以選擇的就是從dp[i-1]跟dp[i-2]轉移，如果選擇dp[i-2]就表示第i項會和第i-1項合併成一堆，加上pref[i-2]也就是把前面的每一堆的數量乘上的數字都加一，也就是加上一個前綴和（由於每一個數字只會被合併一次）

**小證明：每一個數字只會被合併一次**
在轉移的過程中，有沒有可能會發生dp[i]要轉移的對象是dp[i-2]，但dp[i-1]卻已經被融合過了，造成三個烏龜融合在一起的情況？

也就是下圖這一種情況，dp[i-1]、dp[i-2]合併，dp[i]、dp[i-1]合併，變成三個融合超過k的情況（dp[i-2],dp[i-1],dp[i]合併）。
![](https://i.imgur.com/3NufpZ9.png)
答案：不會

首先先從dp[i-3]轉移的dp[i-1]表示第i-2跟i-1是合併成一堆，接下來的dp[i]從dp[i-2]轉移，加上了pref[i-2]，也就是把i-2獨立出去，i跟i-1合併成一堆，因此不會有大於k個烏龜合併在一起的情況。
{% endnote %}

這一題的轉移式不好想，要把東西反序之後比較好dp，轉移式也比較好寫出來！觀察到越後面的數字被重複加的機會比較大（高度越高），因此將數列的順續顛倒搭配前綴和就能比較方便的轉移！

#### 定義

$dp[i]$定義為使用1到i個烏龜，違和度的最大值

#### 轉移式

$dp[i] = max(dp[j]+pref[j]),for\ i-k≤j≤i-1$
這個轉移式是建立在順序顛倒的情況，在輸入的時候順便把逆序。

#### 邊界

$for\ each\ dp[i] = 0$

```cpp=
#include <bits/stdc++.h>
#define int long long
#define N 500005
#define ios ios::sync_with_stdio(0)
#define FOR(i,n) for(int i=0;i<n;i++)
using namespace std;
int n,k,arr[N],pref[N];

signed main(){
    ios;
    cin>>n>>k;
    for(int i=n;i>0;i--)cin>>arr[i];
    for(int i=1;i<=n;i++)pref[i] = pref[i-1]+arr[i];
    
    int dp[N];
    memset(dp,0,sizeof(dp));
    
    deque<pair<int,int>> deq;
    deq.push_back(make_pair(0,0));
    
    for(int i=1;i<=n;i++){
        while(!deq.empty() && deq.front().second<i-k)deq.pop_front();
        dp[i] = deq.front().first;
        while(!deq.empty() && deq.back().first <= dp[i]+pref[i])deq.pop_back();
        deq.push_back(make_pair(dp[i]+pref[i],i));
    }
    cout<<dp[n]<<endl;
}
```

## 手寫作業

這一週的主題是NP問題以及SAT，證明好難、數學好難！

# 資芽第十週：進階圖論（一）

## 上課內容

### 並查集 Disjoint Set

目標：快速判斷兩個元素是否同屬一個集合
功能：詢問元素隸屬的集合、合併兩個集合，在圖論中，集合通常表示連通快，並查集可以查詢任兩點是否連通。在實作MST的時候，當我們要檢查兩個點連接成的邊是否會跟其他已經加入的邊形成環，就會使用並查集幫助我們判斷！
複雜度：非常優秀，可以說是O(1)

```cpp=
#include <bits/stdc++.h>
#define N 100001
using namespace std;
int n,boss[N],num[N];

int find_boss(int a){        //使用路徑壓縮
    if(a==boss[a])return a;
    else return boss[a] = find_boss(boss[a]);
}

void merge(int a,int b){    //啟發式合併
    if(num[a]>=num[b]){     //較小的集合合併到較大的集合
        num[a] += num[b];
        boss[b] = a;
    }
    else{
        num[b] += num[a];
        boss[a] = b;
    }
}
signed main(){
    cin>>n;
    for(int i=1;i<=n;i++){
        boss[i] = i;
        num[i] = 1;
    }
    while(1){
        int a,b;cin>>a>>b;                    //要查詢兩個元素
        a = find_boss(a),b = find_boss(b);    //判斷是否同屬一個集合
        if(a!=b){                             //老大不同屬不同集合
            cout<<"不同集合"<<endl;
            merge(a, b);
        }
        else cout<<"同集合"<<endl;
        for(int i=1;i<=n;i++)cout<<boss[i]<<" ";
        cout<<endl;
    }
}
```

{% note success %}
**路徑壓縮**
在回傳過程中順便利用遞迴把所經過的boss指向最上面的boss，由左圖變成右圖，效率更高（實測其實差不多，可能logn已經夠小加上遞迴本來就比較慢）
用遞迴實現的特性實現，遞迴拉上來的時候順便更新boss[x]：
![](https://i.imgur.com/k8FPb0U.png)

路徑壓縮搭配啟發式合併，複雜度：$O(α(N))$等同常數
{% endnote %}

## 上機作業

### 最小生成樹

[題目連結NEOJ](https://neoj.sprout.tw/problem/734/)
[題目連結TIOJ](https://tioj.ck.tp.edu.tw/problems/1211)
[實作講義](https://reurl.cc/ze8Qr6)
[資芽講義](https://www.csie.ntu.edu.tw/~sprout/algo2021/ppt_pdf/week11/Minimum_spanning_tree.pdf)

這一題手刻最小生成樹，感覺蠻好的，都是利用一些性質來實作最小生成樹

#### 性質一：樹

最小生成圖一定會是一棵樹，不具備環，且一共有n-1條邊
如果這張圖不是樹，則必定有環，即代表可以再拔掉一條邊使權重和更小

#### 性質二：Cycle Property

C是圖上的一個環，e是C上權重最大一條邊，則e必不在MST一部分（證明：反證，假設e在MST，則加入另一邊比e權重更小形成環，此時把e拔掉可以形成更小的權重和，因此矛盾）

#### 性質三：Cut Property

把圖上的點集分割成兩半，則cut上面的邊集合中最小權重的邊e會在MST裡面（證明：反證，假設e不在MST，則加入e會形成連接兩半節點的環，此時拔掉另一比e權重大的邊，會形成更小權重和，因此矛盾）

利用性質二、性質三，就可以利用Kruskal演算法：利用到併查集的維護搭配路徑壓縮，可以快速判斷一個元素是否與另一個元素同屬一個集合（複雜度幾乎可以$O(1)$，更準確來說是[$O(\alpha)$](https://zh.wikipedia.org/wiki/%E9%98%BF%E5%85%8B%E6%9B%BC%E5%87%BD%E6%95%B8)），Kruskal總複雜度為 $O(ElogE)$
另外還有一種實作方式**Prim's algorithm**，等等再看

{% note info %}
**實作步驟**

**1. 初始化設定：** 設定好boss以及集合大小等參數

**2. 對所有的邊依照權重排序：** 利用$O(m\log m)$的時間排序，將邊依照權重由小到大加入MST中

**3. 依序加入邊：** 以下會有兩種情況，可以用並查集+路徑壓縮（優化）判斷是否會形成環

- Case 1: 如果加了這條邊形成環（查到有一樣的boss）
  - 那這條邊會是這個環上的最大邊
  - 根據 Cycle property，<font color="#f00">這條邊不會是MST的一部分</font>
- Case 2: 加了這條邊不會形成環
  - 那這條邊是條橋，連接左右兩棵樹
  - 根據 Cut Property，因為這條邊是這個 cut 上最小的邊
  - 所以<font color="#f00">這條邊會是 MST 的一部分</font>

**4. 判斷是否合法：** n個點所形成的樹會有n-1條邊，最小生成樹是一棵樹就必須滿足條件。
{% endnote %}

#### KRUSKAL'S ALGORITHM

![](https://i.imgur.com/sDAXXKI.png)

```cpp=
#include <bits/stdc++.h>
#define int long long
#define N 200005
#define ios ios::sync_with_stdio(0),cin.tie(0)
using namespace std;
int n,m,num[N],boss[N];
struct Node{
    int x,y,w;
}edge[N];

bool cmp(Node a, Node b){
    return a.w < b.w;
}
int findboss(int a){                    //尋找集合的老大（代表整個集合）
    if(a==boss[a])return a;
    return boss[a]=findboss(boss[a]);   //遞迴搭配路徑壓縮
}

signed main(){
    ios;
    cin>>n>>m;
    for(int i=0;i<m;i++){
        cin>>edge[i].x>>edge[i].y>>edge[i].w;
    }
    for(int i=0;i<n;i++){
        num[i] = 1;
        boss[i] = i;
    }
    sort(edge,edge+m,cmp);                 //依照權重大小放入MST
    int result = 0,num_edge = 0;
    for(int i=0;i<m && num_edge<n;i++){    //樹必須滿足小於n-1條邊
        int a = findboss(edge[i].x),b = findboss(edge[i].y);
        if(a!=b){                          //boss不同可以放入MST（加入不會形成環by Cut Property）
            if(num[a]>=num[b]){            //執行啟發式合併
                boss[b] = a;
                num[a]+=num[b];
            }
            else{
                boss[a] = b;
                num[b]+=num[a];
            }
            result+=edge[i].w;
            num_edge++;
        }
    }
    cout<<result<<endl;
    //這邊可以判斷num_edge==n-1有沒有成立，不過題目是輸入都可以形成MST
}
```

### Prim's Algorithm

是一種貪婪演算法，首先取任一點加入最小生成樹中，接著將連到的邊加入heap中，每一次取出heap中邊權重最小的邊，如果這一條邊連到的點尚未被走訪，則加入這一條邊為最小生成數。正確性證明則可用cut proprity證明每次都加入權重最小的邊即為最小生成數。

![](https://i.imgur.com/ZFELWZq.png)

```cpp=
#include <bits/stdc++.h>
#define Orz ios::sync_with_stdio(0),cin.tie(0)
#define rep(i,a,b) for(int i=a;i<=b;i++)
#define pii pair<int,int>
#define pdd pair<double,double>
#define int long long
#define ll long long
#define ld long double
#define N 200005
#define eps 1e-9
#define x first
#define y second
using namespace std;
int n,m;
bool visit[N];

struct node{
    int to,w;
};
vector<node> edge[N];

struct cmp{
    bool operator()(node a,node b){
        return a.w > b.w;
    }
};

signed main(){
    Orz;
    memset(visit,0,sizeof(visit));
    cin>>n>>m;
    rep(i,0,m-1){
        int a,b,w;cin>>a>>b>>w;
        edge[a].push_back({b,w});
        edge[b].push_back({a,w});
    }
    priority_queue<node,vector<node>,cmp> pq;
    visit[1] = 1;
    for(auto i : edge[1])pq.push(i);
    int num_edge = 0,ans = 0;
    while(num_edge < n-1){
        node x = pq.top();pq.pop();
        if(visit[x.to])continue;
        visit[x.to] = 1;
        num_edge += 1;
        ans += x.w;
        for(auto i : edge[x.to]){
            pq.push(i);
        }
    }
    cout<<ans<<endl;
}
```

### 高棕櫚的意外收穫

[題目連結](https://neoj.sprout.tw/problem/169/)
[講義連結](https://www.csie.ntu.edu.tw/~sprout/algo2021/ppt_pdf/week11/euler_hamilton.pdf)

這一題就是按照字典序print出尤拉路徑，特別用**鄰接矩陣**來存圖是因為要按照字典序輸出，如果用鏈結串列來存還要花時間排序，因此用大一點的空間來節省時間

尤拉路徑也是一筆畫問題，每一個節點一定會有進有出，因此原先的無向圖上每一點一定度數為偶數，若有奇點則選擇其中一個奇點作為起點，當奇點數量超過兩個則代表無解。我們將DFS的過程中離開當前節點的順序紀錄起來，逆序輸出就是一組合法歐拉迴路的解了！
{% note info %}
**實作程序**

**1. 判斷奇點個數**，若奇點個數k：

- k > 2，那麼無解
- k = 2，則選擇其中一個奇點作為起點
- k = 0，則選擇任意一個點作為起點

**2. DFS 執行下列步驟**
> 若當前節點還有尚未走過的邊，那麼拜訪該邊，並在拜訪完後輸出該邊
> 否則離開當前結點

**3. 若還有節點尚未拜訪，則無解**
**4. 否則輸出順序即為一組解**
{% endnote %}

```cpp=
#include <bits/stdc++.h>
#define N 501
using namespace std;
int n,Edge[N][N],ans[1025],ind = 0;

void DFS(int cur){
    for(int i=1;i<=500;i++){
        if(Edge[cur][i]){
            Edge[cur][i]--;Edge[i][cur]--;
            DFS(i);
        }
    }
    ans[ind++] = cur;
}

signed main(){
    cin>>n;
    memset(Edge, 0, sizeof(Edge));
    for(int i=0;i<n;i++){
        int a,b;cin>>a>>b;
        Edge[a][b]++;
        Edge[b][a]++;
    }
    int start = 1;                //開始的節點編號
    for(int i=1;i<=500;i++){
        int sum = 0;
        for(int j=1;j<=500;j++){
            sum+=Edge[i][j];
        }
        if(sum%2!=0){             //找到第一個度數為奇數的節點
            start = i;
            break;
        }
    }
    DFS(start);
    for(int i=ind-1;i>=0;i--)cout<<ans[i]<<endl;
}
```

### 陣線推進

[題目連結](https://neoj.sprout.tw/problem/165/)
拓墣排序題，兩種實作方式，第一是BFS變形實作（queue實作），被pop出來的順序就是topological sort
第二種就是DFS搭配時間戳記，最晚離開的放在最前面

這一題是把入度為0的節點先push進priority_queue（按照字典序），當一個點處理過之後，入度變成0的所有它指向的點在push進queue裡面
拓墣排序可以用在DAG的判定以及解決具有依賴關係的問題

```cpp=
#include <bits/stdc++.h>
#define N 100001
using namespace std;
int t,n,m,deg[N];//入度
int ans[N],ind = 0;

signed main(){
    cin>>t;
    while(t--){
        memset(deg,0,sizeof(deg));
        memset(ans, 0, sizeof(ans));
        ind = 0;
        vector<int> edge[N];
        cin>>n>>m;
        for(int i=0;i<m;i++){
            int a,b;cin>>a>>b;
            edge[a].push_back(b);
            deg[b]++;
        }
        priority_queue<int,vector<int>,greater<int> > qq;//priority_queue取代queue
        for(int i=0;i<n;i++)if(deg[i]==0)qq.push(i);
        while(!qq.empty()){
            int cur = qq.top();
            qq.pop();
            ans[ind++] = cur;
            int len = edge[cur].size();
            for(int i=0;i<len;i++){
                deg[edge[cur][i]]--;
                if(deg[edge[cur][i]]==0)
                    qq.push(edge[cur][i]);
            }
        }
        if(ind==n){
            cout<<ans[0];
            for(int i=1;i<n;i++)cout<<" "<<ans[i];
            cout<<endl;
        }
        else cout<<"QAQ"<<endl;
    }
}
```

## 手寫作業

這一次的手寫也蠻困難的，討論BIT(fenwick tree)的實作以及複雜度！

# 資芽第十一週：進階圖論（二）

## 上課內容

### 雙連通元件

樹壓平、點雙連通、邊雙連通

### SCC強連通元件

LCA（最低共同祖先）

## 上機作業

### 高棕櫚傳遞鏈

[題目連結](https://neoj.sprout.tw/problem/183/)
在一張圖中找割點（定義：拔掉它整張圖就不連通了）
作法：如果用暴力，可以每一個點拔掉，做一次DFS判斷是否聯通（效率太差）

效率更高的就是**Trajan 演算法**找AP(articulation point)

{% note info %}
**Tarjan's algorithm 找 AP**
邊的種類可以分成：Tree edge,Back edge,Forward edge,Cross edge，其中無向圖中只會有樹邊跟回邊（按照無向邊DFS的結果，Foward edge 都會變成子孫的back edge, cross edge 會變成樹邊）

維護一個low函數，代表不經過父節點能到的最小時間戳記（進入）的節點，lv函數為當前節點的時間戳記。一個點是不是割點，只要他的任意子節點的low函數大於等於自己的時間戳記編號，那把這個點拔掉，他小孩就走不到祖先了（如果走得到祖先，對於這一棵子樹，拔掉當前節點就可利用此邊繼續連通），所以他就是割點。
更新：$low[now] = min(low[now],low[next])$分別為利用子孫或靠自己
{% endnote %}

必須要注意的是，割點判斷時要把root的特例獨立判斷。其實root反而比較簡單，如果root有超過一個子樹，代表拔掉root以後會分裂成以每個子樹為單位的連通塊

```cpp=
#include <bits/stdc++.h>
#define ios ios::sync_with_stdio(0),cin.tie(0);
#define int long long
#define N 1000001
using namespace std;
int n,m,low[N],lv[N],es=1,root,son_cnt = 0;
bool visit[N],ans[N];
vector<int> edge[N];
//low函數為不經過父節點能到的最小時間戳記,lv為當前進入時間戳記

void DFS(int now,int father){
    visit[now] = 1;             //將此點設為已拜訪
    low[now] = lv[now] = es++;  //進入的時間戳記
    
    int len = edge[now].size();
    for(int i=0;i<len;i++){     //拜訪每一個子孫
        int next = edge[now][i];
        if(now==root && !visit[next])son_cnt++;//計算root小孩（處理特例）
        if(!visit[next]){       //排除走到祖先的情況
            DFS(next, now);
            if(low[next]>=lv[now] && now!=root)ans[now]=1;
            //無法透過小孩到達比自己淺的節點，將now設為AP
        }
        if(next!=father)low[now] = min(low[now],low[next]);
        //排除指向父親的情況，如果經過父親，拔掉就不能往更上面去
    }
}

signed main(){
    ios;
    cin>>n>>m;
    memset(lv, 0, sizeof(lv));
    memset(low, 0, sizeof(low));
    memset(visit, 0, sizeof(visit));
    memset(ans,0,sizeof(ans));
    
    for(int i=0;i<m;i++){
        int x,y;cin>>x>>y;
        edge[x].push_back(y);
        edge[y].push_back(x);
    }
    for(int i=0;i<n;i++){
        if(!visit[i]){
            root = i;
            son_cnt = 0;
            DFS(i, i);
            if(son_cnt>1)ans[root] = 1;
        }
    }
    for(int i=0;i<n;i++)if(ans[i])cout<<i<<endl;
}
```

### 謠言問題

[題目連結](https://neoj.sprout.tw/problem/179)
跟上一題（高棕櫚傳遞鏈）蠻類似的，一樣找到割點，不同的是要在DFS過程中同時紀錄子樹節點的數量。如果碰到割點，維護拔掉它之後分裂出去那些子樹的節點個數，到時候透過節點總數-拔掉後分裂個數即可推算有幾個人會知道謠言，取min即可

```cpp=
#include <bits/stdc++.h>
#define ios ios::sync_with_stdio(0),cin.tie(0);
#define int long long
#define N 30001
using namespace std;
int n,m,root,lv[N],low[N],tree_cnt[N],es = 1;
bool visit[N],ans[N];
vector<int> edge[N];

int DFS(int now,int father){      //回傳當前子節點個數
    visit[now] = 1;
    lv[now] = es;
    low[now] = es++;
    
    int len = edge[now].size(),sum=1;//計算節點數
    for(int i=0;i<len;i++){
        int next = edge[now][i];     //下一個節點
        
        if(!visit[next]){
            int temp = DFS(next,now);
            sum += temp;
            if(low[next] >= lv[now] && now!=root){ //不能拔掉root
                ans[now] = 1;           //設為AP
                tree_cnt[now] += temp;  //被拔掉後可被分割成幾個連通塊節點數
            }
        }
        if(next!=father)low[now] = min(low[now],low[next]);
    }
    return sum;
}

signed main(){
    ios;
    cin>>n>>m;
    for(int i=0;i<m;i++){
        int x,y;cin>>x>>y;
        edge[x].push_back(y);
        edge[y].push_back(x);
    }
    cin>>root;
    
    memset(ans, 0, sizeof(ans));
    memset(visit, 0, sizeof(visit));
    memset(tree_cnt, 0, sizeof(tree_cnt));
    
    int sum = DFS(root,root),min_cnt = INT_MAX,min_pos = 0;
    
    for(int i=1;i<=n;i++){
        int temp = sum-tree_cnt[i]; //拔掉後剩下連通塊大小（跟root連的）
        if(ans[i] && temp < min_cnt){
            min_pos = i;        //拔掉第幾個
            min_cnt = temp;     //更新拔掉後剩下連通塊大小
        }
    }
    if(min_cnt == INT_MAX)cout<<0<<endl;
    else cout<<min_pos<<" "<<min_cnt<<endl;
}

```

### 高棕櫚傳遞鏈

[題目連結](https://neoj.sprout.tw/problem/183/)
前兩題找割點，這一題是找橋，跟找割點的方法幾乎一樣，而且對於邊還不需要討論root的情況（root變成不是特例），然後判斷割點的 >= 變成 >，原因可以透過畫圖理解（把點拔掉跟把邊拔掉的差別），就可以實作了！
題目有一個特別的要求，按照給邊的順序進行輸出，那可以搭配set來快速查看某一元素是否在集合內$O(logN)$，接著就按照給定的條件來輸出

```cpp=
#include <bits/stdc++.h>
#define ios ios::sync_with_stdio(0),cin.tie(0);
#define int long long
#define N 1000001
using namespace std;
int n,m,lv[N],low[N],timestamp = 1;
bool visit[N];
vector<int> edge[N];
vector<pair<int, int>> ans;
set<pair<int, int>>s;

void DFS(int now,int father){
    lv[now] = low[now] = timestamp++;
    visit[now] = 1;
    
    int len = edge[now].size();
    for(int i=0;i<len;i++){
        int next = edge[now][i];
        
        if(!visit[next]){
            DFS(next, now);
            if(low[next] > lv[now]){
                if(next<now)s.insert(make_pair(next,now));
                else s.insert(make_pair(now, next));
            }
        }
        if(next!=father)low[now] = min(low[now],low[next]);
    }
}

signed main(){
    ios;
    cin>>n>>m;
    for(int i=0;i<m;i++){
        int x,y;cin>>x>>y;
        edge[x].push_back(y);
        edge[y].push_back(x);
        ans.push_back(make_pair(x, y));
    }
    memset(visit, 0, sizeof(visit));
    
    for(int i=1;i<=n;i++){
        if(!visit[i]){
            DFS(i, i);
        }
    }
    for(int i=0;i<m;i++){
        pair<int, int> temp = ans[i];
        if(s.find(temp)!=s.end()){
            cout<<temp.first<<" "<<temp.second<<endl;
        }
    }
}
```

### 芽芽逛大街

[題目連結](https://neoj.sprout.tw/problem/739/)
有向無環圖的 case → DAG 最長路徑!
將每個強連通元件縮成點後，因為內部的點可以一直亂走全部走到，所以只要將新點的點權更新成內部所有點的點權總和，得到一張新的有向無環圖，就可以直接做 DAG 最長路徑得到答案。

```cpp=
#include <bits/stdc++.h>
#define int long long
#define ios ios::sync_with_stdio(0),cin.tie(0);
#define N 500002
using namespace std;
int n,m,dfn[N],low[N],es = 1,stk_in[N],vertex_val[N],deg[N];
//dfn為時間戳記,low為back, cross edge（經過最多一次）到達最小點dfn,stk_in是否在stack內
bool visit[N];
int scc[N],scc_ind = 0,scc_val[N];//紀錄屬於哪個scc,scc編號,scc編號的價值（權重和）
int topological_order[N],ind = 0;

struct edg{
    int to;
    int val;
};

vector<edg> edge[N],new_edge[N];
stack<int> s;

void DFS(int now){
    dfn[now] = low[now] = es++;
    s.push(now);
    stk_in[now] = 1;
    visit[now] = 1;
    int len = edge[now].size();
    for(int i=0;i<len;i++){
        int next = edge[now][i].to;
        if(!visit[next]){       //尚未拜訪則拜訪
            DFS(next);
            low[now] = min(low[now],low[next]);
        }
        else if(stk_in[next]){  //在stk內且已拜訪->同屬一個SCC
            low[now] = min(low[now],dfn[next]);
            //這條邊指向還沒出stack的點，可為cross or back edge 更新low[now]
        }
    }
    //如果是scc就pop stack裡面的東西
    if(low[now] == dfn[now]){
        stk_in[now] = 0;        //pop出stack裡面
        scc[now] = ++scc_ind;   //進行SCC編號
        scc_val[scc_ind] = vertex_val[now]; //更新點權
        while(s.top()!=now){                //pop直到now被找到
            scc[s.top()] = scc_ind;
            stk_in[s.top()] = 0;            //pop出來
            scc_val[scc_ind] += vertex_val[s.top()];
            s.pop();
        }
        s.pop();//將stack 中now也pop
    }
}

signed main(){
    ios;
    cin>>n>>m;
    memset(visit, 0, sizeof(visit));
    memset(stk_in, 0, sizeof(stk_in));
    memset(deg, 0, sizeof(deg));
    for(int i=1;i<=n;i++){
        int x;cin>>x;
        vertex_val[i] = x;
    }
    for(int i=0;i<m;i++){
        int x,y,val;cin>>x>>y>>val;//x指向y
        edge[x].push_back( edg{y,val} );
    }
    for(int i=1;i<=n;i++)
        if(!visit[i])DFS(i);
    
    //枚舉每一條邊更新邊權
    for(int i=1;i<=n;i++){
        int len = edge[i].size();
        for(int j=0;j<len;j++){
            int to = edge[i][j].to;
            if(scc[to]==scc[i]){
                int ind = scc[to];
                scc_val[ind]+=edge[i][j].val;
            }
            else{   //不同SCC指向不同的邊
                new_edge[scc[i]].push_back( edg{scc[to],edge[i][j].val});
                deg[scc[to]]++;
            }
        }
    }
    
    //Topological sort
    queue<int> q;
    
    for(int i=1;i<=scc_ind;i++)
        if(deg[i]==0)q.push(i);

    while(!q.empty()){
        int now = q.front(),len = new_edge[now].size();
        topological_order[ind++] = now;
        q.pop();
        for(int i=0;i<len;i++){
            int next = new_edge[now][i].to;
            if(--deg[next]==0)q.push(next);
        }
    }
    
    //拓墣排序完進行DP找最長路徑
    int dp[ind],ans = 0;
    for(int i=1;i<=scc_ind;i++){
        dp[i] = scc_val[i];
        ans = max(ans, scc_val[i]);
    }
    
    for(int i=0;i<ind;i++){
        int now = topological_order[i],len = new_edge[now].size();
        for(int j=0;j<len;j++){
            int next = new_edge[now][j].to;
            dp[next] = max(dp[next],scc_val[next]+dp[now]+new_edge[now][j].val);
            ans = max(ans, dp[next]);
        }
    }
    cout<<ans<<endl;
}
```

## 手寫作業

介紹雜湊，重點在於rolling hash，這也是下一週隨機課程當中會用到的重要概念。

# 資芽第十二週：隨機算法

## 上課內容

這一週講隨機，主要講rolling hash，字串的hash、隨機的例題，像是矩陣乘法的驗證$O(n^2)$、訪問區間各元素出現次數是否為k的倍數，最近點對的$O(n)$作法等（還有東西還沒有看XD）

## 上機作業

### 欸迪的字串

[題目連結](https://neoj.sprout.tw/problem/265/)
輸入有兩行，第一行包含一個長度介於[1,500000]的字串S，第二行包含一個長度介於[1,500000]的字串T，請輸出一串遞增的數列表示字串S出現在字串T的哪些地方。

**Rolling hash**：
$$H(s[1:n]) = \sum_{i=0}^n S_i\times C^{n-i}$$
我們如果假設我們要找的目標字串S，長度為m，他的雜湊值是：
$$H(S)= \sum_{i=1}^mS_i \times C^{m-i}$$
則對於長度為L與r-1的字串T，其hash值為：
$$H(T[1:L]) = \sum_{i=1}^LT_i\times C^{L-i}$$
$$H(T[1:r-1]) = \sum_{i=1}^{r-1}T_i\times C^{r-1-i}$$
則目標區間長度為m的字串長度即為字串S的雜湊值，將T[1:L]的雜湊值扣掉$C^{m}$乘上T[1:r-1]的雜湊值，即為所求。

$$\begin{split}H(S)&=H(T[r:L])\\ &=H(T[1:L])-H(T[1:r-1])\\
&= \sum_{i=1}^LT_i\times C^{L-i} - C^{m}\times \sum_{i=1}^{r-1}T_i\times C^{r-1-i} \\
&=\sum_{i=1}^LT_i\times C^{L-i}-\sum_{i=1}^{r-1}T_i\times C^{L-i} \\
&= \sum_{i=r}^{L}T_i\times C^{L-i} \\
&= \sum_{i=1}^{m}T_{i+r-1}\times C^{m-1}\end{split}$$
就可以得到當前區間的hash值
所以這整題就變成維護字串T的前綴和，透過以上方式O(n)得到每一個index的hash值，接著O(1)比對即可。
{% note success %}
Q:照最基本的暴力比對可不可行？
A:可以構造出需要比對很多次的字串，有可能會被卡TLE之類的
{% endnote %}
以後一定要記得，必須要行末**輸出一行**，我在這邊WA超久

```cpp=
#include <bits/stdc++.h>
#define int unsigned long long int
#define INF 0x3f3f3f3f
#define N 500005
#define mod 1000000007
using namespace std;
int power[N],hash_func[N],charc[30],n,m,C = 137;
char target[N],a[N];
vector<int> vec;

void init(){
    power[0] = 1;
    srand(time(NULL));
    for(int i=1;i<=m;i++){
        power[i] = ((power[i-1]*C)+mod)%mod;
    }
    for(int i=0;i<30;i++){    //再用rand來使字串更亂（不用也沒差）
        int temp = rand();
        charc[i] = temp;
    }
}
signed main(){
    scanf("%s\n%s",target+1,a+1);
    m = strlen(target+1);
    n = strlen(a+1);
    init();
    hash_func[0] = 0;
    
    for(int i=1;i<=n;i++){
        hash_func[i] = ((C*hash_func[i-1]+(charc[a[i]-'a']))+mod)%mod;
    }
    int sum = 0;
    for(int i=1;i<=m;i++){
        sum = ((sum*C+(charc[target[i]-'a']))+mod)%mod;
    }
    for(int i=0;i<=n-m;i++){
        if(sum == ((hash_func[i+m]-((hash_func[i]*power[m])+mod)%mod)+mod)%mod)
            vec.push_back(i);
    }
    int len = vec.size();
    
    if(len>0)printf("%llu",vec[0]);
    for(int i=1;i<len;i++){
        printf(" %llu",vec[i]);
    }
    cout<<endl;    //這一行是毒瘤
}
```

### 溫力的故事

[題目連結](https://neoj.sprout.tw/problem/266/)
給字串集合A，接下來給m個字串，問你在m個字串中i字串一共出現了幾次。
直覺的想法就是直接用multiset來解這一題。

```cpp=
#include <bits/stdc++.h>
#define ios ios::sync_with_stdio(0),cin.tie(0)
#define int unsigned long long int
#define N 105
using namespace std;
int n,m,power[N],C = 13331;

void init(){
    power[0] = 1;
    for(int i=1;i<=N;i++){
        power[i] = power[i-1]*C;
    }
}

int func(string s){
    int len = s.size(),sum = 0;
    for(int i=1;i<=len;i++){
        sum+=((int)s[i-1]*power[len-i]);
    }
    return sum;
}

signed main(){
    ios;
    init();
    multiset<int> ss;
    cin>>n>>m;
    for(int i=0;i<n;i++){
        string s;cin>>s;
        int temp = func(s);
        ss.insert(temp);
    }
    for(int i=0;i<m;i++){
        string s;cin>>s;
        int temp = func(s);
        cout<<ss.count(temp)<<endl;
    }
}
```

### 想不到題目標題QQ

[題目連結](https://neoj.sprout.tw/problem/793/)
圖中的-2改成-(k-1)，對於圖中的x,y都是設計成大數亂數，要承擔的風險就是這些大數相加之後不應該變成0的卻變成0（機率超小）
還有一個重點，對於每一筆詢問都要求出區間的和，因此要用O(n)維護前綴和！（一開始沒想到tle，不過靈光一閃維護前綴和就過了）
![](https://i.imgur.com/Z4NzA4J.png)

```cpp=
#include <bits/stdc++.h>
#define ios ios::sync_with_stdio(0),cin.tie(0)
#define int long long int
#define N 500005
#define mod 1000000007
using namespace std;
int n,k,m,arr[N],times[N],fuck[N],value[N],pre[N];

void init(){
    memset(times, 0, sizeof(times));
    srand(time(NULL));
    pre[0] = 0;
    for(int i=0;i<=N;i++){
        int temp = rand()%mod;
        fuck[i] = temp;
    }
}

signed main(){
    ios;
    init();
    cin>>n>>k>>m;
    for(int i=1;i<=n;i++){
        cin>>arr[i];
        times[arr[i]]++;
        if(times[arr[i]]%k==0){
            value[i] = -(k-1)*fuck[arr[i]];
        }
        else{
            value[i] = fuck[arr[i]];
        }
        pre[i] = (pre[i-1]+value[i]);
    }

    for(int i=1;i<=m;i++){
        int l,r,sum=0;cin>>l>>r;
        sum = pre[r]-pre[l-1];
        if(sum==0)cout<<1;
        else cout<<0;
    }
    cout<<endl;
}
```

### 矩陣乘法

[題目連結](https://neoj.sprout.tw/problem/740/)
給定三個矩陣ABC，驗證A乘上B是否為C。
當然可以用$O(n^3)$的時間實際驗證A乘上B的結果與C比對，但其實有更快的方法
如下，構造出矩陣R，這一題可以思考要1乘上多少的矩陣，透過兩次O(n^2)的乘法，就可以比對這個R矩陣乘上C矩陣的結果
這題蠻簡單的，一次就過了

```cpp=
#include <bits/stdc++.h>
#define ios ios::sync_with_stdio(0),cin.tie(0)
#define int long long int
#define N 1505
#define mod 1000000007
using namespace std;
int n,m,k,a[N][N],b[N][N],c[N][N],R[N],temp[N],c_new[N];

void init(){
    memset(temp, 0, sizeof(temp));
    srand(time(NULL));
    for(int i=0;i<N;i++){
        int temp = rand()%mod;
        R[i] = temp;
    }
}

void input(){
    cin>>n>>m>>k;
    for(int i=0;i<n;i++){
        for(int j=0;j<m;j++){
            cin>>a[i][j];
        }
    }
    for(int i=0;i<m;i++){
        for(int j=0;j<k;j++){
            cin>>b[i][j];
        }
    }
    for(int i=0;i<n;i++){
        for(int j=0;j<k;j++){
            cin>>c[i][j];
        }
    }
}

signed main(){
    ios;
    init();
    input();
    for(int i=0;i<m;i++){
        for(int j=0;j<k;j++){
            temp[i]+=(b[i][j]*R[j])%mod;
        }
    }
    for(int i=0;i<n;i++){
        for(int j=0;j<m;j++){
            c_new[i]+=(a[i][j]*temp[j])%mod;
        }
    }
    bool flag = 1;
    for(int i=0;i<n;i++){
        int sum = 0;
        for(int j=0;j<k;j++){
            sum+=(c[i][j]*R[j])%mod;
        }
        if(c_new[i]!=sum){
            flag = 0;
            break;
        }
    }
    if(flag)cout<<"Yes"<<endl;
    else cout<<"No"<<endl;
}
```

### 最近點對

[題目連結](https://neoj.sprout.tw/problem/795/)
大魔王題，害我TLE 27次。
二維平面，給定n個點，求出最近的兩個點的距離。
重大BUG害我卡三天的BUG在於，題目要求輸出距離的平方，結果在計算過程中就直接把距離當作是平方，自然有很多點會同時出現在一格網格座標中（我弱，一直TLE），還一直用什麼multi_map之類的，當然就是TLE嘍。
暴力做：$O(n^2)$，分治做：$O(nlogn)$，隨機做：期望$O(n)$,最慘$O(n^2)$
這一題的步驟：

1. 把 N 個點平移到第一象限
2. 把 N 個點的順序隨機打亂，d = dis(𝑎1, 𝑎2)
3. 以 𝑟 = 𝑑/2 的大小將二維平面切成網格狀，並將這些網格也 𝑥𝑦
以座標表示:點(𝑥,𝑦)會落入座標為 (𝑟,𝑟)
<font color="#f00">不會有兩個點在同一個格子內</font>（很重要的性質）
4. 一直把點加進網格中
如果產生新的最近點對一定會出現在以某個點為中心的5×5宮格內
5. 找到更近的最近點對
更新 r= 新的最近點對 /2 ，回到步驟三重來 O(i+1)

{% note info %}
**期望複雜度 $O(n)$ 證明**

考慮加入第i+1個點時出現新的最近點對，發生的機率為：在$C_2^{i+1}$個配對中跟i+1個點產生最近點對共有i種可能，因此機率為$\frac{2}{i+1}$，當機率發生的時候，必須將所有的點都刪掉重新來一遍（因為r變小，unordered_map裡面的東西也要被清空，重新推入i+1個點），需要付出$O(i+1)$的時間，相乘起來每一個點期望的複雜度為$O(1)$，因此總時間複雜度為$O(n)$。
{% endnote %}

實作上可以用unordered_map，因為保證不會出現同一格的情況（一格間的最長距離也就不超過d，所以當出現在同一格就表示該更新了）

優化：先用$O(n^2)$的作法，找到前面的少數pair（$\sqrt n$）的最短距離，這樣可以減少被重新更新的機會
![](https://i.imgur.com/9XfZNH9.png)

[這篇文章](/N9zvIzP_Se-hpWZSaMv-sQ)有分治的作法，雖然分治的複雜度是O(n\log n)，隨機是$O(n)$，但因為隨機常數比較大的關係，時間比分治慢了快兩倍！下圖是分治的執行結果：
![](https://i.imgur.com/eCxfumY.png)

以下是隨機AC Code：

```cpp=
#include <bits/stdc++.h>
#define int long long int
#define ios ios::sync_with_stdio(0),cin.tie(0)
#define N 200005
#define INF 1000000000LL
#define swift 1000000000
using namespace std;
int n,ans;
double r,d;
int dx[25] = {-2,-2,-2,-2,-2,-1,-1,-1,-1,-1,0,0,0,0,0,1,1,1,1,1,2,2,2,2,2};
int dy[25] = {-2,-1,0,1,2,-2,-1,0,1,2,-2,-1,0,1,2,-2,-1,0,1,2,-2,-1,0,1,2};
unordered_map<int, int> m;

void solve();
inline void init();
void solve();
bool insert(int,int,int);
inline double dis(int,int);
inline int Grid(int);

struct node{
    int x,y,ind;
}point[N];

//函式實作
inline void init(){
    m.clear();
}

inline int Grid(int ind){ //input網格座標
    int x = point[ind].x/r;
    int y = point[ind].y/r;
    return x*INF+y;
}
inline int dist(node a,node b){
    int x = a.x-b.x,y = a.y-b.y;
    return (x*x+y*y);
}

inline double dis(node a,node b){
    int x = a.x-b.x,y = a.y-b.y;
    return sqrt(x*x+y*y);
}

void solve(){
    m.insert(make_pair(Grid(0),0));m.insert(make_pair(Grid(1),1));
    for(int ind = 2;ind < n;ind++){
        int x = point[ind].x/r,y = point[ind].y/r,better=0;
        for(int i=0;i<25;i++){
            int nx = x+dx[i],ny = y+dy[i];
            auto it = m.find(nx*INF+ny);
            if(it!=m.end()){
                double distance = dis(point[it->second],point[ind]);
                if(distance<d){
                    better = 1;
                    ans = dist(point[it->second],point[ind]);
                    d = distance;
                    r = d/2;
                }
            }
        }
        if(better){    //前面的格子直接瑱入
            m.clear();
            for(int i=0;i<=ind;i++)m.insert(make_pair(Grid(i),i));
        }
        else{
            m.insert(make_pair(Grid(ind), ind));
        }
    }
}

signed main(){
    ios;
    init();
    cin>>n;
    for(int i=0;i<n;i++){
        int x,y;cin>>x>>y;
        x+=swift;y+=swift;
        point[i].x = x;point[i].y = y;
    }
    random_shuffle(point, point+n);
    int smalln = sqrt(n);
    ans = dist(point[0],point[1]);
    d = dis(point[0], point[1]);
    for(int i=0;i<=smalln;i++){
        for(int j=i+1;j<=smalln;j++){
            d = min(d,dis(point[i], point[j]));
            ans = min(ans,dist(point[i],point[j]));
        }
    }
    r = d/2;
    solve();
    cout<<ans<<endl;
}
```

## 手寫作業

不用交手寫作業！不過還是有一份「欣賞用」的，講Disjoint Set

# 資芽第十三週：根號算法

## 上課內容

好噁心喔，複雜度竟然帶根號！？ 根號算法有很多例題，最常聽到（或遇到？）的就是**分塊**了！把序列分成$\sqrt{K}$ 塊，在做RMQ（區間加值、區間求和等）的其中一種方式

1. RMQ區間極值詢問（可以用分塊做、線段樹、BIT、稀疏表sparse table）
2. Counting Triangle
3. 分塊（值域分塊、塊狀鍊表等）
4. **操作分塊**

其實根號算法的重點其實在於複雜度分析（講師花很多時間講不同作法的複雜度分析），怎麼樣分析讓複雜度跑出根號

## 上機作業

### Counting Triangles

[題目連結](https://neoj.sprout.tw/problem/252/)
我個人認為做法超級精妙的！有很多種作法，挑一個比較好實作的：
{% note success %}
假設可以 $O(1)$ 回答 (x, y) 是否為圖中的一個邊。對於每條邊 (u, v) ，你花$O(min(d_u, d_v))$去算出包含這條邊的三角形個數
總複雜度：$O(M\sqrt{M})$
{% endnote %}

其中對於均攤查詢一條邊有沒有在圖中，總邊數$O(M)$ $\div$ 詢問每一條邊$O(M)$ = 均攤$O(1)$的複雜度，實作起來非常簡單

```cpp=
#include <bits/stdc++.h>
#define ios ios::sync_with_stdio(0),cin.tie(0)
#define N 100001
#define M 100001
using namespace std;

int main(){
    ios;
    vector<int> G[N],query[N];  //G是存圖、query是存詢問
    int n,m,x[M],y[M];  //x,y存邊用
    bool visit[N];
    
    cin>>n>>m;
    for(int i=0;i<m;i++){
        cin>>x[i]>>y[i];
        G[x[i]].push_back(y[i]);
        G[y[i]].push_back(x[i]);
    }
    int sml = 0,big = 0;
    for(int i=0;i<m;i++){    //保證複雜度為根號的關鍵
        if(G[x[i]].size()<=G[y[i]].size())sml = x[i];
        else sml = y[i];    //尋找哪一個度數比較小
        big = x[i]+y[i]-sml;
        for(int j : G[sml]){   //訪問跟sml有相鄰的所有邊，看有沒有(j,big)
            query[j].push_back(big);
        }
    }
    int ans = 0;
    for(int i=0;i<n;i++){
        //均攤O(1)?
        for(int j:G[i]){
            visit[j] = true;
        }
        for(int j:query[i]){
            ans+=visit[j];
        }
        for(int j:G[i]){
            visit[j] = false;
        }
    }
    cout<<ans/3<<endl;
}
```

### 中國人插隊問題

[題目連結](https://neoj.sprout.tw/problem/213/)
如果用一個vector 存整個序列，query $O(1)$，但新增跟刪除都是$O(N)$
因此，如果改用分塊的方法，用deque下去砸，每K個分成一塊，對於插入與刪除都是$O(K+\frac{N}{K})$，詢問是$O(1)$
根據算幾不等式，我們可以取$K = \sqrt{N}$ 時，複雜度可以達到$O(\sqrt{N})$ ，很不錯吧！

```cpp=
#include <bits/stdc++.h>
#define ios ios::sync_with_stdio(0),cin.tie(0)
using namespace std;

signed main(){
    ios;
    int n,m,K;
    cin>>n>>m;
    K = sqrt(n);
    deque<int> deq[n/K+(m/K)+10];
    for(int i=0;i<n;i++){
        int temp;cin>>temp;
        deq[i/K].push_back(temp);
    }
    for(int i=0;i<m;i++){
        string a;cin>>a;
        if(a[0]=='A'){
            int x,y;cin>>y>>x;y--;
            deq[y/K].insert(deq[y/K].begin()+(y%K), x);
            int len = deq[y/K].size(),cur = y/K;
            while(len>K){
                int temp = deq[cur].back();
                deq[cur].pop_back();
                cur++;
                deq[cur].push_front(temp);
                len = deq[cur].size();
            }
        }
        else if(a[0]=='L'){
            int x;cin>>x;x--;
            deq[x/K].erase(deq[x/K].begin()+(x%K));
            int len = deq[x/K].size(),cur = x/K;
            while(len<K && deq[cur+1].size()!=0){
                int temp = deq[cur+1].front();
                deq[cur+1].pop_front();
                deq[cur].push_back(temp);
                cur++;
                len = deq[cur].size();
            }
        }
        else if(a[0]=='Q'){
            int x;cin>>x;x--;
            int len = x%K;
            auto it = deq[x/K].begin();
            cout<<*(it+len)<<endl;
        }
    }
}
```

### 第 Z 小

[題目連結](https://neoj.sprout.tw/problem/742/)
超討厭，一直RE，結果是卡在沒有開long long，哭啊
這一題是值域分塊，一樣是根據值域每K個分成一塊，去維護每一大塊的數字數量總和，這樣在查詢(query)的時候花 $O(C/K)$ 找到相應大塊(C為值域)，再花 $O(K)$ 的時間掃過小塊，而加值減值都是$O(1)$可以處理
複雜度：$O(K+\frac{C}{K})$ 取 K=$\sqrt{C}$ 有最小值：$O(Q\sqrt{C})$

```cpp=
#include <bits/stdc++.h>
#define int long long
#define ios ios::sync_with_stdio(0),cin.tie(0)
#define N 100005
using namespace std;    //值域分塊
const int K = 100;

signed main(){
    ios;
    int q,arr[N],mn[N/K+10];cin>>q;
    memset(arr, 0, sizeof(arr));
    while(q--){
        int temp;cin>>temp;
        if(temp==1){
            int x,y;cin>>x>>y;
            arr[x]+=y;
            mn[x/K]+=y;
        }
        else if(temp==2){
            int x,y;cin>>x>>y;
            arr[x]-=y;
            mn[x/K]-=y;
        }
        else if(temp==3){
            int z,sum = 0,ind=0;cin>>z; //ind查詢位於第幾塊
            while(sum+mn[ind]<z)sum+=mn[ind++];
            int cur_ind = ind*K;
            for(int i=0;i<=K;i++){
                if(arr[cur_ind+i]){
                    sum+=arr[cur_ind+i];
                    if(sum>=z){
                        cout<<ind*K+i<<endl;
                        break;
                    }
                }
            }
        }
    }
}
```

### 小咲的玩具

[題目連結](https://neoj.sprout.tw/problem/722/)
這一題的常數超緊，因此在嘗試使用*hash*時用*unordredmap*被卡*TLE*，在很電的人提示後，要使用黑魔法cc_hash_table才可能會過關

```cpp=
#include <ext/pb_ds/assoc_container.hpp>
using namespace __gnu_pbds;
cc_hash_table<int,int>cc;
```

他的用法其實跟*unordered_map*幾乎一樣吧（至少基本的insert跟find的語法都一樣），不過他可以做到比*unordered_map*更好的效率。不過我用[最近點對](https://neoj.sprout.tw/problem/795/)這一題測試效率卻發現黑魔法會tle，可能是在clear這一個步驟的效率並不是很好吧！

總之，就是對於每一筆詢問(x,y)，不失一般性假設size_y > size_x，則對於y排序並預處理前綴和，同時枚舉x中的所有點並二分搜他在y的位置
最重要的，是對每一筆詢問都儲存起來，這樣複雜度會瞬間少一個N
**補複雜度證明**

```cpp=
#include <bits/stdc++.h>
using namespace std;
#include <ext/pb_ds/assoc_container.hpp>
using namespace __gnu_pbds;
#define int long long
#define N 150006

int n,k,q;
vector<int> vec[N],pre[N];
bool visit[N];
cc_hash_table<int,int>cc;
 
int HASH(int x,int y){
    return x*1000000007+y;
}
signed main(){
    scanf("%lld %lld %lld",&n,&k,&q);
    memset(visit, 0, sizeof(visit));
    while(k--){
        int c,p;scanf("%lld %lld",&c,&p);
        vec[p].push_back(c);
    }
    while(q--){
        int x,y;scanf("%lld %lld",&x,&y);
        if(vec[x].size()>vec[y].size())swap(x, y);
        
        if(cc.find(HASH(x,y))!=cc.end()){
            printf("%lld\n",cc.find(HASH(x,y))->second);
            continue;
        }
        else if(cc.find(HASH(y,x))!=cc.end()){
            printf("%lld\n",cc.find(HASH(y,x))->second);
            continue;
        }
        
        int len = vec[y].size();
        if(!visit[y]){
            sort(vec[y].begin(),vec[y].end());
            pre[y].resize(len+6);
            for(int i=0;i<=len;i++)pre[y][i+1] = pre[y][i]+vec[y][i];
            visit[y] = 1;
        }
        
        int ans = 0;
        for(int j:vec[x]){
            int pos = lower_bound(vec[y].begin(), vec[y].end(),j)-vec[y].begin();
            pos--;
            if(pos<0)ans+=(len*j);
            else ans += (pre[y][pos+1]+j*(len-pos-1));
        }
        printf("%lld\n",ans);
        cc.insert(make_pair(HASH(x,y),ans));
        cc.insert(make_pair(HASH(y,x),ans));
    }
}
```

## 手寫作業

這已經是最後的一堂課了qq，13週的課程就這樣結束了！
![](https://i.imgur.com/o8xCu1i.gif)
![](https://i.imgur.com/tevltPd.png)
手寫作業排名12，上機不知道，但二階段因為剛好二段的關係，有一個禮拜（dp3）的課的上機作業全部放掉（我在幹嘛qq）那真是慘啊！
