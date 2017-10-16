# 处理直流电阻率法的边界条件

通过模拟，发现边界条件对直流电阻率法的影响比较大。
仅用第二类边界条件， $\frac{{\partial}u}{{\partial}n}=0$ 或第一类边界条件，$u=0$ 结果误差均较大，使用第二类边界条件，在网格和电阻率一定（合适）的条件下，结果可能会好一些，当使用第三类边界条件 $u|_{{\Gamma}_{\infty}}=\frac{c}{r{'}}$ 也即 $\frac{{\partial}u}{{\partial}n}+\frac{cos(r,n)}{r}=0$ ，选取合适网格，尽管使用总场法，结果误差依然较小（3%以内），所以在实际计算中，采用第三类边界条件是很有必要的，这与我在硕士论文中提到的简化边界条件的结果并不一样，与黄俊革博士论文中的结果也有一定差异，他得到的结果是齐次边界条件下得到的结果与非齐次边界条件基本一致，在研究区域误差均在5%以内。特别是三维模拟，我在对比带初值的CG法时发现带边界条件的总场法模拟结果总是较好的。



## 二维边界条件

处理二维边界条件，在生成网格时，即输出边界信息，

```Fortran
	...
	open(123,file='./bd.frag')
	N=0
	DO IX=1,NX
	DO IZ=1,NZ
		N1= (IX   -1)*(NZ+1)+ IZ
		N2= (IX   -1)*(NZ+1)+(IZ+1)
		N3=((IX+1)-1)*(NZ+1)+(IZ+1)
		N4=((IX+1)-1)*(NZ+1)+ IZ
		...
		...
		! --------------------------------------------
		! output the boundary fragments
		if(ix==1) then
			write(123,*)    N1,N2,N
		end if
		if(iz==nz) then
			write(123,*)    N2,N3,N
		end if
		if(ix==nx) then
			write(123,*)    N4,N3,N
		end if
		! --------------------------------------------
	END DO
	END DO
	close(123)
	...
```
在总体矩阵的单元分析之后，加入边界条件的单元，其中关键步骤为计算 $cos(r,n),|r|$ ，程序为：

```Fortran
	subroutine get_r_cos(ma,m,n,xy,r,alf_cos)
		implicit none
		integer	ma,m,n
		real	xy(2,*),r,alf_cos
		real	midx,midy
	    real    a(2),b(2),ab,r_a,r_b
	    integer i,j,nz

		midx = (xy(1,m)+xy(1,n))/2.0
		midy = (xy(2,m)+xy(2,n))/2.0

		r = sqrt( (midx-xy(1,ma))**2 + (midy-xy(2,ma))**2 )

	    a(1)=xy(1,m)-xy(1,n)
	    a(2)=xy(2,m)-xy(2,n)
	    if( abs(a(1)) .ge. 10e-3 ) then
	        nz=1
	    else
	        nz=2
	    end if
	    b(:)=1.0;b(nz)=0.0;
	    ab=a(1)*b(1)+a(2)*b(2)
	    b(nz)=-ab/a(nz)

	    a(1)=midx-xy(1,ma);a(2)=midy-xy(2,ma);
	    r_a = sqrt(a(1)**2+a(2)**2 )
	    r_b = sqrt(b(1)**2+b(2)**2 )
	!    cos<a,b>=(a*b)/(abs(a)*abs(b))
	!    where a,b is vector, a*b means dot(a,b)
	    alf_cos=abs( (a(1)*b(1)+a(2)*b(2) )/(r_a*r_b)  )

	end subroutine get_r_cos
```

该程序仅适合二维情况，其中计算边界的外法向为核心
计算过程参照[网上文章所示](http://bbs.csdn.net/topics/330007262)

思路为：
1.在第一个向量中找到一个不为0的数a[i]，如果全为0，则这个向量为0向量，作何其它一个向量都和其垂直。
2.令b中除了b[i]外全置为1(这个数可以变).b[i]=0;
3.求出sum = a*b;i是a 中的一个不为0的数的下标
4.b[i]=-sum/a[i];
给出的c代码为：

```c
	int a[10];// 存放一个向量
	int b[10];//存放另一个向量
	for(int i=0; i<lenofver; i++)//lenofver向量长度
	     if(a[i]) break;
	if(i==lenofver)
	{
	     printf("任一个向量都于所求向量垂直");
	     return ;
	}

	for(int j=0; j<lenofver; j++)
	{
	      if(i!=j) b[j]=1;
	}

	int sum=0;

	for(int k=0; k<lenofver; k++)
	{
	      if(i!=k) sum += a[k]*b[k];
	}
	b[i] = -sum/a[i];
```


## 三维边界条件
