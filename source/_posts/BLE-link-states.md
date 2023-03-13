---
title: 从机设备协议栈——链路层——草稿
date: 2023/3/7
categories: 
- BLE协议栈
---

<center>
BLE 从机设备协议栈实现——链路层相关。
</center>

<!--more-->

***



这里，我们只介绍与当前协议栈所实现功能相关的几个简单的链路层状态

- Standby State：待机状态，不会发送或接收任何包。
- Advertising State：广播状态，向外发送广播数据，同时监听广播通道上的包（如连接请求）。
- Scanning state：扫描状态，监听广播通道上的数据包。
- Initiating state：连接发起态，该状态下会在广播信道上监听一个特定地址设备的广播包，并向其发起连接。
                      可以认为，该状态表示设备（如手机）向目标设备发起连接，并且连接还未完成的这段时间。
- Connection state：连接态。
  对于发起连接的设备来说（例如手机），进入连接态后会作为主设备（Central Role）。
  而被连接的设备，进入连接态后作为从设备/外围设备（Peripheral Role）

关于  Synchronization State 状态以及  Isochronous Broadcasting State 状态，本协议栈目前没有涉及，这里不做介绍。


BLE的链路层数据包空中传输顺序：
- 对于一个字节内的8个bit：除了CRC（ Cyclic Redundancy Check），其它都是LSB（ Least Significant Bit）先传输。
- 对于多个字节：除了CRC和MIC（Message Integrity Check），其它都是先传输低字节。


设备地址（DEVICE ADDRESS）：
- Public device address：全球唯一的蓝牙设备地址，需要购买。
- Random device address：随机地址，又分静态随机地址（Static Device Address）和私有地址（Private Device Address）
  -  Static Device Address：随机地址，一般使用一个固定的随机地址，或者上电后重新生成一个随机地址。
  - Private Device Address：这种类型地址和蓝牙的隐私特性相关，支持隐私特性的蓝牙设备会会周期性（例如10分钟）地改变自己的设备地址，使得自己不能被追踪。
                            但有时候我们又期望其实设备的地址在变动，之前和该设备连接过的设备（如手机），还能“识别”出该设备。
      为此，蓝牙的私有地址也分为两种：可解析私有地址（Resolvable private address） 和不可解析私有地址（Non-resolvable private address）。

      例如，当一个蓝牙设备向外广播数据时，如果它的地址是固定的，那么就可以被追踪。
      如果它周期地改变自己的地址，可以避免被持续追踪（当然，你不能在广播数据里广播特定数据，否则别人根据这个数据还是能追踪到设备），此时就可以使用 Non-resolvable private address。
      
      另一方面，有时我们需要让手机和设备连接绑定后，即使该设备不停改变自己的地址，手机仍旧能“认识”该设备，此时该设备就可以使用Resolvable private address。 
      该地址通过身份解析秘钥（IRK）生成，之前连接配对时，设备会将该IRK发送给手机。之后，即使设备不停变动其地址，手机仍旧可以通过IRK 解析出该地址，从而识别出该设备。


PACKET FORMAT FOR THE LE UNCODED PHYS：
preamble：前导数据1（1M PHY）或2(2M PYH)字节，硬件用来进行频率同步的。交替的0/1 bit流，其最低位需要和access address的LSB保持一致。
Access Address：BEL使用的通道有限（40）个，而一个区域内可能会存在很多BLE设备，为了减少物理通道上的数据冲突，每个空中数据包在前导字节后，是4字节的Access Address。
                对于广播，使用固定的值0x8E89BED6。
                对于连接，每次连接时，master在发起连接时命令中会带有这个值，后续该连接的所有交互包都使用这个access address。
PDU：协议数据单元，具体协议数据。里面放的是广播协议数据，或连接后的协议数据。
CRC：校验码。
Constant Tone Extension：与定位的AOA,AOD相关。

PACKET FORMAT FOR THE LE CODED PHY：  符号率是1Msym/s, 1个Symbol是1 bit
增加了前向纠错，所以需要更多的bit来传输同样的数据，在S=2时，2个Symbol传输1 bit，数据率是500Kbps；在S=8时，8个Symbol传输1 bit，数据率是125Kbps



BIT STREAM PROCESSING：
CRC generation：用来校验数据的完整性，确认接收到的一个数据包，其内容是完整的。应用在PDU上，crc计算需要一个初始值。
- 对于连接后的数据，计算crc的初始值由 连接请求中的指定。
- 对于广播数据，初始值为固定的0x555555
  
DATA WHITENING：用来放置连续的0或1导致 ，应用在 PDU和CRC上。使用linear feedback shift register (LFSR)进行白化。其初值：
- Position 0 is set to one.
- Positions 1 to 6 are set to the channel index of the channel used when 
transmitting or receiving, from the most significant bit in position 1 to the 
least significant bit in position 6.


4 AIR INTERFACE PROTOCOL：
T_IFS：帧间隔，150us，同一通道上的两个连续数据包间的间隔，前一个包的最后一bit 和下一个包的 第一个bit 间的间隔。

TIMING REQUIREMENTS：ble的“间歇性”通信的，双方都需要知道下次通信的“时间点”，因为时钟精度是个需要考虑的参数。
            例如， 如果使用的时钟的精度是2000 ppm（长期），短期抖动是45us，那么如果约定 1s后通信， 接收方需要提前 2045 µs快开始监听，并持续到延后的 2045 µs。
- master(Central)的时钟精度在 连接请求中指示
- slave(Peripheral)使用的精度应等于 +-500ppm 或更好。

**注意**：最终需要提前和延后的窗口，需要考虑主/从两边


Window widening： 由于时钟精度导致需要额外监听的时间（监听时间点的前后都要加），称为窗口扩展。


Advertising interval:
  T_advEvent = advInterval + advDelay
  advDelay：目的是加一个随机扰动，如果两个设备刚好同时广播，且广播interval也一直，如果没有这个随机延迟，就会一直冲突。
  advertising interval (advInterval) shall be an integer multiple of 0.625 ms in the range 20 ms to 10,485.759375 s.

connection created：主机发送了发起连接指令，从机收到了该指令
connection established：主从收到对方发过来的第一个数据通道的数据包。

Connection events：BLE周期性通信，在每个连接事件中，主从设备会交替发送/接收数据。主/从设备任意一方都可以结束本次连接事件。
- connEventCounter：连接事件计数，连接建立后为0，每次连接事件到来加1（即使因为一些原因，本次事件主/从设备未通信上，例如干扰）。

连接事件的时间，实际由connection interval (connInterval), subrate base event (connSubrateBaseEvent), subrate factor (connSubrateFactor), continuation number (connContinuationNumber), and Peripheral latency (connPeripheralLatency) 共同决定。
connInterval shall be a multiple of 1.25 ms in the range 7.5 ms to 4.0 s.

connection interval:The connInterval shall be a multiple of 1.25 ms in the range 7.5 ms to 4.0 s.

亚速率是为了使主从设备可以使用更少了连接事件（减少同步次数，节省功耗），中央设备仅在亚速率连接事件上发送数据，
The value of connSubrateFactor shall be in the range 1 to 500 and shall be set  to 1 for a new connection.
The value of connContinuationNumber shall be in the range 0 to connSubrateFactor - 1 and shall be set to zero for a new connection.
The Central shall only transmit on subrated connection  events and, if the continuation number is non-zero, continuation events.

Peripheral latency ：允许从机可以跳过几个亚速率连接事件，减少从机功耗。
connPeripheralLatency shall be an integer such that connSubrateFactor × (connPeripheralLatency + 1) is less than or equal to 500 and connInterval × connSubrateFactor × (connPeripheralLatency + 1) is less than half connSupervisionTimeout
注意：
- 如果在应用peripheral latency后，在某个从机应该监听的连接事件上，没收到主机发送过来的有效包，那么从机后续应该监听每一个亚速率事件（期间关闭应用peripheral latency），直到收到一个有效包。

Supervision timeout: 
- 发起连接后， 6 * connInterval 都没有建立连接，则认为连接丢失，并结束连接过程。
- connSupervisionTimeout: multiple of 10 ms in the range 100 ms to  32.0 s and it shall be larger than
    (1 + connPeripheralLatency) × connSubrateFactor × connInterval × 2.
- transmit window：central(master)：central 发起连接指令后，第一条数据包会在该窗口内发出。
  第一条数据包的到达时间： transmitWindowDelay + transmitWindowOffset <=  t  <= transmitWindowDelay + transmitWindowOffset +transmitWindowSize

Closing connection events：
- 当主/从数据包中，都没有MD 标记（没有更多数据了），结束本次连接事件。否则，master接可能继续发送数据（本事件内）
  
PS： MD 是用来指示一个连接事件内主从还要不要继续交互，而前面的connContinuationNumber是用来指示主机还要不要在下一个（几个）连接事件交互。

Sleep clock accuracy：由于时钟精度的问题，从机应该在每个连接事件发生时进行重新同步。


Channel Selection：
  unmappedChannel = (lastUnmappedChannel + hopIncrement) mod 37
  remappingIndex = unmappedChannel mod numUsedChannels


Acknowledgment and flow control：
- transmitSeqNum：用来标识链路层所发出的包，进入连接态后初值为0。
                  对于每个新发送的数据包，链路header中的SN设置为该值。
                  对于接受的数据包，如果SN和nextExpectedSeqNum不同，表明这是一个重发的包，nextExpectedSeqNum值保持不变。如果SN 和该值相同，说明是一个新包，nextExpectedSeqNum+=1 

- nextExpectedSeqNum：给对方上一个包的ack，或是让对方重传上一个包，初值为0。
                      发送数据包时，链路header中的NESN设置为该值。
                      当收到一个数据包，如果NESN和 transmitSeqNum 相同，表明自己上一次发送的包没有被ack，需要重发。如果不同，表明上一个发送的包被对方ack了。transmitSeqNum+=1
- 当收到一个CRC校验错误的数据包，我们期望对方重发这个包，所以自己的nextExpectedSeqNum 不应该改变。此外，由于这个包crc错了，包里的NESN也是不置信的，所以我们需要重发自己上一次发过的包，transmitSeqNum不变。


Flow control：
链路层可能因为CRC校验不过，需要对方重发，所以不改变自己的nextExpectedSeqNum。
另一方面，如果当前接收buffer满了，不能放数据了，也可以不改变nextExpectedSeqNum，让对方之后再重发该包，从而缓解当前Buffer满的问题。

Data PDU length management：
-  If either value is not 27 then the Controller should initiate the Data Length Update procedure  at the earliest practical opportunity.


Connection Update procedure：
- 连接参数更新时，subrate factor需要重置为1，continuation number 重置为0
- 主机可能会自动发起连接参数更新（为了协同其它连接）
- At the start of the transmit window, the Link Layer shall reset TLLconnSupervision.
