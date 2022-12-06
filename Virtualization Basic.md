# Virtualization Basic

# 基本概念

## 实现层次

现代系统的多任务并发其实就有虚拟化的思维，包括CPU与内存虚拟化，先看看根据实现层次的分类：

1.**功能级虚拟化**：为了充分运用硬件的每一个部分而提出的一种技术，如为了提高内存使用率(当然也有保护机制的功能)而实现的内存虚拟化，为了提高网卡各部件利用率而实现的SR-VIO等。

2.**语言级虚拟化**：指语言虚拟机，它们对底层的CPU进行虚拟化，如Java虚拟机，Python虚拟机，此时在其上运行的是专用于该虚拟机的指令，一般为字节码，再由虚拟机将其翻译为实际物理机的指令，此时一般采用解释执行的方式，更进一步会采用JIT在运行时采样编译为机器码执行。

3.**接口级虚拟化**：这是一种在同架构下执行其他系统二进制程序的技术，Windows下的程序无法直接在同架构的Linux下系统执行，因为它们的ABI不一致，这涉及可执行文件格式与依赖库，前者只需要编写相应的加载程序，后者面临的问题更加复杂，它涉及很多闭源的依赖库，为此需要完全重新实现相同功能的库，此类软件有如Linux下的wine与mac下的crossover。

4.**容器级虚拟化**：这是一种利用操作系统隔离机制实现虚拟化的技术，在此技术下对于容器内的应用它仿佛运行在一台普通的虚拟机中，但在外部看来它只是一个普通进程，只是由命名空间等实现了一个jail，它和宿主系统使用了同样的内核，因此会有一些ABI兼容性限制，此处就是指LXC及其上的Docker。

5.**系统级虚拟化**：这是一种更完整的虚拟化技术，该技术下需要虚拟完整的硬件，其实对于软件来说，它只关注CPU/内存/外设(中断与IO)这三点，因此虚拟出这三点时就能实现完整的系统虚拟化。

![https://blog.betamao.me/posts/2021/virtualization-principle-summary/pic/1639462279992-a0bd037e-1bb7-4f50-bcdf-3ea25914de83.png](https://blog.betamao.me/posts/2021/virtualization-principle-summary/pic/1639462279992-a0bd037e-1bb7-4f50-bcdf-3ea25914de83.png)

> 注：1. 也有把系统级叫做硬件级，而把容器级叫做系统级 2. 还有种常见的分类，根据功能来分，如网络虚拟化，存储虚拟化等，感觉现在没什么人挖它...

## 实现方式

虚拟化技术有三个指标：同质，高效，资源受控，现在的虚拟化也围绕这三个目标展开，在硬件级虚拟化时有如下三类：

**1.全虚拟化(Full Virtualization)**：全虚拟化技术能最好的实现一致性，它会**完全模拟所有过程**，此时对客户机来说它不需要做任何改变，因此适用性很强，但是原样的实现硬件功能会带来很多不必要的性能开销，因此该方式在某些地方性能较差。

**2.类虚拟化(Para-Virutalization)**：类虚拟化能更好实现等效性，它需要对宿主机做一定的修改，或在宿主机里安装一些驱动，令其通过一些额外的方式与VMM通信从而实现性能提升，该技术会增加实现难度但能显著提升性能，然而很多虚拟机逃逸漏洞也出在这一块。

**3.半虚拟化(Partial-Virtualization)**：它只实现了部分虚拟化，是个不完全体，没见过。

> 注：由于类虚拟化不满足三代虚拟机中提出的一致性，因此翻译叫"类"虚拟化，大多场景也会被翻译为半虚拟化，需意识他们可能是同样的东西。

而在实现方式上，有如下三种：

**1.软件完全虚拟化**：完全用软件实现虚拟化功能，不受硬件制约，但实现复杂效率偏低

**2.硬件辅助虚拟化**：Intel与AMD都提出了各自的CPU硬件虚拟化功能VT与AMD-V，硬件辅助的好处就是效率高实现简单

**3.类虚拟化**：完全虚拟化时VMM只能知道Guest的部分状态，存在[语义鸿沟](https://en.wikipedia.org/wiki/Semantic_gap)，而且不可避免的存在代码重复降低性能，类虚拟化用于弥补这两个缺陷

------

## Hypervisor

虚拟机系统里，会出现三个角色：**宿主机Host，虚拟机监视器VMM(Virtual Machaine Monitor，也叫Hypervisor)，客户机Guest(也就是通常意义上的VM)。**

Hypervisor是一种运行在**基础物理服务器和操作系统之间的中间软件层，可允许多个操作系统和应用共享硬件**。它是一种在虚拟环境中的**“元”操作系统**。他们可以访问服务器上包括磁盘和内存在内的所有物理设备。Hypervisor不但协调着这些硬件资源的访问，也同时在各个虚拟机之间施加防护。**当服务器启动并执行Hypervisor时，它会加载所有虚拟机客户端的操作系统同时会分配给每一台虚拟机适量的内存，CPU，网络和磁盘。**

A **hypervisor** (also known as a **virtual machine monitor**, **VMM**, or **virtualizer**) is a type of computer software, firmware or  hardware that creates and runs virtual machines. A computer on which a hypervisor runs one or more virtual machines is called a *host machine*, and each virtual machine is called a *guest machine*. The hypervisor presents the guest operating systems with a [virtual operating platform](https://en.wikipedia.org/wiki/Platform_virtualization) and manages the execution of the guest operating systems. Unlike an [emulator](https://en.wikipedia.org/wiki/Emulator), the guest executes most instructions on the native hardware.[[1\]](https://en.wikipedia.org/wiki/Hypervisor#cite_note-goldberg1973-1) Multiple instances of a variety of operating systems may share the virtualized hardware resources: for example, [Linux](https://en.wikipedia.org/wiki/Linux), [Windows](https://en.wikipedia.org/wiki/Microsoft_Windows), and [macOS](https://en.wikipedia.org/wiki/MacOS) instances can all run on a single physical [x86](https://en.wikipedia.org/wiki/X86) machine.

VMM**向下管理各种资源，向上提供各种服务**，即它们要实现两件事，物理资源的管理与虚拟资源的管理，从这种实现架构上可分为三种架构

1. **Hypervisor型**：**Hypervisor直接跑在物理机（或原生硬件）上，直接控制硬件资源**，这两件事都由VMM完成，效率高更安全但是实现复杂，如Vmware ESX server，KVM+Kernel
2. **Hosted型**：**物理资源管理由宿主机系统完成，虚拟化与虚拟资源管理由VMM实现**，优点是复用现有系统实现简单，缺点就是暴露面太多，且效率相对较低，如Vmware Workstation
3. **Mixed型**：VMM实现物理资源管理，但再创建一个特殊虚拟机去复用已有系统的某些功能，优点就是简单与安全，缺点就是效率降低了，如XEN：

![image-20221206203505971](https://cdn.jsdelivr.net/gh/AlexsanderShaw/BlogImages@main/img/202212062035033.png)

传统上也把前面两种架构叫做一型和二型：

**Type-1, native or bare-metal hypervisors**

These hypervisors run directly on the host's hardware to control the hardware and to manage guest operating systems. For this reason, they are sometimes called [bare-metal](https://en.wikipedia.org/wiki/Bare_machine) hypervisors.

**Type-2 or hosted hypervisors**

These hypervisors run on a conventional operating system (OS) just as other computer programs do. A virtual machine monitor runs as a [process](https://en.wikipedia.org/wiki/Computer_process) on the host. Type-2 hypervisors abstract guest operating systems from the host operating system.

![image-20221206203519471](https://cdn.jsdelivr.net/gh/AlexsanderShaw/BlogImages@main/img/202212062035582.png)

实现时，VMM可大致分为三部分：

**dispatcher：**VM退出时根据退出原因分发到不同模块去处理

**allocator：**分配各种所需资源，很多资源都需要为其建立一份虚拟拷贝，VM读写时都是操作这份拷贝(shadow structures)

**interpreter：**使用等效的代码解释导致VM退出的特殊指令/事件

------

## 虚拟化常规分类

（本质上跟上面的分类是一样的，这里只是再重述一次比较常规的分类方法。）

**1. 完全虚拟化**

最流行的虚拟化方法，**使用Hypervisor这种中间层软件，在虚拟服务器和底层硬件之间建立一个抽象层**。

**Hypervisor可以捕获CPU指令，为指令访问硬件控制器和外设充当中介**。因而，完全虚拟化技术几乎能让任何一款操作系统不用改动就能安装到虚拟服务器上，而它们不知道自己运行在虚拟化环境下。主要缺点是，性能方面不如裸机，因为Hypervisor需要占用一些资源，给处理器带来开销。

在完全虚拟化的环境下，Hypervisor运行在裸硬件上，充当主机操作系统，而由Hypervisor管理的虚拟服务器运行客户端操作系统(Guest OS)。

![https://ask.qcloudimg.com/http-save/4069933/9i8ps33b3h.jpeg?imageView2/2/w/1620](https://ask.qcloudimg.com/http-save/4069933/9i8ps33b3h.jpeg?imageView2/2/w/1620)

**2. 准虚拟化**

完全虚拟化是处理器密集型技术，因为它要求Hypervisor管理各个虚拟服务器，并让它们彼此独立。减轻这种负担的一种方法就是，**改动客户操作系统，让它以为自己运行在虚拟环境下，能够与Hypervisor协同工作**，这种方法就叫准虚拟化。

准虚拟化技术的优点是性能高。经过准虚拟化处理的服务器可与Hypervisor协同工作，其响应能力几乎不亚于未经过虚拟化处理的服务器。它的客户操作系统(Guest OS)集成了虚拟化方面的代码。该方法无需重新编译或引起陷阱，因为操作系统自身能够与虚拟进程进行很好的协作。

![https://ask.qcloudimg.com/http-save/4069933/xm0thu5fff.jpeg?imageView2/2/w/1620](https://cdn.jsdelivr.net/gh/AlexsanderShaw/BlogImages@main/img/202212062034623.jpeg)

**3. 操作系统层虚拟化**

实现虚拟化还有一个方法，那就是在操作系统层面增添虚拟服务器功能。就操作系统层的虚拟化而言，没有独立的Hypervisor层。相反主机操作系统本身就负责在多个虚拟服务器之间分配硬件资源，并且让这些服务器彼此独立。一个明显的区别是，**如果使用操作系统层虚拟化，所有虚拟服务器必须运行同一操作系统。**

虽然操作系统层虚拟化的灵活性比较差，但本机速度性能比较高。此外，由于架构在所有虚拟服务器上使用单一、标准的操作系统，管理起来比异构环境要容易。

**4. 桌面虚拟**

服务器虚拟化主要针对服务器而言，而虚拟化最接近用户的还是要算的上桌面虚拟化了，桌面虚拟化主要功能是将分散的桌面环境集中保存并管理起来，包括桌面环境的集中下发，集中更新，集中管理。桌面虚拟化使得桌面管理变得简单，不用每台终端单独进行维护，每台终端进行更新。终端数据可以集中存储在中心机房里，安全性相对传统桌面应用要高很多。桌面虚拟化可以使得一个人拥有多个桌面环境，也可以把一个桌面环境供多人使用，节省了license。另外，桌面虚拟化依托于服务器虚拟化。没有服务器虚拟化，这个桌面虚拟化的优势将完全没有了。不仅如此，还浪费了许多管理资本。

**5. 硬件虚拟化**

英特尔虚拟化技术(IVT，Intel Virtualization Technology)是由英特尔开发的一种虚拟化技术，利用IVT可以对在系统上的客操作系统，通过虚拟机查看器(VMM，Virtual Machine Monitor)来虚拟一套硬件设备，以供客操作系统使用。这些技术以往在VMware与Virtual PC上都通过软件实现，而通过IVT的硬件支持可以加速此类软件的进行。

AMD虚拟化(AMD Virtualization)，缩写为“AMD-V”，是AMD为64位的x86架构提供的虚拟化扩展的名称，但有时仍然会用“Pacifica”(AMD开发这项扩展时的内部项目代码)来指代它。

## 虚拟化应用分类

### 服务器虚拟化

服务器虚拟化是将服务器物理资源抽象成逻辑资源，让一台服务器变成几台甚至上百台相互隔离的虚拟服务器，我们不再受限于物理上的界限，而是让CPU、内存、磁盘、I/O等硬件变成可以动态管理的“资源池”，从而提高资源的利用率，简化系统管理，实现服务器整合，让IT对业务的变化更具适应力。

简单理解就是在一台或多台配置很高的服务器上安装众多虚机，让这些虚机共享宿主机的CPU，内存，存储以及I/O等资源，并作为服务器对外提供服务。在服务器虚拟化方面Vmware 一直处于行业领先地位。它是属于**IaaS，即基础设施即服务**。

### 存储虚拟化

存储虚拟化就是**对存储硬件资源进行抽象化表现**。通过将一个（或多个）目标服务或功能与其它附加的功能集成，统一提供有用的全面功能服务。

存储虚拟化，可以将异构的存储资源组成一个巨大的“存储池”，对于用户来说，不会看到具体的磁盘、磁带，也不必关心自己的数据经过哪一条路径通往哪一个具体的存储设备，只需要使用存储池中的资源即可。从管理的角度来看，虚拟存储池可以采取集中化的管理，可以由管理员根据具体的需求把存储资源动态地分配给各个应用。

### 平台虚拟化

平台虚拟化是**集成各种开发资源虚拟出来的一个面向开发人员的统一接口**，软件开发人员可以方便的在这个虚拟平台中开发各种应用并嵌入到云计算系统中，使其成为新的服务供用户使用，平台虚拟化是属于**PaaS（Platform as a Service）**层，该技术相对于IaaS和SaaS 的发展是滞后的，但该技术会是一种趋势，**典型的docker 就属于平台虚拟化**。

### 桌面虚拟化

在一台或多台配置较高的服务器上安装众多虚机，让这些**虚机共享宿主机的CPU，内存，存储以及I/O等资源，并作为桌面服务对外提供给用户使用**。从字面上看与服务器虚拟化很相似，但不同的是客户机相对服务机来说配置要求就没有那么高，而最明显的区别是客户机始终是需要将桌面呈现给客户操作，即用户必须看到并登陆桌面操作使用；而服务机一般则不需要，一般都是管理员或技术人员安装或升级服务的时候才会登陆到服务器上。**桌面虚拟化拥有服务器虚拟化的所有优点**。

尽管桌面虚拟化与服务器虚拟化看上去比较相似，尽管VMware在服务器虚拟化方面一直处于行业领先，但是在桌面虚拟化以及后面说到的应用虚拟化方面 Citrix 思杰一直是处于领先地位。同样的，如果对照云计算架构，它是属于IaaS，PaaS还是SaaS，感觉都不太合适，有人称它为**DaaS，也就是桌面即服务**。

### 应用虚拟化

将应用程序与操作系统解耦合，**为应用程序提供了一个虚拟的运行环**境。在这个环境中，不仅包括应用程序的可执行文件，还包括它所需要的运行时环境。从本质上说，**应用虚拟化是把应用对低层的系统和硬件的依赖抽象出来，可以解决版本不兼容的问题。**

简单理解就是在一台服务器上安装多台虚机，再在其中一台虚机上安装应用程序（如office软件），通过一定的技术将该应用程序及运行环境一起打包并分配给桌面虚拟机对外提供服务。和桌面虚拟化技术一样，应用程序不是存在本地电脑上，也是在后台的数据中心里，只是桌面虚拟化推送的是整个桌面，而应用程序虚拟化推送的是某个应用程序，用户只能看到应用程序。这里强调的应用程序虚拟化，应该处于**SaaS层，也就是软件即服务**。

## 虚拟化技术分类

### CPU虚拟化技术

CPU虚拟化是VMM中最重要的部分，因为访问内存或者I/O的指令本身就是敏感指令（客户机的特权指令），所以内存虚拟化和I/O虚拟化都依赖于CPU虚拟化。

在x86体系架构中，处理器有4个运行级别，分别为Ring0、Ring1、Ring2和Ring3，其中Ring0级别最高，操作系统内核就运行在Ring0层，因为它需要直接控制和修改CPU的状态，而应用程序一般运行在Ring3 级别。

我们可以通过软件虚拟化技术来实现VMM，但这会增加系统的复杂度以及带外额外的性能开销，目前，两大CPU厂商Intel 和 AMD 都已经在其CPU硬件上加入了专门针对虚拟化技术的支持，使其性能几乎能达到物理机水平。常见的KVM 搭建虚拟机就需要硬件基于支持。

### 内存虚拟化技术

从操作系统的角度，对物理内存有两条基本认识：

1）内存都是从物理地址0开始的；

2）内存地址都是连续的，或者说至少在一些大的粒度上连续。

但是在虚拟环境下，由于VMM 与客户操作系统对物理内存的认识上存在冲突，造成了物理内存的真正拥有者VMM必须对客户机操作系统所访问的内存地址进行虚拟化，使模拟出来的内存符合客户机操作系统的两条基本认识，这个模式过程就是内存虚拟化；为了达成上述两条基本认识，内存虚拟化引入了一层新的地址空间—**客户机物理地址，这个地址不是真正的物理地址，而是又VMM管理的“伪”物理地址。**

当引入客户机地址后，内存虚拟化的主要任务就是处理以下两个方面的问题：

1）实现地址空间的虚拟化，维护宿主机地址与客户机物理地址之间的映射关系；这个问题可以通过两次地址转换来支持地址空间的虚拟化，即客户机虚拟地址（GVA）->客户机物理地址（GPA） -> 宿主机物理地址（HPA）。在这个上 Intel VT-x 提供了一种EPT内存虚拟化技术以及 AMD提供了一种NPT内存虚拟化技术都支持在CPU硬件上自动完后。

2）截获宿主机对客户机物理地址的访问，并根据所记录的映射关系，将其转换成宿主机的物理地址。这个问题从实现来说比较复杂，尤其是还要考虑性能问题。

### I/O 虚拟化技术

性能和通用性是I/O虚机技术的两项重要指标，其不同于CPU和内存虚拟化多由硬件支持实现，如intel VT-x、VT-d等IA架构扩展，还有EPT页表等。客户机可以使用的设备大致可分为三类：

设备模拟： 完全由纯软件模拟的设备，如qemu来模拟设备。该方法最大的好处就是通信性强，不需要专用的驱动，但是缺点也很明显，那就是性能比较底下；一般不适合产品发布与应用。

设备半虚拟化： 实现半虚拟化设备，如qemu/kvm 中采用Virtio半虚拟化设备来解决纯软件模拟设备效率低下的问题，但是客户端机需要安装相应的驱动。

PCI 设备直接分配 （PCI device assignment）： 也称为设备透传，如，Intel VT-d 技术引入DMA重映射硬件，以提供设备重映射和设备直接分配的功能；但使用这种方式有一个缺点就是一个物理设备资源只能分配给一个虚机使用，为了实现多个虚机公用同一物理设备资源并设备直接分配，PCI-SIG组织发布了一个I/O虚拟化技术标准——SR-IOV。

SR-IOV 是PCI-SIG 组织发布的一个新规范，目的是消除VMM对虚拟化I/O操作的干预，以提高数据传输的性能。这个规范定义了一个标准机制，可以实现多个设备的共享，它继承了Passthrough I/O 技术，绕过VMM 直接发送和接收I/O数据，同时利用IOMMU 减少内存保护和内存地址转换的开销。

## 虚拟化实现分类

虚拟化技术的大致分类情况，即分为全虚拟化、半虚拟化和硬件辅助虚拟化3大类。而虚拟化技术最主要的虚拟主体就是硬件CPU、内存和IO。或着说细分下来，我们又可以分为：

- CPU的全虚拟化技术、半虚拟化技术和硬件辅助虚拟化技术
- 内存的全虚拟化技术、半虚拟化技术和硬件辅助虚拟化技术
- IO设备的全虚拟化技术、半虚拟化技术和硬件辅助虚拟化技术

### 完全虚拟化

全虚拟化技术：最初所使用的虚拟化技术就是纯软件的全虚拟化（Full Virtualization)技术，它在虚拟机（VM）和硬件之间加了**Hypervisor（VMM**）。在客户机操作系统看来，完全虚拟化的虚拟平台和现实平台是一样的，客户机操作系统察觉不到是运行在一个虚拟平台上，这样的虚拟平台可以运行现有的操作系统，无须对操作系统进行任何修改，因此这种方式被称为完全虚拟化。

进一步说，客户机的行为是通过执行反映出来的，因此VMM需要能够正确处理所有可能的指令。在实现方式上，以x86架构为例，完全虚拟化经历了两个阶段：软件辅助的完全虚拟化和硬件辅助的完全虚拟化。

cpu的软件全虚拟化其主要用到模拟仿真技术和二进制翻译技术。

### 半/准虚拟化

Para Virtualization，也叫做准虚拟化技术。在全虚拟化的基础上，把客户操作系统进行了修改，即不需要Hypervisor耗费一定的资源进行翻译操作，因此Hypervisor的工作负担变得非常的小，因此整体的性能也有很大的提高；比较具有代表性的产品是Xen，其早期版本主要采用Hypercall技术实现敏感指令或特权指令的处理。

半虚拟化的思想就是，让客户操作系统知道自己是在虚拟机上跑的，工作在非ring0状态，那么它原先在物理机上执行的一些特权指令，就会修改成其他方式，这种方式是可以和VMM约定好的，这就相当于，通过修改代码把操作系统移植到一种新的架构上来，就像是定制化。所以XEN这种半虚拟化技术，客户机操作系统都是有一个专门的定制内核版本，和x86、mips、arm这些内核版本。这样以来，就不会有捕获异常、翻译、模拟的过程了，性能损耗非常低。这就是XEN这种半虚拟化架构的优势。这也是为什么早期版本XEN半虚拟化只支持虚拟化Linux，无法虚拟化windows原因，微软不修改代码无法实现半虚拟化。

### 硬件辅助全虚拟化技术

在硬件还未提供很好的支持之前，基于软件的虚拟化技术已经给出了两种可行的解决方案：全虚拟化和半虚拟化。这里说的硬件主要是指CPU和内存，随着Intel 的VT-x和AMD 的AMD-V硬件虚拟化技术的提升，已经能将CPU和内存的性能提高到真机水平；但是设备（如磁盘、网卡）是有数目限制的，尽管VT-d技术已经可以做到一部分硬件隔离，但是大部分情况还是需要软件进行模拟，在全虚拟化的情况下是通过类似qemu进行模拟，而半虚拟化则可以通过虚拟机之前共享内存的方式利用特权虚拟机的设备驱动直接访问硬件，从而达到更高效的性能水平。在全虚拟化情况下，为了提升I/O性能，如qemu/kvm 会通过在虚拟化中安装VirtIO 前端驱动（相当于修改客户机操作系统）来与qemu中VirtIO后端驱动进行协作，从而达到更高效的性能水平。可以预测，该技术将是未来虚拟化技术的核心。

前面半虚拟化驱动中说到，早期版本Xen 半虚拟化技术由于需要修改操作系统，因此不能虚拟化不经修改的windows 操作系统，但是有从事Xen虚拟化工作的朋友一定知道在Xen上是完全可以跑Windows的虚机的，这其中的原因就是在2005年末，Intel开始努力完善并加强自己的硬件虚拟化产品，发布了具有Intel VT虚拟化技术的一系列处理器产品，AMD差不多也在2006年的时候开始发力，大力支持通过硬件虚拟化技术来优化产品的性能。因此后面发布的Xen 3.0以上版本就开始支持全虚拟化，即不需要修改操作系统就可以运行Windows 客户机。

## 虚拟化产品分类

### VMware

VMware 提供了很多的虚拟化产品，从服务器到桌面都有很多应用。主要有面向企业级应用的 ESX Server，面向服务端的入门级产品 VMware Server，面向桌面的主打产品 VMware Workstation，面向苹果系统的桌面产品 VMware Fusion，还有提供整套虚拟应用产品的 VMware vSphere，细分的话还有 VMware vStorage（虚拟存储），VMware vNet（虚拟网络）等。

### Hyper-V

Hyper-V 是微软的一款虚拟化产品，是微软首个采用类似VMware和开源Xen一样的基于Hypervisor的技术。其采用**微内核**结构，兼顾了安全性和性能的要求。从架构上来说，Hyper-V只有“**硬件—Hyper-V—虚拟机**”三层，本身非常小巧，代码简单，其**不包含任何第三方驱动程序**，所以安全可靠，执行效率高，充分利用硬件资源，使虚拟机系统性能更接近真实系统性能。

### Xen

Xen 是一款开源虚拟机软件，Xen 从一开始是作为一个半虚拟化的解决方案出现的。因此，为了支持多个虚拟机，内核必须针对Xen 做出特殊的修改才可以运行，但是到了2005 年，intel开始为Xen 添加硬件虚拟化的支持，在多方的努力下，从Xen 3.0开始正式支持Intelde VT 技术和IA64 架构，从而使得Xen 虚拟机可以运行完全没有修改过的操作系统。

其架构主要包含3个部分：

1. Xen Hypervisor：直接运行硬件之上，是Xen 客户操作系统与硬件资源之间的访问接口；
2. Domian 0：运行在Xen 管理程序之上，是具有直接访问硬件和管理其他客户操作系统特权的客户操作系统；它作为一个特殊的虚拟机存在。是经过修改的Linux内核，是运行在Xen Hypervisor 之上独一无二的虚拟机，拥有访问物理I/O资源的特权，并且可以与其它运行在Xen Hypervisor 之上的虚拟机进行交互，所有的Xen 虚拟环境都必须先运行Domain 0，然后才能运行其它的虚拟客户机。
3. Domain U：指运行在Xen 管理程序之上的普通客户操作系统或业务操作系统，其不能直接访问硬件资源。

### KVM

KVM 是Kernel-based Virtual Machine 的简称，中文全称叫内核虚拟机，也是一款开源软件，于 2007 年 2 月被集成到了 Linux 2.6.20 内核中。它使用Linux 自身的调度器进行管理，所以相对于Xen，其核心源码很少。KVM 采用的是基于 Intel VT 或者AMD V 的硬件辅助虚拟化技术，但是仅有KVM模块是远远不够的，因为用户无法直接访问控制内核模块去做事情，因此，必须有一个用户空间的工具才行，这个辅助的用户空间工具，开发者可以选择已经成型的开源虚拟化软件qemu，qemu是强大的虚拟化软件，kvm使用了qemu的基于X86的部分，并稍加改造，形成了可控制的KVM内核模块的用户空间工具，因此我们常听到或见到的都称作qemu-kvm。

对于KVM用户空间工具，尽管qemu可以创建和管理KVM虚拟机，但是由于其工具效率不高，不易于使用，RedHat 为KVM开发了更多的辅助工具，如libvirt，virsh，virt-manager等。

KVM模块是KVM虚拟机的核心部分。其主要功能包括：初始化CPU硬件，打开虚拟化模式，将虚拟客户机运行在虚拟机模式下，并对虚拟客户机的运行提供一定的支持。其初始化过程如下：

1. 初始化CPU硬件：KVM是基于硬件进行虚拟化，CPU必须支持虚拟化技术。
2. 打开cpu控制寄存器CR4中的虚拟化开关，并通过执行特特定指令将宿主机操作系统至于虚拟化模式中的根模式。
3. KVM 模块创建特殊设备文件/dev/kvm，并等待来自用户空间的命令。

![image-20221206203635853](https://cdn.jsdelivr.net/gh/AlexsanderShaw/BlogImages@main/img/202212062036956.png)

(https://en.wikipedia.org/wiki/Hypervisor)

# Hypervisor

## What is a Hypervisor?

A [hypervisor](https://en.wikipedia.org/wiki/Hypervisor) is computer software or hardware that enables you to host multiple virtual machines. Each virtual machine is able to run its own programs. A hypervisor allows you to access several virtual machines that are all working optimally on a single piece of computer hardware.

It can access all physical devices residing on a server. It can also access the memory and disk. It can control all aspects and parts of a virtual machine.

## **How does a Hypervisor work?**

The servers need to execute the hypervisor, and the hypervisor, in turn, loads the client operating systems of the virtual machines. The hypervisor allocates the correct CPU resources, memory, bandwidth and disk storage space for each virtual machine. A virtual machine can create requests to the hypervisor through a variety of methods, including API calls.

### Type 1 Hypervisor —— Bare metal, native

This is when the hypervisors are **run on the host's hardware** to control it as well as manage the virtual machines on it. If you are currently using [Microsoft Hyper-V hypervisor](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/), [VMware ESX/ESXi](https://www.pluralsight.com/blog/it-ops/what-is-vmware-esx-server-and-why-you-need-it), [Oracle VM Server for x86](https://www.oracle.com/virtualization/technologies/vm/downloads/server-storage-vm-downloads.html#OVMx86), [KVM](https://www.linux-kvm.org/page/Main_Page), or [Citrix XenServer](https://www.citrix.com/downloads/citrix-hypervisor/product-software/xenserver-70-standard-edition.html), then this is the type of hypervisor with which you are working.

### Type 2 Hypervisor —— **Embedded, hosted**

These hypervisors are run **as software using an operating system** such as Windows, Linux or FreeBSD. This is what the Virtage hypervisor, VirtualBox and VMWare Workstation are classified as. Examples of type II hypervisors include [Parallels Desktop for Mac](https://www.parallels.com/pd/general/?gclid=CjwKCAjwwL6aBhBlEiwADycBICKQ_LCP79KAJHwqSCMfi5Ygmk0hxwgpwn6h8mndVWTaBHS8cIW7mhoCYowQAvD_BwE), [Windows Virtual PC](https://support.microsoft.com/en-us/topic/description-of-windows-virtual-pc-262c8961-90e5-1125-654f-d87cd5ba16f8), [Oracle Virtual Box](https://www.virtualbox.org/), and [VMware Workstation](https://www.vmware.com/products/workstation-pro.html).

## Reference

[1]. **[VMware Virtualization 101 Hands-on Lab](https://customerconnect.vmware.com/evalcenter?p=virtualization-hol-22)**

[2]. [Virtualization 101: What is a Hypervisor?](https://www.pluralsight.com/blog/it-ops/what-is-hypervisor)

[3]. https://www.pluralsight.com/blog/it-ops/vmware-vsphere-best-practices