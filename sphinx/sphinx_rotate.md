# sphinx –rotate机制详解


今日，发现sphinx出现 sphinx.new.sp*诸多文件。出现这样的情况是因为 searchd没有加载新索引文件。遂Google之，到sphinx官网论坛后得知rotate的机制后方解决！  

sphinx的searchd在启动时会创建一个 .spl 锁文件，并在关闭时会删除它。在indexer创建索引时如果发现有 .spl文件，则不会创建新索引，因为这时已经标志sphinx正在运行中，除非使用 –rotate。  

roate运行机制  

->indexer完成索引  
->发送SIGHUP 给searchd（同时在终端输出索引已经完成）  
->searchd接到中断信号->等待所有子进程退出  
->重命名 当前索引为旧索引为 .old  
->重命名 .new 索引文件作为当前索引  
->尝试加载当前索引文件->如果加载失败，searchd会把.old文件回滚为当前文件，并把刚建立的新索引重命名为 .new
->加载成的话：完成无缝衔接  

综上：解决问题的办法是：  

关闭searchd :killall -9 searchd  
重启 searchd :searchd -c ../sphinx.conf  