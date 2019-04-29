# Report of Legos
* **Summary of major innovations**
---
>**背景**

    随着硬件技术的发展，传统的将所有硬件集成到单个服务器上进行管理的方式存在许多缺陷。
    例如硬件资源利用率低、扩展性差、异构性差以及故障处理等问题。

传统的kernel在多核处理器硬件上不能很好地扩展, 当处理器核数增加时,上层应用将在kernel部分花费更多的时间,kernel成为导致scalability差的主要瓶颈. 因此,已经有相关研究提出新的kernel设计方式(例如Barrelfish,Corey和fos),以实现更好的扩展性. 相比其他的传统kernel而言,Linux 社区对Linux在scalability方面做了更多的改进, 因此,本文选取Linux作为传统kernel的代表,选取一些典型应用测试集对Linux在多核处理器下的scalability问题做了评估. 该测试集(论文又叫MOSBENCH)包含7个分属不同应用的可并行化的用户程序,分别是Exim(mail server),memcached(Object cache),Apache( Web server),PostgreSQL(Database),gmake(Parallel build),Psearchy(File indexer)和a multicore MapReduce library. 每个可并行化的应用程序对Linux kernel的特定component有更高的要求(本文主要指该component在多核处理器场景下的scalability表现). 

* **Problems this paper mentioned**
    
    * 传统的单一架构服务器存在的问题
        1.很难实现机器的高内存利用率和CPU资源利用率
        2. 在现有的数据中心添加新的硬件设备成本很高
        3. 当服务器中单个组件出现故障，整个机器就不可用了
        4. 系统的异构性很差
    
    硬件资源分解虽然能解决上述问题，但依旧存在诸多挑战。传统的操作系统，例如，Monolithic kernels，microkernels和exokernel无法通过重构后部署于硬件分解的场景中，因为依旧会出现网络开销过大、单一组件故障导致的系统不可用的问题。

* **The important related works/papers**
    * Memory Disaggregation and Remote memory
    1. Lim等人首先提出了硬件分解内存的概念，通过两种方式访问硬件分解后的内存：（1）网络（2）透明地内存指令。
    * Storage Disaggregation
    1. ReFlex将网络和存储紧密地整合在一起，最小化软件开销，实现了存储资源分解

* **Some intriguing aspects of the paper**
    * 将传统操作系统的功能分解成不同moniter，每个moniter管理一个硬件组件，并提供虚拟化、保护等功能。
    * moniter之间共享最小的状态，彼此之间松耦合。
    * 在每个硬件组件上都有一个硬件控制器运行相应的moniter，例如ASIC和FPGA。以此极大地增强了系统的异构性。
    * 系统的各个组件之间不维护缓存一致性，以减少网络开销，应用可以使用消息接口实现自己的一致性保障。
    * 通过全局资源管理器来控制组件故障对系统的影响。

 * **Test/compare/analyze**

     **LegoOS**在x86-64结构上实现，支持大多数通用的Linux系统调用API。通过限制普通服务器内部资源使用的方式模拟分解后的硬件资源组件，例如，在mComponets和sComponents组件的模拟过程中限制服务器的可用cores的数目为2.
     
     各个组件之间通过**LITE**连接
     * Micro-benchmark
    
     网络性能：对**LegoOS**各组件之间的网络性能以及应用与组件之间的网络性能进行了测试，并与Linux进行对比，性能优越。
    
     访存性能：在多线程模式下进行测试，并分别在拥有一个和两个mComponets的**LegoOS**中进行访存测试
     
     存储性能：由于三次网络开销的原因，LegoOS的write-and-fsync性能比Linux要差。
     
     * Macro-benchmark
        * TensorFlow和Phoenix
        
        实验设置：256MB 64-way的ExCache，pComponent、mComponent和sComponent各一个，其他的实验设置参数见论文
        
        1. ExCache大小对应用性能的影响
        
        相比将文件swap到本地SSD和本地ramdisk，LegoOS的性能都是最好的，主要是因为其有效的网络栈实现以及最少的数据拷贝。
        
        2. ExCache管理策略的影响
      
        得出piggy-back dirty cache line flush于FIFO和free list的设计策略实现最好的性能
        
        3. mComponent数目和副本数对性能的影响
        
        由于ExCache piggyback optimization的影响，当mComponent数增加时，性能反而下降。Replication有2%到23%的性能开销。
        4. 运行多个应用时的测试结果
        
        实验设置见论文，测试结果显示，在2P1M场景下，对mComponent的访问成为性能瓶颈。在1P2M场景下，性能大小受ExCache大小的影响。
        
 * **Solutions that can improve the research**    
    * 针对内存组件的负载均衡策略设计，如数据迁移等策略
    * 同理针对存储组件的负载均衡设计
    

* **If you write this paper, then how would you do?**

    我认为论文的架构和语言表述方面都做得很好，很多地方值得借鉴和学习。
    
* **the survey paper list in the same area of the paper**

   * ReFlex: Remote Flash at the Performance of Local Flash
   * Decibel: Isolation and Sharing in Disaggregated Rack-Scale Storage
