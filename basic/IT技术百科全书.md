### 阻塞与非阻塞判断依据？

当存在磁盘或数据库相关的操作时候，是阻塞的，如果仅仅是对流的操作就是非阻塞的。切勿将流操作默认为阻塞的操作，只要不涉及磁盘相关操作，还是非阻塞的。

### IaaS、PaaS、SaaS的区别？

- **IaaS：**基础设施服务，Infrastructure-as-a-service

- **PaaS**：平台服务，Platform-as-a-service
- **SaaS**：软件服务，Software-as-a-service

看到这个其实已经很清楚了，SaaS是作为这三者中集成度是最高的。IaaS提供一些基础的工具或者依赖服务等基础资源，PaaS提供平台化的服务，像一些云服务提供商提供的一些大数据计算服务等，SaaS是开箱即用，不需要懂任何技术，普通人就可以安装使用。

### Bit/Byte/mbps

**Bit：**位

**Byte：**字节

1byte=8bit

一个英文字母=1byte

一个中文字=2byte

1 KB = 1024 Bytes

mbps=mega bits per second(兆位/秒)是速率单位，所以正确的说法应该是说usb2.0的传输速度是480兆位/秒,即480mbps。 

mb=mega bytes(兆比、兆字节)是量单位，1mb/s（兆字节/秒）=8mbps（兆位/秒）。