https://www.huaweicloud.com/articles/63b7188d445f452f148dc953f61861e7.html

Linux内核内存管理的一项重要工作就是如何在频繁申请释放内存的情况下，避免碎片的产生。

Linux采用伙伴系统解决外部碎片的问题，采用slab解决内部碎片的问题.

1，伙伴系统（Buddy System）

       用一种有效方法监视内存。保证在内核只要申请一小块内存的情况下，不会从大块连续空闲内存中截取一段，从而保证大块内存的连续性和完整性。

      在物理页面管理上实现了基于区的伙伴系统（zone based buddy system）。对不同区的内存使用单独的伙伴系统(buddy system)管理,而且独立地监控空闲页。相应接口alloc_pages(gfp_mask, order)，_ _get_free_pages(gfp_mask, order)等。

     步骤：1，把空闲页框分组为11种块链表，分别包含2的0-10次方个连续页（一个页面64k），称为页框块。最大页框块：2^10页，即4MB。2，分配内存时，优先分配最符合待分配大小的页框块。

2，Slab分配器
