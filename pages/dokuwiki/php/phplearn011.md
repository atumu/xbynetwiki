title: phplearn011 

#  PHP学习之Math处理 
##  随机数 
0、rand()，速度慢、范围小，不再推荐使用。

1、mt_rand — 生成更好的随机数，用来替换古老的rand()函数。
int mt_rand ( void )
int mt_rand ( int $min , int $max )
如果没有提供可选参数 min 和 max，mt_rand() 返回 0 到 mt_getrandmax() 之间的伪随机数。例如想要 5 到 15（包括 5 和 15）之间的随机数，用 mt_rand(5, 15)。

2、mt_srand — 播下一个更好的随机数发生器种子
void mt_srand ([ int $seed ] )
自 PHP 4.2.0 起，不再需要用 srand() 或 mt_srand() 给随机数发生器播种 ，因为现在是由系统自动完成的。