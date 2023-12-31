# matlab-learn

### 矩阵

横向展示
a = [1, 2, 3, 4]

纵向展示(可以给分号想象为换行)
b = [1; 2; 3; 4]

二维矩阵获取索引
A(1,2) 第一行第二列
A(8) 一列一列查
A([1,3]) 生成新的一维数组 第一个数是A(1) 第二个数是A(3)
A([1,3;1,3]) 生成一个新的二维数组
A([1,3],[1,3]) 生成一个新的二维数组 第一个[1,3]表示为 第一个row和第三个row 第二个为column 取交集
A(3,:)整个第三行

拼接矩阵
F = [a,b] 横向合并
F = [a;b] 纵向合并

### 冒号运算符

[1:100] 快速生成一个1-100的数组
[1::2:100] 快速生成一个等差为2的1-100数组

### 矩阵之间运算

A*B = A第一行 * B第一列 ， A第一行 * B第二列 以此往复
A.*B =  A对应数字和B对应数字相乘的结果作为新数组的值
A./B 同理

特殊矩阵
linspace()
eye(n) 对角线为1 其余是0
zeros(n1,n2) n1 x n2 数值都是0
ones(n1,n2)
diag([2,3,4]) 对角线是2,3,4 其余是0
linspace(x1,x2) 返回包含 x1 和 x2 之间的 100 个等间距点的行向量
rand(n) 生成n x n的0-1随机矩阵
linspace(x1,x2,n) 生成 n 个点。这些点的间距为 (x2-x1)/(n-1)

### 矩阵方法

max(A) 返回每一列最大值
max(max(A)) 返回矩阵最大值
min/sum/mean(平均值) 同理

sort(A) 每一列进行排序 小的在第一列
sortrows(A) 依然是每一列进行排序 但是切换是整行进行切换
size(A) 查看几行几列
length(A) 返回最大维度值
