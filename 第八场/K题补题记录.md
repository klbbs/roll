# G题补题记录

### 根据AI补题得到思路

1. #### 先特判,如果连通块数量>2,直接输出0,如果连通块数量等于2,输出两个集合大小相乘再乘以k

2. #### 由于存在最小生成树,我们可以统计出n*(n-1)/2条边都可以填充,计算出这个方案数

3. #### 我们知道肯定是有重边不行的,所有遍历所有边,把重边的贡献减去即可

```cpp
#include <bits/stdc++.h>
#define endl '\n'
#define int long long
#define vii vector<vector<int>>
#define vi vector<int>
using namespace std;
typedef long long LL;
typedef pair<int,int> PII;


const int INF = 0x3f3f3f3f;
const int mod = 1e9 + 7;
const int N = 1e5+10;

int lg[N];
class solution
{
    public:
        void solve();
        void ycl();
        solution(){ycl();};
};

void solution::ycl()
{
    for(int i=2;i<N;i++)
        lg[i] = lg[i/2] + 1;
}

/*
 *    from jiangly
 *    整理 by https://www.cnblogs.com/WIDA/p/17633758.html
*/

/**   并查集（DSU）
 *    2023-08-04: https://ac.nowcoder.com/acm/contest/view-submission?submissionId=63239142
**/
struct DSU {
    std::vector<int> f, siz;
    
    DSU() {}
    DSU(int n) {
        init(n);
    }
    
    void init(int n) {
        f.resize(n);
        std::iota(f.begin(), f.end(), 0);
        siz.assign(n, 1);
    }
    
    int find(int x) {
        while (x != f[x]) {
            x = f[x] = f[f[x]];
        }
        return x;
    }
    
    tuple<int,int,bool> same(int x, int y) {
        int fx = find(x),fy = find(y);
        return {fx,fy,fx==fy};
    }
    
    bool merge(int x, int y) {
        x = find(x);
        y = find(y);
        if (x == y) {
            return false;
        }
        if(siz[x] < siz[y])
            swap(x,y);
        siz[x] += siz[y];
        f[y] = x;
        return true;
    }
    
    int size(int x) {
        return siz[find(x)];
    }

    void clear()
    {
        for(int i=0;i<f.size();i++)
            f[i] = i,siz[i] = 1;
    }
};

void solution::solve()
{
    int n,m,k;cin>>n>>m>>k;
    struct edge
    {
        int u,v,w;
    };
    vector<edge>edges(m);
    DSU ds(n+1);
    for(auto &[u,v,w]:edges)cin>>u>>v>>w,ds.merge(u,v);
    set<int>cnt;
    // 找有多少个集合
    for(int i=1;i<=n;i++)
        cnt.insert(ds.find(i));
    {
        int len = cnt.size();
        if(len > 2)
        {
            cout<<0<<endl;
            return;
        }
        if(len > 1)
        {
            LL ans = 1;
            for(auto i:cnt)
                ans = ans*ds.siz[i]%mod;
            ans = ans*k%mod;
            cout<<ans<<endl;
            return;
        }
    }
    sort(edges.begin(),edges.end(),[&](auto &eu,auto &ev){
        return eu.w < ev.w;
    });
    LL all = 0;
    LL all_edges = (LL)n*(n-1)/2;
    LL has = 0;
    ds.clear();
    int lastw = 0;
    for(int i=0;i<m;i++)
    {
        auto &[u,v,w] = edges[i];
        int cw = w - 1;
        if(lastw < cw)
        {
            LL t = (all_edges - has)%mod;
   			/*
   			* 因为最小生成树包含所有点,假设我们已经找到了最短生成树的某些点,这些点都俩俩可以互相连接一条边,
   			* 这些边都不用讨论,因为之前加了这些贡献
   			* 那么那些没有被找到的点也可以俩俩添加一条边,但是由于最短生成树上两点路径上边权不知道,假设为某条边权w
   			* 由于边权已经排序,我们可以通过这种cw-lastw一直把它的贡献加满
   			*/
            all = (all + t*(cw-lastw)%mod)%mod;
        }
        int fu,fv;bool fg;
        tie(fu,fv,fg) = ds.same(u,v);
        if(!fg)
        {
            has = (has + (LL)ds.siz[fu]*ds.siz[fv]);
            ds.merge(u,v);
        }
        lastw = cw;
    }
    // 最后has肯定是等于all_edges的,不需要下面的代码
    //if(k > lastw)
    //    all = (all + (all_edges - has)*(k-lastw)%mod)%mod;

    vector<vector<PII>>g(n+1);
    ds.clear();
    auto add = [&](int u,int v,int w)
    {
        g[u].emplace_back(v,w);
        g[v].emplace_back(u,w);
    };
    auto build_g = [&]()
    {

        for(auto &[u,v,w]:edges)
        {
            int fu,fv;bool fg;
            tie(fu,fv,fg) = ds.same(u,v);
            if(!fg)
            {
                add(u,v,w);
                ds.merge(u,v);
            }
        }
    };
    // 把最小生成树的图建出来,我们要用到lca求最小生成树两点路径上最大权值边
    build_g();

    int mx = lg[n] + 1;
    vii lca(mx+1,vi(n+1,0)); 
    vii mx_w(mx+1,vi(n+1,0));
    vi dep(n+1,0);
    function<void(int,int,int)>dfs = [&](int fa,int u,int lw)
    {
        // cout<<u<<' ';
        dep[u] = dep[fa] + 1;
        mx_w[0][u] = lw;
        lca[0][u] = fa;
        for(auto &[v,w]:g[u]) 
        {
            if(v == fa)continue;
            dfs(u,v,w);
        }
    };
    dfs(0,1,0);
    // cout<<"debug"<<endl;
    for (int i = 1; i < mx; ++i) {
        for (int j = 1; j <= n; ++j) {
            if (lca[i - 1][j] != 0) {
                lca[i][j] = lca[i - 1][lca[i - 1][j]];
                mx_w[i][j] = std::max(mx_w[i - 1][j], mx_w[i - 1][lca[i - 1][j]]);
            }
        }
    }
    
    auto get = [&](int u,int v)
    {
        int res = 0;
        if (dep[u] < dep[v]) std::swap(u, v);
        // dep u > dep v

        // 跳到同层同时记录路径max边权
        for (int i = mx - 1; i >= 0; --i) {
            if (lca[i][u] != 0 && dep[u] - (1 << i) >= dep[v]) {
                res = std::max(res, mx_w[i][u]);
                u = lca[i][u];
            }
        }

        if (u == v) return res;

        // 同时上跳
        for (int i = mx - 1; i >= 0; --i) {
            if (lca[i][u] != lca[i][v]) {
                res = std::max({res, mx_w[i][u], mx_w[i][v]});
                u = lca[i][u];
                v = lca[i][v];
            }
        }
        // lca找到的是公共祖先的子节点,所以还要res取max
        res = std::max({res, mx_w[0][u], mx_w[0][v]});

        // 返回最大值
        return res;
    };

//    cout<<get(1,5)<<endl;

    LL more = 0;
    for(auto &[u,v,w]:edges)
    {
        int val = get(u,v) - 1;
        if(val > 0)
        more = (more + val)%mod;
    }
    cout<< (all - more + mod)%mod <<endl;
}


signed main()
{
    ios_base::sync_with_stdio(false);cin.tie(nullptr);cout.tie(nullptr);
    int T = 1;
    solution *AC = new solution();
    //AC->ycl();
    cin>>T;
    while(T --)
    {
        AC->solve();
    }
    delete(AC);
    return 0;
}
```

