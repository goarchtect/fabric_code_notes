# Fabric 1.0源码之旅(3)-椭圆曲线算法
BCCSP模块中，涉及椭圆曲线算法，本文专门讲解。
椭圆曲线算法go标准库即支持，如下均为标准库代码。
## 1、椭圆曲线算法概论
### 1.1、无穷远点、无穷远直线、射影平面
* 平行线相交于无穷远点；
* 直线上有且只有一个无穷远点；
* 一组相互平行的直线有公共的无穷远点；
* 平面上任何相交的两直线，有不同的无穷远点；
* 全部无穷远点沟通一条无穷远直线；
* 平面上全部无穷远点和全部普通点构成射影平面。
### 1.2、射影平面点定义
对于普通平面上点(x, y)，令x=X/Z，y=Y/Z，Z≠0，则投影为射影平面上的点为(X : Y : Z)。
如点(1，2)在射影平面的坐标为：(Z : 2Z : Z) Z≠0，即(1 : 2 : 1)或(2 : 4 : 2)均为(1, 2)在射影平面上的点。
Z=0时，(X : Y : 0)即为无穷远点，Z=0即为无穷远直线。
### 1.3、椭圆曲线方程
椭圆曲线的定义：
一条椭圆曲线是在射影平面上满足方程Y²Z+a1XYZ+a3YZ²=X³+a2X²Z+a4XZ²+a6Z³的所有点的集合，且曲线上的每个点都是非奇异（或光滑）的。
该方程为维尔斯特拉斯方程，是一个齐次方程。
所谓“非奇异”或“光滑”的，即满足方程的任意一点都存在切线。

椭圆曲线存在无穷远点(0, Y, 0)，可以在平面坐标系中用椭圆曲线、加一个无穷远点来表示。
令x=X/Z，y=Y/Z，代入椭圆曲线方程，即椭圆曲线普通方程：y²+a1xy+a3y = x³+a2x²+a4x+a6。
### 1.4、椭圆曲线上的加法
任意取椭圆曲线上两点P、Q （若P、Q两点重合，则做P点的切线）做直线交于椭圆曲线的另一点R’，过R’做y轴的平行线交于R。我们规定P+Q=R。

根据这个法则，可以知道椭圆曲线无穷远点O∞与椭圆曲线上一点P的连线交于P’，过P’作y轴的平行线交于P，所以有 无穷远点 O∞+ P = P 。
这样，无穷远点 O∞的作用与普通加法中零的作用相当（0+2=2），我们把无穷远点 O∞ 称为 零元。同时我们把P’称为P的负元（简称，负P；记作，-P）。

根据这个法则，可以得到如下结论 ：如果椭圆曲线上的三个点A、B、C，处于同一条直线上，那么他们的和等于零元，即A+B+C= O∞ 。
k个相同的点P相加，我们记作kP。如：P+P+P = 2P+P = 3P。
### 1.5、有限域椭圆曲线
椭圆曲线是连续的，并不适合用于加密；所以，我们必须把椭圆曲线变成离散的点，我们要把椭圆曲线定义在有限域上。
* 我们给出一个有限域Fp
* Fp中有p（p为质数）个元素0,1,2,…, p-2,p-1
* Fp的加法是a+b≡c(mod p)
* Fp的乘法是a×b≡c(mod p)
* Fp的除法是a÷b≡c(mod p)，即 a×b^(-1)≡c (mod p)，b^(-1)也是一个0到p-1之间的整数，但满足b×b^(-1)≡1 (mod p)
* Fp的单位元是1，零元是0

同时，并不是所有的椭圆曲线都适合加密。y²=x³+ax+b是一类可以用来加密的椭圆曲线，也是最为简单的一类。
下面我们就把y²=x³+ax+b这条曲线定义在Fp上：

选择两个满足下列条件的小于p(p为素数)的非负整数a、b，4a³+27b²≠0　(mod p) 。
则满足下列方程的所有点(x,y)，再加上 无穷远点O∞ ，构成一条椭圆曲线。 
y²=x³+ax+b  (mod p) 其中 x,y属于0到p-1间的整数，并将这条椭圆曲线记为Ep(a,b)。

Fp上的椭圆曲线同样有加法，但已经不能给以几何意义的解释。
```
无穷远点 O∞是零元，有O∞+ O∞= O∞，O∞+P=P 
P(x,y)的负元是 (x,-y)，有P+(-P)= O∞ 
P(x1,y1),Q(x2,y2)的和R(x3,y3) 有如下关系： 
　　x3≡k2-x1-x2(mod p) 
　　y3≡k(x1-x3)-y1(mod p) 
　　其中若P=Q 则 k=(3x1²+a)/2y1  若P≠Q，则k=(y2-y1)/(x2-x1)
```

例 已知E23(1,1)上两点P(3,10)，Q(9,7)，求1)-P，2)P+Q，3) 2P。
```
1)  –P的值为(3,-10) 
2)  k=(7-10)/(9-3)=-1/2，2的乘法逆元为12 因为2*12≡1 (mod 23) 
	k≡-1*12 (mod 23) 故 k=11。 
	x=112-3-9=109≡17 (mod 23); 
	y=11[3-(-6)]-10=89≡20 (mod 23) 
	故P+Q的坐标为(17,20) 
3)  k=[3(3²)+1]/(2*10)=1/4≡6 (mod 23) 
	x=62-3-3=30≡20 (mod 23) 
	y=6(3-7)-10=-34≡12 (mod 23) 
	故2P的坐标为(7,12) 
```