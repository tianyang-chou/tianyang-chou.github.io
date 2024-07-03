## 如何从AOCC获取优化信息

> 首先需要明确的是，AOCC不像GCC或者LLVM一样，它是闭源编译器。

#### 查看向量化Pass的Optimization Remark

```bash
clang -O3 -Rpass=vector -Rpass-analysis=vector -Rpass-missed=vector -S xx.c #可将vector替换成别的Pass名字
# 输出为：
# xx.c:19:14: remark: Cannot SLP vectorize list: vectorization is impossible with available vectorization factors [-Rpass-missed=slp-vectorizer]
# xx.c:22:5: remark: Vectorized horizontal reduction with cost -26 and with tree size 21 [-Rpass=slp-vectorizer]
```

#### 查看每个Pass执行前后的LLVM IR

```bash
# 先打印所有Pass产生的IR
clang -O3 -mllvm -print-before-all -mllvm -print-module-scope -S xx.c &> output.txt
# 在输出文件中找到对应的Pass，即可摘出所需要的IR，比如在output.txt中摘出SLPVectorizer Pass执行前的IR，形成before.ll文件
opt -o asm.s -S -p slp-vectorizer -debug-only=slp-vectorizer before.ll
# 上述指令要执行成功，必须使用assertions enabled版本的opt，也就是build opt的时候打开assertion开关（AOCC不需要编译，所以似乎做不到，只有llvm能做到）。执行成功后会打印slp这个pass的执行过程
# 上述两条指令执行后，可以得到before.ll和asm.s两个文件，分别记录着SLP Pass执行前的LLVM IR以及执行后的汇编，可以通过对比这两个文件的内容来推测AOCC的优化思路
```

