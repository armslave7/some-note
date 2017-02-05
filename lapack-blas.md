# 关于blas和lapack的安装和使用

下载源代码，在Linux平台下，通过make工具即可编译连接出静态链接库*liblapack.a  librefblas.a  libtmglib.a*文件
在编译过程中，lapack默认认为已经包含了blas库，所以默认会编译出错，首先```make blaslib```就可以了。

编译成功之后，make会自动开始测试，此过程可能会出错，但是并不影响，如果要完成测试，需要设置程序栈上限，通过```sudo ulimit```即可。

将静态链接库拷贝到*/usr/local/lib*中

在编译用到改库的程序的make或者链接代码中添加```-llapack -lrefblas```即可。

例如

```fortran
	! blas_test.f90
	program main
	    integer,parameter :: n=100
	    integer i
	    real(kind=8)    a(n),b(n),x(n)

	    do i=1,n
	        a(i)=i;b(i)=i;
	    end do

	    call dcopy(n,a,1,b,1)

	    return
	end program
```

输入```gfortran blas_test.f90 -lrefblas```即可
