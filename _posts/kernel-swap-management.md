[Swap Management](https://www.kernel.org/doc/gorman/html/understand/understand014.html)

当私有或匿名页超过内存空闲空间时，由于没有磁盘备份，不能直接丢弃，而需要被替换到磁盘的特殊空间，称为*swap area*。

swap空间主要作用在于：1）扩大进程可用物理内存；
2）