# Report of Legos
* **Summary of major innovations**
---
>**背景**

    随着硬件技术的发展，传统的将所有硬件集成到单个服务器上进行管理的方式存在许多缺陷。
    例如硬件资源利用率低、扩展性差、异构性差以及故障处理等问题。

将硬件资源进行分解后独立管理，并将各个硬件模块通过网络进行连接可以很好地缓解上述问题。

虽然硬件资源分解具有诸多优点，但目前仍没有相应的操作系统和软件系统对其进行很好地管理。本文针对硬件分解提出一种新的操作系统设计模型-**splitkernel**，该模型将传统的操作系统功能分解成彼此松耦合的**moniter**,这些**moniter**分别运行在不同的硬件模块上并对其进行管理。

针对这种新的操作系统设计模式，本文构建了**LegoOS**，从用户的角度来看**LegoOS**就是一组分布式服务器，在**LegoOS**内部，单个应用程序可以横跨多个处理器、内存和存储硬件组件。

本文利用普通服务器模拟了各个硬件组件，并在x86-64指令集上实现了**LegoOS**。实验结果表明，**LegoOS**在极大地提升硬件资源利用率并降低故障率的同时，性能与传统的单一服务器相同。

* **Problems this paper mentioned**
    
    * 传统的单一架构服务器存在的问题
        1.
            很难实现机器的高内存利用率和CPU资源利用率
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
