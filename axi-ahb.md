# axi-ahb  

疑问：  
1. strb的用处有哪些？用于narrow传输，存在数据总线宽度大于传输数据的情形，此时使用strb进行数据选通。
2. ahb中，例举一个从access到setup状态的例子。从状态机的迁移图可知，在access阶段，psel依旧置高，并可以进入setup阶段    
3. axi中，为啥回环传输的长度只能是2，4，8，16次传输？<font color=#660000">猜测是为了解决地址对齐问题。如果为2的倍数，可以通过左移右移快速得到对齐地址。</font>  
4. axi中Lower_Byte_Lane是如何决定的？  
5. Byte invariance是要解决那个问题，以及如何解决的？主机处理多个ID，每个ID有多个事务时，此时存在同一ID的多个事务不能紧密的返回，只是可以先处理另一个ID的事务。在此过程中，一定要保持同一ID的多个事务的顺序不变。  

## ahb  

// TODO  

## axi  

主要特性：  
1. 控制信息和数据信息通道的分离  
2. Register Slices。由于各个通道之间没有明显的关系，因此可以以延时为代价提高系统频率；以存储开销为代价减少关键路径。
3. burst传输可以减少控制信息通道的使用  
4. outstanding 以及 o-o-o 传输  

## 控制信息和数据信息的分离  

可将所有的数据传输分类为如下几个通道：  
读相关：AR R  
写相关：AW W B  

### 读相关  


![Global Signal](img/global_signal.png)  
<center>全局信号</center>  
&emsp;  
&emsp;  

![AR Signal](img/AR%20Signal.png)  
<center>读地址通道信号</center>  
&emsp;  
&emsp;  

地址通道信息可以从以下思考点分类：  
名字：ARID  
地址：ARADDR  
需要读多少数据：ARLEN, ARSIZE  
传输类型：  
   1. 地址类型：ARBURST
   2. transaction类型：ARLOCK, ARCACHE, ARPORT  

暂时不理解的信号：ARQOS, ARREGION, ARUSER  
同步信号：ARVALID, ARREADY  


![R Signal](img/R_Signal.png)  
<center>读数据通道信号</center>  
&emsp;  
&emsp;  

读数据通道信号：  
名字：RID  
数据：RDATA  
交易是否是最后一笔：RLAST  
是否传输成功：RRESP  
暂时不理解：RUSER  
同步信号：RREADY  

### 写相关  

写相关的写地址通道信号与读地址通道信号一致  

由于写数据与写回应的信号发出主体不一致，因此需要一个单独的回应通道。  

![writer response channel](img/B_channel.png)  
<center>写回应通道信号</center>  
&emsp;  
&emsp;  


## 读写交易  

VALID：表明master将相关控制或者数据信息准备妥当  
READY：表明slave已经准备按master的信号进行相应的动作  

### 读写交易信号的先后关系  

![Read transaction handshake dependencies](img/Read%20transaction%20handshake%20dependencies.png)  
<center>Read transaction handshake dependencies</center>  
&emsp;
&emsp;  

![AXI3 write transaction handshake dependencies](img/AXI3%20write%20transaction%20handshake%20dependencies.png)  
<center>AXI3 write transaction handshake dependencies</center>  
&emsp;
&emsp;  

![AXI4 and AXI5 write transaction handshake dependencies](img/AXI4%20and%20AXI5%20write%20transaction%20handshake%20dependencies.png)    
<center>AXI4 and AXI5 write transaction handshake dependencies</center>  
&emsp;
&emsp;  

### 地址结构  

AxLEN：4bit
1. AXI3：[1:16]  
2. AXI4：burst type = incr [1:256]; else [1:16]。此时AxLEN为8bit  
3. if burst_type == wrapping, AxLEN = [2|4|8|16]  

AxSIZE：3bit
1. [1|2|4|.....|64|128]  

AxBURST:2bit:  
1. FIXED：地址不变。用于FIFO的装载或排空  
2. INCR：地址递增。用于常规的内存顺序访问  
3. WRAP：开始地址必须对齐于每一个transfer的大小。长度必须为2，4，8，16.  

xRSP：  
RRESP：对于一笔burst，可以有多个RRSP  
BRESP：对于一笔burst，只能有一个BRESP  

## transaction属性  

//TODO  

## O-O-O  

Manager使用标识符识别不同的交易。相同ID必须严格保持顺序，不同ID的transaction顺序没有做限制。  

读数据顺序：从机必须确保返回数据的RID值是ARID所想要的；互联架构在返回同一个主机对不同从机相同ARID时的情形时，必须确保以主机发送的顺序返回；读数据重排序深度决定了从机能够重排序的个数，当深度为1时，表示全都顺序执行。  

写数据顺序：主机必须以事务地址相同的顺序发送写数据；互联架构在组合不同主机的写事务时，必须确保以地址的顺序转发。  

**在相同的通道上，具有相同ID和相同目的地的事务，必须保持有序**  

**具有相同ID的事务回应，其返回顺序和发射顺序一样**  

//TODO 可以构造一些相应的场景进行阐释
