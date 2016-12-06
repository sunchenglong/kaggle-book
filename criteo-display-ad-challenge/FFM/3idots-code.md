# 3idiots实现代码详解

代码地址：  https://github.com/guestwalk/kaggle-2014-criteo

## [README](src/README)

小数据集上使用本工程的方法:
1. 建立两个符号链接(train.tiny.csv与test.tiny.csv分别是训练和测试的小样本):
     ```bash
     $ ln -s train.tiny.csv tr.csv
     $ ln -s test.tiny.csv te.csv
     ```

2. 生成预测结果文件"submission.csv":
    ```bash
     $ run.py
    ```
3. 为需要提交的结果文件生成md5,submission.csv:
    ```
     $ md5sum submission.csv
     19a913d1577c3d419f62e38c34305341  

     submission.csv
    ```

## [Makefile](src/Makefile)
```c
all: gbdt ffm-train ffm-predict

gbdt:
	make -C solvers/gbdt
	ln -sf solvers/gbdt/gbdt

ffm-train:
	make -C solvers/libffm-1.13
	ln -sf solvers/libffm-1.13/ffm-train

ffm-predict:
	make -C solvers/libffm-1.13
	ln -sf solvers/libffm-1.13/ffm-predict

clean:
	rm -f gbdt ffm fc.trva.t10.txt submission.csv *.sp* te.csv tr.csv
	make -C solvers/gbdt clean
	make -C solvers/ffm clean
```
从makefile中可以看出来，由c语言主要实现了三个模块，gbdt,ffm-train,ffm-predict

## [run.py](src/run.py)

```python
#!/usr/bin/env python3

import subprocess, sys, os, time

NR_THREAD = 1

start = time.time()

cmd = './utils/count.py tr.csv > fc.trva.t10.txt'
subprocess.call(cmd, shell=True)

cmd = 'converters/parallelizer-a.py -s {nr_thread} converters/pre-a.py tr.csv tr.gbdt.dense tr.gbdt.sparse'.format(nr_thread=NR_THREAD)
subprocess.call(cmd, shell=True)

cmd = 'converters/parallelizer-a.py -s {nr_thread} converters/pre-a.py te.csv te.gbdt.dense te.gbdt.sparse'.format(nr_thread=NR_THREAD)
subprocess.call(cmd, shell=True)

cmd = './gbdt -t 30 -s {nr_thread} te.gbdt.dense te.gbdt.sparse tr.gbdt.dense tr.gbdt.sparse te.gbdt.out tr.gbdt.out'.format(nr_thread=NR_THREAD)
subprocess.call(cmd, shell=True)

cmd = 'rm -f te.gbdt.dense te.gbdt.sparse tr.gbdt.dense tr.gbdt.sparse'
subprocess.call(cmd, shell=True)

cmd = 'converters/parallelizer-b.py -s {nr_thread} converters/pre-b.py tr.csv tr.gbdt.out tr.ffm'.format(nr_thread=NR_THREAD)
subprocess.call(cmd, shell=True)

cmd = 'converters/parallelizer-b.py -s {nr_thread} converters/pre-b.py te.csv te.gbdt.out te.ffm'.format(nr_thread=NR_THREAD)
subprocess.call(cmd, shell=True)

cmd = 'rm -f te.gbdt.out tr.gbdt.out'
subprocess.call(cmd, shell=True)

cmd = './ffm-train -k 4 -t 18 -s {nr_thread} -p te.ffm tr.ffm model'.format(nr_thread=NR_THREAD)
subprocess.call(cmd, shell=True)

cmd = './ffm-predict te.ffm model te.out'.format(nr_thread=NR_THREAD)
subprocess.call(cmd, shell=True)

cmd = './utils/calibrate.py te.out te.out.cal'.format(nr_thread=NR_THREAD)
subprocess.call(cmd, shell=True)

cmd = './utils/make_submission.py te.out.cal submission.csv'.format(nr_thread=NR_THREAD)
subprocess.call(cmd, shell=True)

print('time used = {0:.0f}'.format(time.time()-start))
```
run.py实现的流程图如下图所示：
![img](../img/3idiot.png)


对照上述图片和代码可以发现，图中gbdt, ffm-predict, ffm-train分别对应于[gbdt](src/solvers/gbdt), [ffm](src/solvers/libffm-1.13)两个模块的实现，在Makefile中对这两个模块的三个函数进行了符号链接.

但是Pre-A与Pre-B的实现是[pre-a.py](src/converters/pre-a.py)与[pre-b.py](src/converters/pre-b.py),在run.py中使用[parallelizer-a.py](src/converters/parallelizer-a.py)与[parallelizer-b.py](src/converters/parallelizer-b.py)进行驱动，目的是数据预处理使用“多线程”(其实使用的subprocess模块进行的实现)，加速数据处理和特征提取。

## 并发数据预处理Pre-A与Pre-B的实现
[parallelizer-a.py](src/converters/parallelizer-a.py)
