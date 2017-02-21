# 从网上摘录的用于求解复数的ssorcg程序---备忘

[链接](http://blog.sina.com.cn/s/blog_590bf7290100rdrq.html)


SSOR-PCG FORTRAN版CSR压缩存储

最近因为项目需要，自己写了个迭代的SSOR-PCG程序，基于上三角CSR压缩存储（或者是下三角CSC），我认为这些东西应该没有必要让大家重复开发，特发表在次，希望能够帮助到大家，但是，希望发表论文的时候，挂上网址吧。


```fortran
subroutine SSOR_PCG_CSR(ND,NNZ,A,B,X,irw,jcol,EPS,ITMAX,W,IT,reg_err)
!***********************************************************************************
!**   本程序参考林绍忠《用预处理共轭梯度法求解有限元方程组及程序设计》改进的迭代格式
!**    采用SSOR预处理，预处理矩阵为:M=(2-W)^{-1}*(D/W+L)*(D/W)^{-1}*(D/W+L)^{T}
!***********************************************************************************
implicit none
!***********************************************************************************
!**   输入和输出
!***********************************************************************************
integer,intent(in) :: ND,NNZ
integer,intent(in) :: irw(NNZ),jcol(ND+1) ! CSR压缩存储索引矩阵pointB,pointE
complex(8),intent(in) :: A(NNZ),B(ND)     !矩阵变量A*x=B
complex(8),intent(inout) :: X(ND)         !输入初始估计值和输出最终结果
real(8),intent(in) :: EPS                 !收敛误差
integer, intent(in) :: ITMAX              !最大迭代次数
real(8), intent(in) :: W                  !松弛因子
integer, intent(out) :: IT                !收敛总计迭代次数
real(8), intent(out) :: reg_err(ITMAX)    !每次迭代后的残差
!***********************************************************************************
!**   程序中间变量
!***********************************************************************************
complex(8) :: Y(ND),Z(ND),D(ND),V(ND),TEMP(ND)
integer :: i
complex(8) :: delta, tao, beta,delta1
complex(8), external :: m_dot_product
integer :: icol(NNZ),jrow(ND),ID(NNZ)
!***********************************************************************************
!**   设置初始值
!***********************************************************************************
forall(i=1:ND)
    V(i) = (2.0-W)*A(jcol(i))/W      !这里V当成了一维向量，实际上它是一个对角矩阵，及A的对角阵计算来
                                     !若想节约内存，可以把用到V的地方用A(jcol())来代替，这里我主要是为
                                     !了程序跟理论公式的一致性
end forall
call ID_CHANGE(ND,NNZ,IRW,JCOL,ICOL,JROW,ID)
!***********************************************************************************
!**   开始计算
!***********************************************************************************
call MatMulVec_CSR(ND,NNZ,A,X,Y,irw,jcol)
Y = Y - B
call forward_solve(ND,NNZ,A,Y,Y,ICOL,JROW,ID,W)
forall(i=1:ND)              !这里可以直接用Z=V*Y代替，主要目的是为了方便替换V,而且forall的效率不错^_^
    Z(i) = -V(i)*Y(i)     
end forall
call back_solve(ND,NNZ,A,Z,D,irw,jcol,W)
IT = 0
delta = m_dot_product(Y,V*Y,ND)
beta = 1
do i = 1,ITMAX
!    if( abs(delta) < EPS) then
!        return
!    endif
    tao = delta/m_dot_product(D,2.0*Z-V*D,ND)
    X = X + tao*D
! 多种收敛方式
    if( sqrt(sum(abs(tao*D/X)**2)) < EPS ) then
        return
    endif
    call forward_solve(ND,NNZ,A,Z-V*D,TEMP,ICOL,JROW,ID,W)
    Y = Y + tao*(TEMP+D)
    delta1 = m_dot_product(Y,V*Y,ND)
    beta = delta1/delta
    Z = -V*Y + beta*Z
    call back_solve(ND,NNZ,A,Z,D,irw,jcol,W)
    delta = delta1
    IT = IT + 1
    reg_err(IT) = abs(delta)
enddo

return
end subroutine SSOR_PCG_CSR
!================this is split line================================================!
subroutine MatMulVec_CSR(ND,NNZ,MAT,VEC,VEC_out,irw,jcol)
implicit none
!***********************************************************************************
!**   输入和输出
!***********************************************************************************
integer,intent(in) :: ND,NNZ
integer,intent(in) :: irw(NNZ),jcol(ND+1)   !CSR压缩存储索引矩阵pointB,pointE
complex(8),intent(in) :: MAT(NNZ),VEC(ND)   !矩阵变量MAT和向量VEC
complex(8) :: VEC_out(ND)              !输出最终结果
!***********************************************************************************
!**   程序中间变量
!***********************************************************************************
integer :: i,j
VEC_out = (0.0d0,0.0d0)
do i = 1 , ND
     VEC_out(i) = VEC_out(i) + MAT(jcol(i))*VEC(i)
    do j = jcol(i)+1, jcol(i+1)-1
        VEC_out(i) = VEC_out(i) + MAT(j)*VEC(irw(j))
        VEC_out(irw(j)) = VEC_out(irw(j)) + MAT(j)*VEC(i)
    enddo
enddo
return
end subroutine MatMulVec_CSR
!================this is split line================================================!
!***********************************************************************************
!** 对于对称上三角按行压缩存储的前推，需要进行每行的搜索，花费大量的时间，
!** 因此需要进行一下简单的重排，做一个映射数组，来影射到按列存储，见子程序ID_CHANGE。
!***********************************************************************************
subroutine forward_solve(ND,NNZ,MAT,VEC,VEC_out,icol,jrow,ID,W)
implicit none
!***********************************************************************************
!**   输入和输出
!***********************************************************************************
integer,intent(in) :: ND,NNZ
integer,intent(in) :: icol(NNZ),jrow(ND),ID(NNZ)   !CSR压缩存储索引矩阵pointB,pointE
complex(8),intent(in) :: MAT(NNZ),VEC(ND)   !矩阵变量MAT和向量VEC
real(8), intent(in) :: W
complex(8),intent(out) :: VEC_out(ND)              !输出最终结果
!***********************************************************************************
!**   程序中间变量
!***********************************************************************************
integer :: i,j,k
complex(8) :: sum

VEC_out(1) = VEC(1)/MAT(ID(jrow(1)))*W
do i = 2,ND
    sum = (0.0d0,0.0d0)
    if( i == icol(jrow(i-1)+1) ) then
        VEC_out(i) = (VEC(i)-sum)/MAT(ID(jrow(i-1)+1))*W
    else
        do j = jrow(i-1)+1,jrow(i)-1
            sum = sum + MAT(ID(j))*VEC_out(icol(j))
        enddo
        VEC_out(i) = (VEC(i)-sum)/MAT(ID(jrow(i)))*W
    endif
enddo
return
end subroutine forward_solve
!================this is split line================================================!
subroutine back_solve(ND,NNZ,MAT,VEC,VEC_out,irw,jcol,W)
implicit none
!***********************************************************************************
!**   输入和输出
!***********************************************************************************
integer,intent(in) :: ND,NNZ
integer,intent(in) :: irw(NNZ),jcol(ND+1)   !CSR压缩存储索引矩阵pointB,pointE
complex(8),intent(in) :: MAT(NNZ),VEC(ND)   !矩阵变量MAT和向量VEC
real(8), intent(in) :: W
complex(8),intent(out) :: VEC_out(ND)              !输出最终结果
!***********************************************************************************
!**   程序中间变量
!***********************************************************************************
integer :: i,j
complex(8) :: sum
do i = ND,1,-1
    sum = (0.0d0,0.0d0)
    if( i == irw(jcol(i+1)-1) ) then
        VEC_out(i) = (VEC(i)-sum)/MAT(jcol(i+1)-1)*W
    else
        do j = jcol(i+1)-1,jcol(i)+1,-1
            sum = sum + MAT(j)*VEC_out(irw(j))
        enddo
        VEC_out(i) = (VEC(i)-sum)/MAT(jcol(i))*W
    endif
enddo

return
end subroutine back_solve
!================this is split line================================================!
function m_dot_product(A_in,B_in,N)
!***********************************************************************************
!**   向量的内积
!***********************************************************************************
implicit none
integer :: N
complex(8) :: A_in(N), B_in(N)
complex(8) :: m_dot_product
integer :: i
m_dot_product = 0.0
do i = 1 , N
    m_dot_product = m_dot_product + A_in(i)*B_in(i)
enddo
return
end function m_dot_product
!================this is split line================================================!
subroutine ID_CHANGE(ND,NNZ,irw,jcol,ICOL,JROW,ID_CH)
!***********************************************************************************
!**   矩阵A影射,由下三角CSC压缩存储影射到下三角CSR压缩存储
!***********************************************************************************
implicit none
!***********************************************************************************
!**   输入和输出
!***********************************************************************************
integer,intent(in) :: ND,NNZ
integer,intent(in) :: irw(NNZ),jcol(ND+1)   !CSR压缩存储索引矩阵pointB,pointE
integer,intent(out) :: icol(NNZ),jROW(ND)   !CSC压缩存储索引矩阵pointB,pointE
integer,intent(out) :: ID_CH(NNZ)
!***********************************************************************************
!**   程序中间变量
!***********************************************************************************
integer :: i,j,k,Now_col_nnz,N
icol(1) = 1
jrow(1) = 1
Now_col_nnz=1
ID_CH(1) = 1
N = 1
do i = 2 , ND
    now_col_nnz = 0
    do j = 1 , i-1
        do k = jcol(j)+1,jcol(j+1)-1
            if( irw(k) == i) then
                Now_col_nnz = Now_col_nnz + 1
                N = N + 1
                icol(N) = j
                ID_CH(N) = k
            endif
        enddo
    enddo
    Now_col_nnz = Now_col_nnz + 1
    N = N + 1
    icol(N) = i
    ID_CH(N) = jcol(i)
    jROW(i) = jROW(i-1) + now_col_nnz
enddo
return
end subroutine ID_CHANGE
```m


下面是测试程序

```
program SSOR_PCG_test
implicit none
integer, parameter :: N=4,NNZ=7,ITMAX = 100
complex(8) :: rhs(n),X(N),expected_sol(N)
real(8) :: reg_err(ITMAX)
integer :: jcol(N+1)
integer :: irw(NNZ)
complex(8) :: A(NNZ)
real(8) :: W,EPS
INTEGER :: IT
!example 1 N=10 NNZ = 18
! Fill all arrays containing matrix data.
!      DATA jcol /1,5,8,10,12,15,17,18,19/
!      DATA irw      /1, 3,     6,7,        &
                !         2,3,   5,            &
                !           3,         8,      &
                !              4,    7,        &
                !                5,6,7,        &
                !                  6, 8,      &
                !                    7,        &
                !                      8/
!      DATA A      /7.D0,      1.D0,           2.D0, 7.D0,         &   
!                        -4.D0,8.D0,     2.D0,                     &
!                              1.D0,                      5.D0,    &
!                                   7.D0,      9.D0,               &
!                                        5.D0, 1.D0, 5.D0,         &
!                                             -1.D0,      5.D0,    &
!                                                   11.D0,         &
!                                                         5.D0/
!     DATA rhs /17.,6.,15.,16.,13.,7.,32.,15./
!example 2 N=6,NNZ=10
!   DATA A/8.,-3.,10.,15.,-6.,-2.,11.,9.,-4.,12./
!   DATA irw /1,2,2,3,4,6,4,5,6,6/
!   DATA jcol /1,3,4,7,8,10,11/
!   DATA rhs/2.,17.,9.,26.,21.,46./
!example 3 N=4,NNZ=7
   DATA A/1.,5.,2.,6.,3.,7.,4./
   DATA irw /1,3,2,4,3,4,4/
   DATA jcol /1,3,5,7,8/
   DATA rhs/6.,8.,15.,17./

    X = (0.0D0,0.0D0)
    EPS = 1.0D-10                     
    W = 1
    call SSOR_PCG_CSR(N,NNZ,A,rhs,X,irw,jcol,EPS,ITMAX,W,IT,reg_err)
    write(*,*) X,IT,REG_ERR(IT)
    stop
    end program

```

大家有什么意见和建议（如：程序的改进，程序的收敛，收敛判断等），可以留言，也可以给我发邮件。
我用大型稀疏线性方程组测试了下，下面是测试结果：
===============================网格相关数据===============================
               X方向网格数NX：                     19
               Y方向网格数NY：                     16
               Z方向网格数NZ：                     26
               节点总数ND：                      9180
               总单元数：                        7904

     1024101
      565886      907253     1024102
init mesh is used time   0.156250000000000    
Change the ID use the time:    42.0468750000000    
第1次迭代后的残差为：    0.4142391878D+30
第2次迭代后的残差为：    0.2002487884D-01
第3次迭代后的残差为：    0.2157131128D-03
第4次迭代后的残差为：    0.1817152425D-03
第5次迭代后的残差为：    0.2412774049D-04
第6次迭代后的残差为：    0.1665214474D-03
第7次迭代后的残差为：    0.8162543084D-05
The whole iterative use the time:   0.421875000000000    
the program used time is    42.9218750000000    
网格数是9180
矩阵的阶数为：9180*3
非零数是：1024102
可以看出，绝大部分计算时间花费在ID映射转换时，若我们需要进行反演，我们可以把这个ID相关的存储起来，这样就不必花费这个时间来计算。若是需要进行多个模型，但是网格一样的情况，我们可以把ID用文件存储起来，批处理进行计算，花费时间相当小。
针对不同的稀疏矩阵，可以进行不同的优化，完全带状稀疏矩阵最好处理，对于分块带状矩阵，处理稍微麻烦点点，而且不同的处理效率不一样。下面的结果是我做的一点优化后的重排。重排节约时间一个数量级，没测试更大的矩阵，但是我真的更大效果更明显。
===============================网格相关数据===============================
               X方向网格数NX：                     19
               Y方向网格数NY：                     16
               Z方向网格数NZ：                     26
               节点总数ND：                      9180
               总单元数：                        7904
     1024101
      565886      907253     1024102
init mesh is used time   0.125000000000000
Change the ID use the time:    3.93750000000000
第1次迭代后的残差为：    0.1975780101D+24
第2次迭代后的残差为：    0.2836308853D-05
The whole iterative use the time:   0.156250000000000
the program used time is    4.51562500000000
