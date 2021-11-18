### Liunx下CPU信息

* 处理器核数

    ```shell
    cat /proc/cpuinfo|grep "cpu cores"|uniq
    ```

* 逻辑处理器核数

    ```shell
    cat /proc/cpuinfo|grep
    # 如果"siblings"和"cpu cores"一致，则说明不支持超线程或者超线程没有开启
    # 如果"siblings"和"cpu cores"的两倍，则说明支持超线程，并且开启超线程
    ```

* 系统物理处理器封装ID

    ```shell
    cat /proc/cpuinfo|grep "physical id"|sort|uniq|wc -l 或者lscpu|grep "CPU socket"
    ```

* 系统逻辑处理器ID

    ```shell
    cat /proc/cpuinfo|grep "processor"|wc -l
    ```

### 各种CPU信息说明

* 处理器核数 ：processor cores，即俗称"CPU核数"，也就是每个物理CPU中core的个数，例如"Inter(R) Xeon(R) CPU E5-2680 v2 @ 2.8GHz"是10核处理器，它在每个socket上有10个"处理器核"。具有相同core id的CPU是同一个core的超线程。

* 逻辑处理器核心数：sibling是内核认为的单个物理处理器所有的超线程个数，也就是一个物理封装中的逻辑核的个数。如果sibling等于实际物理核数的话，就说明没有启动超线程；反之，则说明启用超线程。
* 系统物理处理器封装ID：Socket中文翻译成"插槽"，也就是所谓的物理处理器封装个数，即俗称"物理CPU数"，管理员称为"路"。例如一块"Inter(R) Xeon(R) CPU E5-2689 v2 @ 2.8GHz"有两个"物理处理器封装"。具有相同physical id的CPU是同一个CPU封装的线程或核心。
* 系统逻辑处理器ID：逻辑处理器数的英文名是logical processor，即俗称的"逻辑CPU数"，逻辑核心处理器就是虚拟物理核心处理器的一个超线程技术。例如"Inter(R) Xeon(R) CPU E5-2689 v2 @ 2.8GHz"支持超线程，一个物理核心能模拟两个逻辑处理器，即一块"Inter(R) Xeon(R) CPU E5-2689 v2 @ 2.8GHz"有20个"逻辑处理器"。
