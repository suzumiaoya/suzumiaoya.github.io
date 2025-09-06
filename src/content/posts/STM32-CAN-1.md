---
title: STM32 CAN通信从了解到会用
published: 2025-09-05
description: 一个简易的CAN通信教程，旨在快速入门配置和使用，较少涉及原理
tags: [STM32, 嵌入式, 通信, CAN, 电控, 单片机]
category: 嵌入式学习
draft: false
---

## CAN通信简介与其特点

CAN通信是一种**广播制**的**异步**串行**总线**通信协议。

### 总线与时钟源

与I2C类似，CAN通信通过两条总线(CAN_H和CAN_L)将所有的设备（节点）并联在总线上，各节点通过发送基于CAN_H和CAN_L的电平信号来传播信息。尽管CAN是一种异步通信协议（各节点不依赖于共同的时钟源来确定自己什么时候应该做什么事），但是电平信号的发送需要一个时钟源来提供发送的频率，因此**在配置CAN总线时，需要为CAN总线配置一个时钟源**，时钟源的频率关系着通信中的波特率。

### 广播通信机制

CAN通信的一大特点是**广播**：**每个参与到CAN通信的节点或设备都可以发送消息（报文），并且都可以被其他节点接收到，通信不是点对点的，而是直接发送到总线上的广播式的**。
PS: 与I2C不同，CAN通信中不存在所谓的“主机”和“从机”，所有参与到CAN通信中的设备都可以发送和接收消息。
由于每个设备都可以向总线上发送报文，会出现同一时间有多个设备向总线发送报文的情况，这时候就引出了CAN通信的“仲裁”机制，CAN通信的“仲裁”机制会解决这一问题。

### CAN通信的消息结构与基于CAN_ID的仲裁机制

前面讲道，每个节点都可以向总线发送报文，这个报文就是来自节点的消息，接下来介绍报文的格式。
报文分为**标准数据帧**和**拓展数据帧**。标准帧结构中，用于标示消息的发送者的CAN_ID只有11位，也就是说，参与CAN通信的对象最多只有2048个；而拓展帧结构则将CAN_ID拓展到了29位，增加了可以参与CAN通信的对象数量。为了能够快速上手和使用，本文只介绍标准帧格式。（其实拓展帧结构我也没学明白，但是一般2048个CAN_ID已经很够用了，不是我需要用到的东西，直接忽略）

* 标准数据帧的结构
  1. **标识符(CAN_ID)**: 11位长度的数据，用于**标示报文的发送者**，是编写报文和配置过滤（就是选择接收来自哪个CAN_ID的消息）时的关键。类似于发信时写在信封上的落款。
  2. **数据长度码(Data Length Code, DLC)**：4位，从0到8，**表示报文中携带的数据（数据场）的长度**。类似于信封的尺寸。
  3. **数据场(Data Field)**：0-8字节，是报文中**需要传递的数据**。类似于信封中信的内容。

在使用CAN通信时，需要为发送的报文设置以上的三种内容。

* 基于CAN_ID的仲裁机制
  CAN总线上的显性电平(逻辑0)会覆盖掉隐形电平(逻辑1)，当节点向总线发送报文时，标识符(CAN_ID)最先以二进制的形式被发送出去，CAN_ID越小，二进制中0就出现得越早，也就是说**CAN_ID越小的报文在发送报文时的优先级越高**。
  当多个节点同时发送报文时，发送的报文优先级较低的节点读取到电平上的逻辑0，就会将自己的报文发送延后，转而开始接收总线上正在传输的报文。如此一来，仲裁机制就解决了多个节点同时发送消息的优先级问题。

上文为“发送”部分的内容。接下来介绍“接收”部分的内容。

### 消息的接收与过滤器机制

前文提到CAN通信是“广播”机制的通信协议，任何节点都可以发送报文，任何节点都可以接收报文，被接收的报文会被存入一个叫FIFO的缓冲器中，在工程实际中，有时候我们只关注来自某一节点的数据，而不关注其他的数据，我们只希望将来自这一节点的数据存入FIFO并进行处理，这就引入了CAN通信中的过滤器机制。
通过设置过滤器，可以**只接收特定CAN_ID或特定范围CAN_ID的报文**，过滤器的设置有以下两种不同的模式：

1. 标识符列表模式(LISTMODE)
   * 列表模式通过设置需要关注的CAN_ID，来实现只接收包含特定CAN_ID的报文。
2. 标识符掩码模式(MASKMODE)
   * 通过设置标识符(Identifier)和掩码(MASK)，实现接收包含一定范围内CAN_ID的报文，或是包含特定CAN_ID的报文。
   * 匹配规则：**(接收到的报文ID & 掩码) == (报文配置的标识符 & 掩码)**
   * 掩码位的含义：
      1. 为 1 的位：表示这一位必须与“标识符”中对应的位相匹配。
      2. 为 0 的位：表示这一位是“通配符”，不用关心，无论是0还是1都能通过。

### 数据的处理，轮询或中断

在接收到数据后，需要对数据进行处理，除了DMA外，常见的两种模式是轮询和中断。
轮询：MCU不断对标志消息是否接收的变量进行询问，当确认被接收时，执行特定功能（一般写在while(1)循环里）
中断：MCU不关系什么时候消息被接收，当消息被接收时，会产生中断，MCU接收到中断信号后就会去执行中断函数里的内容，需要对数据进行的操作就写在中断函数里。

CAN的过滤器机制大大减少了数据的接收量，实现了精准的数据接收，**一般采用中断模式对数据进行处理**。

## CAN总线的配置与代码使用

### STM32CubeMX配置

1. 打开CAN外设
   * 在"Pinout & Configuration"视图中，选择"Connectivity"选项卡，找到"CAN"，点开并选择"Active"
   * 激活后，对应的 CAN_TX 和 CAN_RX 引脚会自动高亮显示在芯片引脚图上。检查一些这些引脚有没有被其他外设占用。
2. 配置CAN的相关参数
3.
   * 工作模式
    1. Normal：正常模式，节点会把报文发送到总线上，也可以接收总线上的报文。
    2. Loopback：回环模式，节点发送的报文会传到总线上，同时传输到自己的输入端，但是接收时只能接收到自己发出的数据，不能接收总线上的数据。
    3. Silent：静默模式，节点只会接收总线上的报文，自己不发送报文。
    4. Silent Loopback：静默回环模式，节点发出的报文不传到总线，直接传回自己的输入端，接收时也只能接收自己发出的数据。

   * 配置时钟源与波特率
    1. 首先为CAN通信配置时钟源，在"Clock Configuration"选项卡中，找到PCLK1(APB1 Peripheral clocks)，这里的频率就是传入CAN总线的时钟频率，你可以在这里进行更改。回到"Pinout & Configuration"选项卡，打开"Connectivity -> CAN -> Parameter Settings"，设置Prescaler（预分频器，会对APB1上从来的频率进行分频），Time Quanta in Bit Segment 1和Time Quanta in Bit Segment 2，这三个参数决定了最后的波特率，计算公式:
      > Baud Rate = CAN_Clock / (Prescaler * (1 + TS1 + TS2))
    2. 自动重传：根据你的项目情况进行设置，开启后如果报文发送失败，CAN控制器会自动重新发送。

   * 配置中断向量
    * 选择NVIC Settings，根据你的需要启动任意一个TX，RX的中断向量，TX和RX后面的数字代表着该向量对应的FIFO编号，TX0，RX0对应FIFO0，TX1，RX1对应FIFO1。你可以在System -> NVIC中设置优先级。

### HAL代码使用

1. 定义必要变量
   定义用于发送和接收的报文头结构体，以及存储发送/接收数据的数组：

   ```C
      CAN_TxHeaderTypeDef   TxHeader;
      CAN_RxHeaderTypeDef   RxHeader;

      uint8_t               TxData[8];
      uint8_t               RxData[8];

      uint32_t              TxMailbox; // 用于存储发送邮箱编号
   ```

   CAN的初始化函数**HAL_CAN_Init()**会被CubeMX自动生成并调用。
2. 配置过滤器与FIFO
   在Private User Code区域写入以下代码：
   * 掩码模式(MASKMODE)

   ```C
    void CAN_Filter_Config(){
      CAN_FilterTypeDef  sFilterConfig; // 声明过滤器结构体

      // 1. 配置过滤器
      sFilterConfig.FilterBank = 0;                        // 选择过滤器组0 (0-13 or 0-27, 不同的MCU可能会有所不同)
      sFilterConfig.FilterMode = CAN_FILTERMODE_IDMASK;      // 设置为掩码模式
      sFilterConfig.FilterScale = CAN_FILTERSCALE_32BIT;       // 设置为32位位宽

      // 2. 设置要接收的ID和掩码
      //    对于标准帧(11位), ID必须左移5位，以对齐32位寄存器的高位
      //    假设要接收ID为**0x321**的报文
      sFilterConfig.FilterIdHigh = (0x321 << 5);             // 32位ID的高16位 (对于标准帧，CAN_ID在这里，一般只用配置这一部分)
      sFilterConfig.FilterIdLow = 0x0000;                      // 32位ID的低16位 (RTR, IDE, EXID等位，暂时不用考虑)

      //    掩码位为1表示必须匹配, 为0表示不关心(通配符)
      //    为了精确匹配0x321，掩码也设置为0x321，并确保所有相关位都为1
      sFilterConfig.FilterMaskIdHigh = (0x7FF << 5);         // 掩码的高16位, 0x7FF等于全为1（16进制的7FF转化为2进制就是11个1），覆盖11位ID的所有位，根据你的实际需要修改这里的16进制数字，使之可以匹配你希望接收的CAN_ID报文
      sFilterConfig.FilterMaskIdLow = 0x0000;                  // 掩码的低16位

      // 3. 分配过滤器到FIFO
      sFilterConfig.FilterFIFOAssignment = CAN_RX_FIFO0; // 将通过此过滤器的报文关联到FIFO0，这个FIFO的编号要和你上面配置中断时的编号对应

      // 4. 激活过滤器
      sFilterConfig.FilterActivation = ENABLE;

      // 5. 调用HAL库函数应用配置
      if (HAL_CAN_ConfigFilter(&hcan, &sFilterConfig) != HAL_OK)
      {
        /* Filter configuration Error */
        Error_Handler();
      }

    }
   ```

   * 列表模式(LISTMODE)

   ```C
    void CAN_Filter_Config(){
      CAN_FilterTypeDef  sFilterConfig;

      // 1. 配置过滤器
      sFilterConfig.FilterBank = 1;                         // 使用过滤器组1, 避免与之前的冲突
      sFilterConfig.FilterMode = CAN_FILTERMODE_IDLIST;       // 设置为列表模式
      sFilterConfig.FilterScale = CAN_FILTERSCALE_16BIT;        // 设置为16位位宽

      // 2. 设置要接收的ID列表 (关键步骤)
      //    在16位列表模式下，一个32位的过滤器被拆分为两个16位的ID槽位
      //    ID同样需要左移5位
      sFilterConfig.FilterIdHigh = (0x100 << 5);              // 第一个ID: 0x100
      sFilterConfig.FilterIdLow = (0x200 << 5);               // 第二个ID: 0x200

      //    在列表模式下, Mask寄存器被用作另外两个ID槽位
      //    如果还想接收0x300和0x400
      sFilterConfig.FilterMaskIdHigh = (0x300 << 5);          // 第三个ID: 0x300
      sFilterConfig.FilterMaskIdLow = (0x400 << 5);           // 第四个ID: 0x400
      //    注意：在16位列表模式下，一个过滤器组最多可以精确匹配4个标准帧ID。
      //    如果只需要匹配两个ID，将MaskIdHigh和MaskIdLow也设置成IdHigh和IdLow即可。

      // 3. 分配过滤器到FIFO
      sFilterConfig.FilterFIFOAssignment = CAN_RX_FIFO0;    //与上相同，FIFO编号匹配RX，TX中断的编号

      // 4. 激活过滤器
      sFilterConfig.FilterActivation = ENABLE;

      // 5. 应用配置
      if (HAL_CAN_ConfigFilter(&hcan, &sFilterConfig) != HAL_OK)
      {
      /* Filter configuration Error */
      Error_Handler();
      }

    }
   ```

3. 启动CAN控制器和中断
   在配置好过滤器后，需要调用以下函数才能使CAN控制器开始工作：

   ```C
    HAL_CAN_Start(&hcan);
   ```

   如果使用中断模式处理数据，需要使用以下代码来激活中断

   ```C
    HAL_CAN_ActivateNotification(&hcan, CAN_IT_RX_FIFO0_MSG_PENDING) //注意FIFO的编号；RX表示接收中断，TX表示发送中断
   ```

   >以上两行代码可以直接写在CAN_Filter_Configuration里，也可以根据你的需要在特定的位置使用。
4. 发送数据
   发送数据分为三步：填充报文头，填充要发送的数据，调用发送函数
   1. 填充报文头

   ```C
    TxHeader.StdId = 0x123;         // 设置标准ID
    TxHeader.ExtId = 0x00;           // 扩展ID，标准帧模式下不使用
    TxHeader.RTR = CAN_RTR_DATA;   // 帧类型：数据帧
    TxHeader.IDE = CAN_ID_STD;       // 帧格式：标准帧
    TxHeader.DLC = 8;                // 数据长度 (0-8字节)
    TxHeader.TransmitGlobalTime = DISABLE;
   ```

   2. 准备要发送的数据

   ```C
    将你要发送的数据填入上文的TxData[]里
   ```

   3. 调用发送函数

   ```C
    HAL_CAN_AddTxMessage(&hcan, &TxHeader, TxData, &TxMailbox);
   ```

   >以上的三部分代码可以合并成一个发送函数，在你需要发送数据时直接使用。
5. 接收数据
   推荐使用中断模式进行，在USER_CODE_BEGIN_04部分写入中断回调函数：

   ```C
    void HAL_CAN_RxFifo0MsgPendingCallback(CAN_HandleTypeDef *hcan)
      {
          // 1. 获取接收到的消息
          if (HAL_CAN_GetRxMessage(hcan, CAN_RX_FIFO0, &RxHeader, RxData) == HAL_OK)
          {
              // 2. 可以在这里根据 RxHeader.StdId 判断是哪个节点发来的数据
              if (RxHeader.StdId == 0x321)
              {
                  // 3. 处理 RxData 中的数据
                  // 例如：点亮一个LED，控制电机等
              }
          }
      }
   ```

   >如果需要在发送数据后进行操作，则在相同的地方写出以下函数的实现：

   ```C
   void HAL_CAN_RxFifo0MsgPendingCallback(CAN_HandleTypeDef *hcan)
   ```
