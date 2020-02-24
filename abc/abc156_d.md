
今日の見直し問題は ABC156 D -Bouquet です。

問題URL:https://atcoder.jp/contests/abc156/tasks/abc156_d  
解説URL:https://img.atcoder.jp/abc156/editorial.pdf#page=7

AtCoderによる解説（PDF）はシンプルなので、自分なりに噛み砕いてまとめてみます。

> #### 問題文  
> あかりさんは $n$ 種類の花を $1$ 本ずつ持っています。  
> あかりさんは、これらの花から $1$ 本以上を選び、花束を作ろうとしています。  
> ただし、あかりさんは $a$ と $b$ の $2$ つの数を苦手としていて、いずれかと一致するような本数の花からなる花束は作ることができません。  
> あかりさんが作ることのできる花束は何種類あるでしょうか。
$(10^9+7)$ で割った余りを求めてください。
>ここで $2$ つの花束は、一方では使われているが、 もう一方では使われていない種類の花があるとき、別の種類の花束であるとみなします。
>
> #### 制約
>- 入力は全て整数である。
>- $2 \leq n \leq 10^9$
>- $1 \leq a < b \leq \min(n, 2 \times 10^5)$



# step 1: 答えを $\bmod (10^9+7)$ で割った値の求め方
今回の計算では、最終的な答えのみならず計算途中でも非常に大きな値を扱うことになります。
C++におけるサイズ最大の整数型 long long 型（範囲：$-2^{64} 〜 2^{64-1}$）を使っても、 $2^n$ においてはたった $n \geq 64$ で収まらなくなります。  

そこで、数値 $a$ と数値 $b$ の演算の過程で、結果を型の大きさ内に収めつつ、最終的な答えまで $\bmod (10^9+7)$ を保つことができる操作を知る必要があります。

今回使用する演算の操作を以下にまとめました。

| 演算  |	操作 |
|-----|--------|---------------------|
|足し算 $a+b$ | 計算途中でもあまりをとってよい|	
|掛け算 $a \times b$ |計算途中でもあまりをとってよい|	
|引き算 $a−b$|計算途中でもあまりをとってよい<br>ただし最終結果が負になるなら $10^9+7$ を足す|	

参考：https://qiita.com/drken/items/3b4fdf0a78e7a138cd9a

今回の問題では$\bmod$ が $10^9+7$ なので、最悪の場合で以下のような計算が出てきたとしても、

$$(10^9+6 \pmod{10^9+7}) \times (10^9+6 \pmod{10^9+7})$$
$$=(10^9+6) \times (10^9+6)$$
$$=1000000012000000036$$
$$\leq 2^{64-1} = 9223372036854775807$$

となり、演算結果は一時的に long long 型に収めることができます。



# step 2: 解法の見通し
まずは、答えを $\bmod (10^9+7)$ で割る…は考えずに、解法を考えます。

まず、「 $a$, $b$ 本の花からなる花束は作ることができない」という制約を無視して問題を解きます。  
$n$ 種類の各花について「使う」「使わない」の $2$ 択があると考えると、作ることのできる花束は $2^n$ 種類です（公式 $\sum_{k=0}^n {}_n \mathrm{C} _k = 2^n$ からもわかる）。

ただし、$1$ 本以上の花を選ばなければいけないとあるので、1本も選ばない場合 ${}_n \mathrm{C} _0$ を除きます。
したがって、作ることのできる花束の種類は $2^n - {}_n \mathrm{C} _0$ です。

$a$ 本の花からなる花束は、${}_n \mathrm{C} _a$ 種類、$b$ 本の花からなる花束は${}_n \mathrm{C} _b$ 種類なので、「 $a$, $b$ 本の花からなる花束は作ることができない」という制約を考慮した答えは、
$$2^n - {}_n \mathrm{C} _0 - {}_n \mathrm{C} _a-{}_n \mathrm{C} _b$$
となります。

step 1 より、引き算の過程でmodをとっても構いません（ただし最終結果が負になるなら $10^9+7$ をたします）。



# step 3: 繰り返し二乗法

step 2 では、$n = 10^9$ となるときの $2^n \bmod (10^9 + 7)$ の計算が問題となります。
$n = 10^5$ 程度では、以下のような $O(n)$ の関数 mod_pow_1 でも $2^n \bmod (10^9 + 7)$ を計算できますが、$n = 10^9$ では間に合いません。

```c++
int pow_mod_1(long long x, long long n, long long mod)
{
    long long ans = 1;
    for(long long i = 0; i < n; i++)
    {
        ans *= x;
        ans %= mod; //step 1 参照。掛け算においては計算途中でmodをとってよい。
    }
    return ans;
}
```

そこで用いるのが、繰り返し自乗法と呼ばれるテクニックです。

以下の関数 mod_pow_2 では、 $x$ の $n$ 乗を $O(\log n)$ で計算することができます。

```c++
long long mod_pow_2(long long x, long long n)
{
    if (n == 0)
    {
        return 1;
    }
    if (n % 2 == 0)
    {
        return mod_pow_2(x * x, n / 2);
    }
    return (x * mod_pow_2(x, n - 1));
}
```

$2^{10}$ を求める例で考えてみると、上記の関数では、実質的には以下のような手順で計算していることになります。

$$2^{10} = (2^5)^2$$
$$= 4 \times (2^4)^2$$
$$= 4 \times ((2^2)^2)^2$$
$$= 4 \times (((2^1)^2)^2)^2$$
$$= 4 \times 256 \times (((2^0)^2)^2)^2$$
$$= 1024$$ 

ただし、このままでは $\bmod$ がとれていないし、 $n \geq 64$ 程度で対応できなくなるので、以下のように改良を加えます。

```c++
int mod_pow_3(long long x, long long n, long long mod)
{
    if (n == 0)
    {
        return 1;
    }
    if (n % 2 == 0)
    {
        return mod_pow_3(x * x % mod, n / 2, mod) % mod;
    }
    return (x * mod_pow_3(x, n - 1, mod)) % mod;
}
```
参考：http://satanic0258.hatenablog.com/entry/2016/04/29/004730

$2^{10} \bmod 100$ を求める例で考えてみると、上記の関数 mod_pow_3 では、実質的には以下のような手順で計算していることになります。
$$ 2^{10} \bmod 100 = ((2^5)^2) \bmod 100$$
$$= (4 \bmod 100 \times (2^4)^2) \bmod 100$$
$$= (4 \bmod 100 \times ((2^2)^2)^2) \bmod 100$$
$$= (4 \bmod 100 \times (((2^1)^2)^2)^2) \bmod 100$$
$$= (4 \bmod 100 \times 256 \bmod 100 \times (((2^0)^2)^2)^2) \bmod 100$$
$$= (4 \times 56 \times 1) \bmod 100$$
$$= 224 \bmod 100$$
$$=24$$ 

# step 3: ${}_n \mathrm{C} _k$ の計算


次は、${}_n \mathrm{C} _k$ の求め方を考えます。

$k$ 本の花からなる花束の種類数 ${}_n \mathrm{C} _k \pmod{10^9 + 7}$ は、

$$
{}_n \mathrm{C} _k \bmod{10^9 + 7} = \frac{n \times (n − 1) \times \cdots \times (n − k + 1)}{k \times (k − 1) \times \cdots \times 1}\bmod{10^9 + 7}\\
=\frac{{}_n \mathrm{P} _k}{{}_k \mathrm{P} _k}\bmod{10^9 + 7}
$$

です。このとき、$X={}_n \mathrm{P} _k$、$Y={}_k \mathrm{P} _k$ と置くと、

$$\frac{X}{Y} \bmod{10^9 + 7} = X \times Y^{-1} \bmod{10^9 + 7}\tag{1}$$

と変形できます。

$X={}_n \mathrm{P} _k$、$Y={}_k \mathrm{P} _k$ は、以下の関数を用いて $O(k)$ で知ることができます。

```c++
long long permutation(long long x, long long k, long long mod)
{
    long long ans = 1;

    for (long long i = 0; i < k; i++)
    {
        ans *= x - i;
        ans %= mod; //step 1 参照。掛け算においては計算途中でmodをとってよい。 
    }

    return ans;
}
```

ただし、このままでは $Y^{-1} \bmod{10^9+7}$ を求めることが困難です。
そこで、 $\bmod{10^9+7}$ の世界において、 $Y^{-1}$ と合同である整数を探します。

フェルマーの小定理
> $p$ を素数とし、$a$ を $p$ の倍数でない整数（$a$ と $p$ は互いに素）とするときに、
> $$a^{p-1}\equiv 1 \pmod{p} $$  
引用元：https://ja.wikipedia.org/wiki/%E3%83%95%E3%82%A7%E3%83%AB%E3%83%9E%E3%83%BC%E3%81%AE%E5%B0%8F%E5%AE%9A%E7%90%86

を用いると、$10^9 + 7$ が素数であることから、

$$Y^{(10^9+7)−1} \equiv 1 \pmod{10^9 + 7}$$

が成り立ちます。  
両辺に $Y^{−1}$ を掛けると、

$$Y^{(10^9+7)−2} \equiv Y^{−1} \pmod{10^9 + 7}$$

したがって、

$$X \times Y^{-1} \equiv X × Y^{(10^9+7)−2} \pmod{10^9 + 7}$$

を得ることができました。

$Y^{(10^9+7)−2} \bmod{10^9+7}$ の値は、繰り返し二乗法を用いて求めることができます。

以上で、${}_n \mathrm{C}_k \bmod{10^9+7}$ の値を求めることができるようになりました。

$n$ 、$k$ 、$\bmod$ の値を入力することで ${}_n \mathrm{C}_k \bmod{10^9+7}$ の値を出力する関数を以下に示します。

```c++
long long combination(long long n, long long k, long long mod)
{
    long long X, Y;

    X = permutation(n, k, mod);
    Y = permutation(k, k, mod);

    long long inverse_Y = mod_pow(Y, mod - 2, mod);

    return (X * inverse_Y) % mod;
}
```


# まとめ
今回のポイントは

- 繰り返し自乗法
- 指数関数の$\bmod(10^9+7)$の求め方

でした。

acannieのcode: https://github.com/acannie/AtCoder/blob/master/abc/abc156/d.cpp

ご覧いただきありがとうございました。