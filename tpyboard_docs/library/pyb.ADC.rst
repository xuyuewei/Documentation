.. currentmodule:: pyb
.. _pyb.ADC:

class ADC -- 数模转换
=========================================

.. only:: port_tpyboard

    用法::

        import pyb
    
        adc = pyb.ADC(Pin('Y11'))       # create an analog object from a pin
        val = adc.read()                # read an analog value
    
        adc = pyb.ADCAll(resolution)    # create an ADCAll object,分辨率（resolution=12）
        val = adc.read_channel(channel) # read the given channel
        val = adc.read_core_temp()      # read MCU temperature
        val = adc.read_core_vbat()      # read MCU VBAT
        val = adc.read_core_vref()      # read MCU VREF

 
构造器
------------


.. only:: port_tpyboard

    .. class:: pyb.ADC(pin)

       创建并关联指定针脚的ADC对象。

方法
-------

.. only:: port_tpyboard

    .. method:: ADC.read()

       读取模拟引脚上的值并将其返回。返回值将介于0和4095之间。

    .. method:: ADC.read_timed(buf, timer)
    
       指定``timer``对象的频率，读取``buf``对象的模拟值。

       ``buf`` 有bytearray 或 array.array 用法。12位的ADC模拟量直接存放到``buf`` 
       如果``buf``只有8位元素（例如一个ByteArray）然后采样分辨率将降至8位。

       ``timer`` 定时器对象，每次定时器触发时都会读取一个示例。定时器必须已初始化和运行所需的采样频率。

       为了支持这个函数的以前的行为，``timer``也可以是一个整数，它指定的频率（赫兹）在。在这种情况下定时器（6）将自动配置为在给定的频率运行。
       
       使用定时器对象（首选方式）示例::

           adc = pyb.ADC(pyb.Pin.board.X19)    # create an ADC on pin X19
           tim = pyb.Timer(6, freq=10)         # create a timer running at 10Hz
           buf = bytearray(100)                # creat a buffer to store the samples
           adc.read_timed(buf, tim)            # sample 100 values, taking 10s

       使用整数的频率示例::

           adc = pyb.ADC(pyb.Pin.board.X19)    # create an ADC on pin X19
           buf = bytearray(100)                # create a buffer of 100 bytes
           adc.read_timed(buf, 10)             # read analog values into buf at 10Hz
                                               #   this will take 10 seconds to finish
           for val in buf:                     # loop over all values
               print(val)                      # print the value out

       此函数不分配任何内存。

AdCall对象
-----------------

.. only:: port_tpyboard

    实例化这个改变所有ADC引脚模拟输入。原单片机温度，VREF和VBAT的数据可以在ADC通道16访问，分别为17和18。适当的缩放将需要应用。芯片上的温度传感器的绝对精度差，仅适用于检测温度变化。

    .. method:: pyb.ADCAll(resolution)
    
       定义ADC的分辨率，可以设置为8/10/12。

    .. method:: adc.read()
    
       读取adc的值，返回值与adc分辨率有关，8位最大255，10位最大1023，12位最大4095。

    .. method:: adc.read_channel(channel)
    
       读取指定adc通道的值。

    .. method:: adc.read_core_temp()
    
       读取内部温度传感器。

    .. method:: adc.read_core_vbat()
    
       读取vbat电压 vback = adc.read_core_vbat() * 1.21 / adc.read_core_vref()。

    .. method:: adc.read_core_vref()
    
       读取vref电压（1.21V参考）3V3 = 3.3 * 1.21 / adc.read_core_vref()
。

    .. method:: adc.read_core_vbat()
    
       读取vbat电压 vback = adc.read_core_vbat() * 1.21 / adc.read_core_vref()。

    .. method:: adc.read_core_vbat()
    
       读取vbat电压 vback = adc.read_core_vbat() * 1.21 / adc.read_core_vref()。
       
    ``read_core_vbat()`` and ``read_core_vref()`` 方法读的后备电池电压和（1.21v参考）使用3.3V电源作为参考基准电压。假设ADCALL对象被实例化的``adc = pyb.ADCAll(12)``3.3V电源电压可以计算:
    
    ``v33 = 3.3 * 1.21 / adc.read_core_vref()``

    如果3.3V 是正确的话读取 ``adc.read_core_vbat()`` 将会是有效。如果电源电压降到3.3V以下，
    例如，在电池供电的放电电池系统中, 该稳压器将无法保持3.3V电源，导致读取不正确。 
    在这种情况下，读取的值仍然有效，使用以下内容:

    ``vback = adc.read_core_vbat() * 1.21 / adc.read_core_vref()``

    可以访问这些值不产生副作用 ``ADCAll``::
    
        import pyb
        import time
        import stm
        from pyb import Pin
        
        def adcread(chan):                              # 16 temp 17 vbat 18 vref
            assert chan >= 16 and chan <= 18, 'Invalid ADC channel'
            start = pyb.millis()
            timeout = 100
            stm.mem32[stm.RCC + stm.RCC_APB2ENR] |= 0x100 # enable ADC1 clock.0x4100
            stm.mem32[stm.ADC1 + stm.ADC_CR2] = 1       # Turn on ADC
            stm.mem32[stm.ADC1 + stm.ADC_CR1] = 0       # 12 bit
            if chan == 17:
                stm.mem32[stm.ADC1 + stm.ADC_SMPR1] = 0x200000 # 15 cycles
                stm.mem32[stm.ADC + 4] = 1 << 23
            elif chan == 18:
                stm.mem32[stm.ADC1 + stm.ADC_SMPR1] = 0x1000000
                stm.mem32[stm.ADC + 4] = 0xc00000
            else:
                stm.mem32[stm.ADC1 + stm.ADC_SMPR1] = 0x40000
                stm.mem32[stm.ADC + 4] = 1 << 23
            stm.mem32[stm.ADC1 + stm.ADC_SQR3] = chan
            stm.mem32[stm.ADC1 + stm.ADC_CR2] = 1 | (1 << 30) | (1 << 10) # start conversion
            while not stm.mem32[stm.ADC1 + stm.ADC_SR] & 2: # wait for EOC
                if pyb.elapsed_millis(start) > timeout:
                    raise OSError('ADC timout')
            data = stm.mem32[stm.ADC1 + stm.ADC_DR]     # clear down EOC
            stm.mem32[stm.ADC1 + stm.ADC_CR2] = 0       # Turn off ADC
            return data

        def v33():
            return 4096 * 1.21 / adcread(17)

        def vbat():
            return  1.21 * 2 * adcread(18) / adcread(17)  # 2:1 divider on Vbat channel

        def vref():
            return 3.3 * adcread(17) / 4096

        def temperature():
            return 25 + 400 * (3.3 * adcread(16) / 4096 - 0.76)

        pyb.delay(50)
        print('v33:',v33())
        print('vbat:',vbat())
        print('vref:',vref())
        print('temperature:',temperature())

