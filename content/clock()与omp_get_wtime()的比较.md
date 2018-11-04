+++
title = "clock()与omp_get_wtime()的比较"
date = 2018-11-04
category = "Concurrency"
+++

使用OpenMP开发C语言并行程序时，有时需要计算程序运行时间，这时有两个选择，使用头文件 `<time.h>` 中提供的`clock()`函数和使用omp_get_wtime()，这两者都可以测量程序运行时间，但在并行程序的背景下，这两者有差别。
在计算一个程序的运行时间的时候，如果只是简单的线性执行的程序，那么使用clock() 就可以计算出程序的执行时间，但是其实这个时间是CPU的时间。如果你用clock()计算并行程序执行时间，发现它会把所有CPU的执行都叠加起来。而使用omp_get_wtime()得到的时间是程序绝对运行时间。
下面这个程序可以方便的看出这两者的差别。

```
#include <omp.h>
#include <stdio.h>
#include <time.h>
#define NUM_THREADS 2
static long num_steps = 233333333;
double step;
// using openmp to calculate the \pi
int main() {
    int i;
    clock_t start,stop;
    omp_set_num_threads(NUM_THREADS);
    printf("NUM_THREADS is %d.\n",NUM_THREADS);
    double x, pi, sum = 0.0;
    step = 1.0 / (double) num_steps;
    start=clock();
    double start_wtime=omp_get_wtime();
#pragma omp parallel for reduction(+:sum) private(x)
    for (i = 1; i <= num_steps; i++) {
        x = (i - 0.5) * step;
        sum += 4.0 / (1.0 + x * x);
    }
    stop=clock();
    pi = step * sum;
    clock_t stop1=clock();
    double end_wtime1=omp_get_wtime();
    printf("%lf\n", pi);
    double duration=(double)(stop-start)/CLOCKS_PER_SEC;
    double end_wtime=omp_get_wtime();
    double duration1=(double)(stop1-start)/CLOCKS_PER_SEC;

    printf("Duration tested by clock(): %lf\n",duration);
    printf("Duration tested by omp_get_wtime(): %lf\n",end_wtime-start_wtime);
    printf("Duration 2 tested by clock(): %lf\n",duration1);
    printf("Duration 2 tested by omp_get_wtime(): %lf\n",end_wtime1-start_wtime);
    return 0;
}
```
当设置NUM_THREAD为2时，运行结果如下：

可以看出前者运行时间大概是后者的2倍。
当设置NUM_THREAD为2时，运行结果如下：