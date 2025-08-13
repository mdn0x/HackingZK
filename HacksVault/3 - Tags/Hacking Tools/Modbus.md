Modbus is an industrial protocol developed by Modicon, acquired by Schneider Electric. Modbus is widely used to connect industrial devices; protocol specification is available and royalty-free.

Modbus protocol is a master/slave protocol: the master reads and writes slaves' registers.

Modbus RTU is usually used via RS-485 (serial network): one master is present with one or more slaves. Each slave has an unique 8-bit address. 

Modbus data is used to read and write "registers" which are 16-bit long. The most common register is called "holding register" which is readable and writable; registry type "input register" is readable only. The registers "coil" and "discrete input" are 1-bit long: coils are readable and writable, discrete inputs are readable only.

Modbus register types:

- Discrete Input (Status Input): 1bit, RO
    
- Coil (Discrete Output): 1bit, R/W
    
- Input Register: 16bit, RO
    
- Holding Register: 16bit R/W
    

Most commonly Modbus function:

|   |   |
|---|---|
|Function Code|Register Type|
|1|Read Coil|
|2|Read Discrete Input|
|3|Read Holding Registers|
|4|Read Input Registers|
|5|Write Single Coil|
|6|Write Single Holding Register|
|15|Write Multiple Coils|
|16|Write Multiple Holding Registers|

Modbus TCP encapsulates Modbus RTU request and response data packets in a TCP packet transmitted over standard Ethernet networks. The TCP Modbus standard port is 502.

For more information see [Modbus 101 - Introduction to Modbus](https://www.csimn.com/CSI_pages/Modbus101.html).