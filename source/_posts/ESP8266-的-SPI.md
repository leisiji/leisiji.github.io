---
title: ESP8266 的 SPI
date: 2018-01-28 14:57:26
tags: ESP8266
---

# 硬件的连接

我的手头上没有其他的 SPI 外设，刚好有两个 ESP8266，找了一下网上的 ESP8266 SPI 的例程，刚好就是两个 ESP8266 的 SPI 通信。
管脚图如下：
<!--more-->
![](/images/管脚图.png)
管脚的复用如下：

- GPIO14 is SPI Clock
- GPIO15 is SPI CS pin
- GPIO12 is SPI MISO
- GPIO13 is SPI MOSI

> 注意：不要使用 SD0、SD1、CLK，我之前用了这个通信不成功。而且 MISO 是接 MISO，不是 MOSI。

# 程序的编写

[例程](https://github.com/espressif/esp8266-rtos-sample-code) 在 Periphal 的 `HSPI_SLave` 里面。

一些重要的参数在 `spi_register.h` 的里面。

# 程序的详解
```C
// Test spi master interfaces.
void spi_master_test()
{
    SpiAttr hSpiAttr;
    hSpiAttr.bitOrder = SpiBitOrder_MSBFirst;
    hSpiAttr.speed = SpiSpeed_16MHz;
    hSpiAttr.mode = SpiMode_Master;
    hSpiAttr.subMode = SpiSubMode_0;

    // Init HSPI GPIO
    WRITE_PERI_REG(PERIPHS_IO_MUX, 0x105);
    PIN_FUNC_SELECT(PERIPHS_IO_MUX_MTDI_U, 2);//configure io to spi mode
    PIN_FUNC_SELECT(PERIPHS_IO_MUX_MTCK_U, 2);//configure io to spi mode
    PIN_FUNC_SELECT(PERIPHS_IO_MUX_MTMS_U, 2);//configure io to spi mode
    PIN_FUNC_SELECT(PERIPHS_IO_MUX_MTDO_U, 2);//configure io to spi mode

    SPIInit(SpiNum_HSPI, &hSpiAttr);
    uint32 value = 0xD3D4D5D6;
    uint32 sendData[8] ={ 0 };
    SpiData spiData;

    printf("\r\n =============   spi init master   ============= \r\n");

//  Test 8266 slave.Communication format: 1byte command + 1bytes address + x bytes Data.
    printf("\r\n Master send 32 bytes data to slave(8266)\r\n");
    memset(sendData, 0, sizeof(sendData));
    sendData[0] = 0x55565758;
    ....
    sendData[7] = 0x71727374;
    spiData.cmd = MASTER_WRITE_DATA_TO_SLAVE_CMD;
    spiData.cmdLen = 1;
    spiData.addr = &value;
    spiData.addrLen = 4;
    spiData.data = sendData;
    spiData.dataLen = 32;
    SPIMasterSendData(SpiNum_HSPI, &spiData);
  

    printf("\r\n Master receive 24 bytes data from slave(8266)\r\n");
    spiData.cmd = MASTER_READ_DATA_FROM_SLAVE_CMD;
    spiData.cmdLen = 1;
    spiData.addr = &value;
    spiData.addrLen = 4;
    spiData.data = sendData;
    spiData.dataLen = 24;
    memset(sendData, 0, sizeof(sendData));
    SPIMasterRecvData(SpiNum_HSPI, &spiData);
    printf(" Recv Slave data0[0x%08x]\r\n", sendData[0]);
    ...

    value = SPIMasterRecvStatus(SpiNum_HSPI);
    printf("\r\n Master read slave(8266) status[0x%02x]\r\n", value);

    SPIMasterSendStatus(SpiNum_HSPI, 0x99);
    printf("\r\n Master write status[0x99] to slavue(8266).\r\n");

}
```

- `SpiAttr` 显然是用来初始化 SPI，关于 SPI 的设置参数就不会详细解释了，网上应该有许多的教程了。
- 接着的寄存器操作看[这里](https://bbs.espressif.com/viewtopic.php?t=1342)的解释。简单来说 `WRITE_PERI_REG(PERIPHS_IO_MUX, 0x105)` 是在配置 SPI80M 时钟时使用。
- 之后就是将 GPIO12、13、14、15 设置对应的功能。
- `memset`: 作用是在一段内存块中填充某个给定的值，它对较大的结构体或数组进行清零操作的一种最快方法。
- `value = 0xD3D4D5D6` 是写入 spiData 的 addr，这个是一直找不到是什么的地址。留作以后吧。


从机接受信号之后返回信号
```C
// Test spi slave interfaces.
void spi_slave_test()
{
    //
    SpiAttr hSpiAttr;
    hSpiAttr.bitOrder = SpiBitOrder_MSBFirst;
    hSpiAttr.speed = 0;
    hSpiAttr.mode = SpiMode_Slave;
    hSpiAttr.subMode = SpiSubMode_0;

    // Init HSPI GPIO
    WRITE_PERI_REG(PERIPHS_IO_MUX, 0x105);
    PIN_FUNC_SELECT(PERIPHS_IO_MUX_MTDI_U, 2);//configure io to spi mode
    PIN_FUNC_SELECT(PERIPHS_IO_MUX_MTCK_U, 2);//configure io to spi mode
    PIN_FUNC_SELECT(PERIPHS_IO_MUX_MTMS_U, 2);//configure io to spi mode
    PIN_FUNC_SELECT(PERIPHS_IO_MUX_MTDO_U, 2);//configure io to spi mode

    printf("\r\n ============= spi init slave =============\r\n");
    SPIInit(SpiNum_HSPI, &hSpiAttr);
    //CLEAR_PERI_REG_MASK(SPI_SLAVE(SpiNum_HSPI), 0x3FF);

    SPISlaveRecvData(SpiNum_HSPI,spi_slave_isr_sta);
    uint32 sndData[8] = { 0 };
    ...
    sndData[7] = 0x21201f1e;

    SPISlaveSendData(SpiNum_HSPI, sndData, 8);
    WRITE_PERI_REG(SPI_RD_STATUS(SpiNum_HSPI), 0x8A);
    WRITE_PERI_REG(SPI_WR_STATUS(SpiNum_HSPI), 0x83);
}
```


# 注意事项

由于需要直接接入 GPIO15，所以一开始不能够先接入 GPIO15。原因如下：
MTD0 (也叫做 GPIO15) : GPIO0 : GPIO2 控制 boot 的模式，

| mode | GPIO 0 |  GPIO 2 |  GPIO 15|
|----|-----|----|-----|
|UART Download Mode (Programming)|    0|   1|   0|
|Flash Startup (Normal)|  1|   1|   0|
|SD-Card Boot|    0|   0|   1|


# LoRa 模块

LoRa 的通信采用 SPI，一下就是记录连接和修改代码的过程。
在 Arduino 上连接是非常的方便，只需要找到对应的 API 写代码就可以了。
下面讲的主要是将 Arduino 的代码移植到 ESP8266 的 RTOS 上。
移植后的代码在 [这里](https://github.com/leisiji/Study_ESP8266/tree/master/MyDriver)
注意：有一些功能尚未移植。
下面会挑一些代码出来讲解，讲述开发过程的坑。

## SX1278.c
```C
/* reset lora */
void SX1278_Reset(SX1278_hw_t *hw){
  // reset pin setup
  PIN_FUNC_SELECT(PERIPHS_IO_MUX_GPIO4_U, FUNC_GPIO4);
  // cs pin setup
  PIN_FUNC_SELECT(PERIPHS_IO_MUX_MTDO_U, FUNC_GPIO15);

  // perform reset
  GPIO_OUTPUT_SET(hw->reset,0);
  vTaskDelay(10/portTICK_RATE_MS);
  GPIO_OUTPUT_SET(hw->reset,1);
  vTaskDelay(10/portTICK_RATE_MS);
}
```
注意 LoRa 复位：需要维持 nss 管脚 10us 以上的低电平，在等待 5ms 以上的高电平。

> GPIO15，也就是 NSS 管脚需要设置为普通的输出模式，作为软选择管脚，绝对不能设置为 `FUNC_HSPI_CS0` 的特殊功能。

```C
/* SPI 发送数据，返回的参数可以检测 SPI 是否发送成功 */
int singleTransfer( SX1278_hw_t *hw, uint8_t address, uint8_t value)
{
  uint8_t response;
  /* 测试 SPI 的连通性 */
  GPIO_OUTPUT_SET(hw->nss, 0);
  SPISettings defalut_SPISettings = { 8E6, MSBFIRST, SPI_MODE0};

  beginTransaction(&defalut_SPISettings);
  transfer(address);
  response = transfer(value);
  endTransaction();
  GPIO_OUTPUT_SET(hw->nss, 1);

  return response;
}

/* SPI.h 中结构体的定义 */
typedef struct 
{
    uint32_t clock;
    uint8_t  bitOrder;
    uint8_t  dataMode;
}SPISettings;
```
从上面可以看到 Arduino 中的 API 的 transfer 是先要传入一个地址，后来再运行一次函数才会向地址传入值。

而在 SPI 的 transfer 函数中：
```C
uint8_t transfer(uint8_t data) {
    while(SPI1CMD & SPIBUSY) {}
    // reset to 8Bit mode
    setDataBits(8);
    SPI1W0 = data;
    SPI1CMD |= SPIBUSY;
    while(SPI1CMD & SPIBUSY) {}
    return (uint8_t) (SPI1W0 & 0xff);
}
```
ESP8266 的 SPI 通信格式为：命令（7 位） + 地址（1 位） + 读写数据（8 位）。总共 16 字节。所以传输分两次完成。


而且通过 SPI 读取和写入 LoRa 的寄存器稍微不同（地址的运算不同）：
```C
u8_t readRegister(SX1278_hw_t *hw, uint8_t address)
{
  return singleTransfer(hw, address & 0x7f, 0x00);
}

void writeRegister(SX1278_hw_t *hw, uint8_t address, uint8_t value)
{
  singleTransfer(hw, address | 0x80, value);
}
```
最后就是接收数据的回调函数：
```C
void onReceive(SX1278_hw_t *hw, void(*callback)(int)){
  _onReceive = callback;
  if (callback)
  {
    writeRegister(hw, REG_DIO_MAPPING_1, 0x00);

    /* config interrupt on dio0 */
    GPIO_ConfigTypeDef gpio_config_type;
    gpio_config_type.GPIO_IntrType = GPIO_PIN_INTR_POSEDGE;// rising edge interrupt
    gpio_config_type.GPIO_Pullup = GPIO_PullUp_EN;
    gpio_config_type.GPIO_Mode = GPIO_Mode_Input;
    gpio_config_type.GPIO_Pin = GPIO_Pin_5;

    gpio_config(&gpio_config_type);
    GPIO_REG_WRITE(GPIO_STATUS_W1TC_ADDRESS, GPIO_Pin_5);

    gpio_intr_handler_register(onDio0Rise, hw);
    _xt_isr_unmask(1 << ETS_GPIO_INUM);     // Enable the GPIO interrupt
  }else{
    _xt_isr_mask(1 << ETS_GPIO_INUM);       // Disable the GPIO interrupt
  }
}
```
接收回调需要监听 DIO0 的上升沿中断，特别注意 `gpio_intr_handler_register` 的写法，回调函数的参数不能写在函数那里，要传入注册函数的第二个参数。
还有几个函数：

- `packetRssi(SX1278_hw_t *hw)` 是用来读取包里面的 RSSI 值；
- `write_bytes(SX1278_hw_t *hw,const uint8_t *buffer, u64 size)` 是用来写入多个字节，从而通过 LoRa 发送；
- `available()` 是判断当前 `REG_RX_NB_BYTES` 是否为 0，也就是 RX 接收寄存器是否还有下一个字节。

特别写一下发送和接收：
```C
// 发送
uint8_t size = sprintf(buffer,"hello %d\n",counter);
beginPacket(&hw,false);     
write_bytes(&hw, (uint8_t *)buffer, size);
endPacket(&hw);
os_printf("Sending packet: ");
os_printf("%d\n",counter);
counter++;

--------------------------------------------------------

// 连续接收
void onReceive_LoRa(int packetSize){
    os_printf("received a packet");
    int i;
    for (i = 0; i < packetSize; ++i)
    {
        int read_data = read(&hw);
        os_printf("%s\n", (char)read_data);
    }

    // print RSSI of packet
    os_printf(" with RSSI");
    os_printf("%d\n", packetRssi(&hw));
}

onReceive(&hw, onReceive_LoRa);
receive_entry(&hw, 0);
```

其他的一些函数可以通过查看 Arduino 中 ESP8266 关于 SPI 的章节。


## LoRa 设置的参数
除了管脚之外，下面几个参数是比较重要的：
```C
typedef struct {
    ...
    long frequency;
    u8_t implicitHeaderMode;
    u8_t packetIndex;
} SX1278_hw_t;
```

