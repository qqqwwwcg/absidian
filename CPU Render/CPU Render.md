## SRT(scale,rotation,translation)
Scale:
$$\begin{matrix}
Sx&0&0\\
0&Sy&0\\
0&0&Sz\\
\end{matrix}$$
Rotation:
欧拉角旋转
	拆解成绕XYZ的三个旋转矩阵的叠加
	ex:绕X轴旋转，此时退化到2D，在YZ坐标系，绕X轴旋转，矩阵易得，且矩阵X方向默认即可
	2D：
		$$\begin{matrix}
cos\alpha&-sin\alpha\\
sin\alpha&cos\alpha\\
\end{matrix}$$
	3D：
	$$\begin{matrix}
	1&0&0\\
0&cos\alpha&-sin\alpha\\
0&sin\alpha&cos\alpha\\
\end{matrix}$$
	Rotatain = Ma * Mb * Mc
TODO：
		2D的旋转矩阵推导，后续补充
		对顶点的坐标变化，实际上是对其模型坐标系进行变化 见3blue1brown
		万向节死锁问题

TODO：四元数旋转 3blue1brown

TODO：罗德里格斯旋转

Translation
	将SR的非齐次坐标，变为齐次坐标，得到放射矩阵(==线性变换==)

注：SRT的变换顺序，先进行缩放，再绕自身坐标系进行旋转，最后平移坐标系
TODO：旋转矩阵的不会进行缩放，模长为1 知识连动到点云配准的旋转平移矩阵

## MVP
M:model transform 从模型坐标系变换到世界坐标系
V：view tranform 从世界坐标系变换到相机坐标系
P：project transform 将相机坐标系下的点 投影到 NDC空间

NDC后，再投影到屏幕平面

TODO: 知识连动至GAMES101 正交投影 透视投影
注：后续会进行view prot裁剪等

## 屏幕裁剪


## 直线光栅化
### 暴力方法
y = kx +b ，通过带入xi (==int==)，计算得到相应的yi,再对yi进行取整,得到相应像素点
**ps: yi的取整，int(yi+0.5)，数学意义上进行四舍五入**
### DDA
核心思想：通过增量计算，替代yi = xi * k + b中的乘法计算(xi * k)
①计算斜率
	k = (y2-y1) / (x2-x1) (x1 ≠ x2)
②增量计算
	当|K|  <= 1时，选取x轴为主要步进方向
	X i+1  = Xi +1
	Y i+1 = Yi + k，此时避免了yi = xi * k +b中的乘法计算
	当|K| > 1时，选取y轴为主要步进方向
	Y i+1 = Yi +1
	X i+1 = Xi + 1/k
	当|K| = ∞，即x1 = x2，说明此时是直线
	Y i+1 = Yi +1
tips1: 为什么是|K|，因为0<k<1,和-1<k<0，流程一样，x都是主方向
tips2:为什么要区分主要步进方向
当k>1是，如果仍选取x为主要不仅方向，会失真
本来应该有5个像素点，现在仅有3个
![[DDA.png]]
[(86条消息) DDA画线算法+代码详解-直线扫描算法之一_dda算法例题及解题思路_热带宇林V的博客-CSDN博客](https://blog.csdn.net/u013378269/article/details/103555482)

### 中点画线法
DDA的运算过程中，有很多浮点数的运算，中点画线法中去只有整数变量，并且去除了乘除法
核心思想：
	假设x为主要步进方向，Pixel(Xi,Yi)的下一个点只有两种可能Pixel(Xi + 1, Yi)或者 Pixel(Xi +1, Yi + 1)
	这取决于两点中点：Pixel(Xi +1, Yi+0.5)落在直线的上方还是下方
	通过将Xi +1, Yi + 0.5代入直线的一般式来判断f(x，y)的符号，
	f>0：中点落在直线上方，取Pixel(Xi + 1, Yi)
	f=0：任取一点
	f<0:   中点落在直线下方，取Pixel(Xi + 1, Yi + 1)
	注：此时的一般式中，==y的系数要>0==
	f(x,y) = (y2 -y1)*x + (x2 - x1)*y + x1*y2 - x2*y1=0

增量更新：
	对于每次都需要计算 f （xi + 1, yi + 0.5）
	其实，在计算 f （xi + 1, yi + 0.5）之前，已经计算过 delta = f(xi，yi-0.5)
	delta = delta + (y2-y1)+(x2-x1)，化乘法为加法
	同时，delata只参与符号判断，其中存在0.5这个浮点数，因此可以用2 * delta参与运算 去除浮点数
	再进一步 将2 * int，用位运算int << 1来实现，化乘法为位运算
	至此，bresenham中只存在整数的加减和位运算

[(86条消息) 【计算机图形学】画线算法——中点画线算法_地泽万物的博客-CSDN博客](https://blog.csdn.net/qq_41883085/article/details/102730878)

### Bresenham
bresenham本质是中点画线的增量更新版本，bresenham中只存在整数的加减和位运算

ex：假定绘制的直线，1.起点小于终点，2. 0 < k <1
	对于给定的起点(x1,y1), 终点(x2,y2)，x1<x2, y1 < y2
	k = (y2-y1) / (x2-x1)
	delta = d0 = 0 在y方向的投影的变化量
	由于是屏幕像素，所以x每次+1,delta每次+k (0<k<1)，将delta与0.5比较，大于则y+1,delta -=1, 否则y不变
![[bresenham1.jpg]]



伪代码：
```
let dx = x2 - x1;
let dy = y2 - y1;
let k = dy/dx;
let mut a = 0;

let mut x = x1;
let mut y = y1;

while x <= x2{
	drawPixel(x,y);
	
	x +=1;
	a += k;
	if a > 0.5{
		y +=1;
		a -=1;
	}
}
```
优化0.5的浮点数
a > 0.5 => a * 2 -1 > 0, 
let b = a * 2 -1
优化k的除法
let mut delta = b * dx = (0 * 2 -1) * dx = -dx
a = a+k => (2 * a +1) * dx  = (2 * (a + k) +1) * dx => delta = delta + 2 * dy
a > 0.5 => a - 0.5 >0 => (2 * (a -0.5)+1) * dx >0 => delta >0
a = a-1 => (2 * a +1) * dx  = (2 * (a -1) +1) * dx => delta = delta - 2 * dx
```
let dx = x2 - x1;
let dy = y2 - y1;
let mut delta = - dx;

let mut x = x1;
let mut y = y1;

while x <= x2{
	drawPixel(x,y);
	
	x +=1;
	delta += 2 * dy;
	if delta > 0{
		y +=1;
		delta -= 2 *dx;
	}
}
```
优化乘法  2 * dx 2 * dy：
```
let dx = x2 - x1;
let dy = y2 - y1;
let kx = 2dx = dx <<1;
let ky = 2dy = dy <<1;
let mut delta = - dx;

let mut x = x1;
let mut y = y1;

while x <= x2{
	drawPixel(x,y);
	
	x +=1;
	delta += ky;
	if delta > 0{
		y +=1;
		delta -= kx;
	}
}
```

上述仅为 1.起点小于终点，2. 0 < k <1的情况，需要对下图的8中情况都包括
![[bresenham.jpg]]
首先，通过对P1、P2进行交换，保证x1 < x2，则上图的左边四种情况被过滤掉
其次，与DDA相似，0< k < 1 与 -1< K<0，流程一样，仅仅是dy的方向不同
最后，对于|K|<1,|k >1，x，y的步进方向不同，仅对此两种情况进行流程区分
完整代码：参考RenderHelp
```
if x1 == x2 & y1 == y2{
	draw_pixel(x1,y1);
	return;
}

let mut x=x1;
let mut y=y1;
if x1 == x2{
	let inc = if y1 < y2{ 1 } else { -1 };
	while y != y2{
		draw_pixel(x,y)
		y += inc;
	}
	draw_pixel(x,y)
}else if y1==y2{
	let inc = if x1<x2{1} else{-1};
	while x != x2{
		draw_pixel(x,y)
		x +=inc;
	}
	draw_pixel(x,y)
}else{
	let dx = if x2 > x1 {x2 - x1} else{x1 -x2}; dx>0
	let dy = if y2 > y1 {y2 - y1} else {y1 - y2}; dy >0
	let kx = 2dx = dx <<1;
	let ky = 2dy = dy <<1;

	if dx > dy{  |K|<1,x为主要步进方向
		保证x1 < x2
		if x1 > x2{
			(x1,y1) . swap (x2,y2);
			x=x1;
			y=y1;	
		}
		let mut delta = - dx;
		let inc = if y1 < y2 {1} else{-1}；
		
		while x != x2{
			draw_pixel(x,y);

			x +=1;
			delta += inc * dy;
			if delta.abs() >0{
				y += inc;
				delta += inc * kx;	
			}
		}
		draw_pixel(x,y);
	}else{    |K|>1,y为主要步进方向
		保证y1 < y2
		if y1 > y2{
			(x1,y1) . swap (x2,y2);
			x=x1;
			y=y1;	
		}
		let mut delta = - dy;
		let inc = if x1 < x2 {1} else{-1}；
		
		while y != y2{
			draw_pixel(x,y);

			y +=1;
			delta += inc * dx;
			if delta.abs() >0{
				x += inc;
				delta += inc * ky;	
			}
		}
		draw_pixel(x,y);
	}
}
```
[skywind3000/RenderHelp: 可编程渲染管线实现，帮助初学者学习渲染 (github.com)](https://github.com/skywind3000/RenderHelp)
[Bresenham 直线算法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/106155534)

## 三角面片光栅化