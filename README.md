## 段页式虚拟存储管

模拟操作系统中的段页式虚拟内存管理。

### 题目描述：
* 内存大小64K，页框大小为1K，一个进程最多有4个段，且每个段最大为16K。一个进程驻留集最多为8页。
* 驻留集置换策略：局部策略（仅在进程的驻留集中选择一页）
* 页面淘汰策略：FIFO、LRU
  **要实现的功能**
1. 创建进程：输入进程名、每个段大小
2. 销毁进程：回收该进程占有的内存
3. 查看一个进程当前的驻留集、页面淘汰策略（FIFO or LRU）、段表、页表
4. 查看内存的使用情况
5. 地址映射：请求一个逻辑地址（段号、段偏移），判断是否缺页，如果不缺页，计算输出其物理地址；若缺页，根据驻留集置换策略，选择一页换出并将请求的页换入，再计算其物理地址。

### 详细设计

**一些策略：**
* 放置策略：决定一个进程驻留集存放在内存什么地方。优先存放在低页框。当一个进程进入时，从低页框选择未使用的页框。
* 初始载入策略：创建进程后，决定最开始将那些页载入内存，从第0个、第1个段...依次载入页，直到驻留集已全部载入
####一些类的说明：
1. ######Memory内存模拟类：
  由于是模拟，内存可以看作是Frame页框的数组。提供了两个方法：
* 申请内存：**mallocFrame(String id, int n) **返回申请到的页框的页框号数组。根据放置策略（优先选择低页框），从数组下标0处遍历数组，选择没有被占用的页框。
  ![img](file:///C:\Users\ADMINI~1.凯\AppData\Local\Temp\msohtmlclip1\01\clip_image002.jpg)
* 释放内存：**freeFrame(int[] frames)**释放frames数组中页框号的内存。
2. ###### PCB类：
* **initLoad()**函数中体现了初始载入策略（从第0个、第1个段...依次载入页，直到驻留集已全部载入）。
  ![img](file:///C:\Users\ADMINI~1.凯\AppData\Local\Temp\msohtmlclip1\01\clip_image004.jpg)
* 当发生缺页中断后，需要根据置换策略（FIFO or LRU）选择一个页换出内存，并将另一个页载入。**replacePage(int inSN, intinPN)**代码如下：
  ![img](file:///C:\Users\ADMINI~1.凯\AppData\Local\Temp\msohtmlclip1\01\clip_image006.jpg)
* FIFO的实现**selectReplacePage_FIFO()**：
  在PCB类中维护一个队列，记录进程中页（段号、页号）的载入顺序，每当页载入内存时入队；要将页换出时，返回队头元素。
  ![img](file:///C:\Users\ADMINI~1.凯\AppData\Local\Temp\msohtmlclip1\01\clip_image008.jpg)
  ![img](file:///C:\Users\ADMINI~1.凯\AppData\Local\Temp\msohtmlclip1\01\clip_image010.jpg)
* LRU的实现**selectReplacePage_LRU()**：
  在PageEntry页表项类中有usedTime变量，记录该页上一次被访问的时间。当该页被初始载入或被访问时，重置时间；要将页换出时，遍历进程所有页，选出usedTime最小的页，返回段号、页号。
  ![img](file:///C:\Users\ADMINI~1.凯\AppData\Local\Temp\msohtmlclip1\01\clip_image012.jpg)
  ![img](file:///C:\Users\ADMINI~1.凯\AppData\Local\Temp\msohtmlclip1\01\clip_image014.jpg)

### 功能实现（OS类）：

* 创建进程：**createProcess(String id, int[] segments)**
  检验创建进程的合法性（进程名是否重复、段的个数、每个段大小是否合法）
  创建PCB对象
  调用**mallocFrame(String id, intn)**申请内存。若内存不足，提示创建失败，返回
  调用**initLoad()**初始载入一些页
  ![img](file:///C:\Users\ADMINI~1.凯\AppData\Local\Temp\msohtmlclip1\01\clip_image016.jpg)
* 将逻辑地址映射为物理地址：**toPhysicalAddress()**
  检验（进程、段是否存在；访问地址是否越界）
  根据段偏移计算页号、页偏移
  判断访问的页是否存在，不存在调用**replacePage(int inSN, intinPN)**，选择一页换出内存，并将请求的页载入
  计算物理地址，重置该页使用时间
  ![img](file:///C:\Users\ADMINI~1.凯\AppData\Local\Temp\msohtmlclip1\01\clip_image018.jpg)
* 销毁进程：**destroyProcess(String id)**
* 查看进程：**showProcess(String id)**
* 查看内存：**showMemory()**
