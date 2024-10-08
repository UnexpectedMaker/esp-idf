USB 主机维护者注释（架构）
===========================

:link_to_translation:`en:[English]`

主机协议栈大致分为多个抽象层，每个层代表不同的 USB 概念和不同级别的 USB 主机操作。例如，较高的层可能呈现为设备和应用数据传输的抽象，而较低的层则为端点和 USB 传输的抽象。

层描述
------

从最低层（离用户最远的层）到最高层（离用户最近的层），主机协议栈的层描述如下表所示：

.. list-table:: 主机协议栈的各层功能简述
    :widths: 20 10 70
    :header-rows: 1

    * - 层
      - 文件
      - 描述
    * - 主机控制器 (DWC_OTG)
      - N/A
      - 此层代表 {IDF_TARGET_NAME} 的 USB 控制器硬件，所提供的 API 是控制器的寄存器接口。
    * - LL
      - ``usbh_ll.h``
      - LL（低级）层根据 ESP-IDF 的 :doc:`硬件抽象 API 指南 <../../../api-guides/hardware-abstraction>` 抽象了 USB 控制器的基本寄存器访问。换句话说，此层提供的 API 能访问控制寄存器，也可以格式化/解析控制器的 DMA 描述符。
    * - HAL
      - ``usbh_hal.h``， ``usbh_hal.c``
      - HAL（硬件抽象层）根据 ESP-IDF 的 :doc:`硬件抽象 API 指南 <../../../api-guides/hardware-abstraction>`，将 USB 控制器的操作步骤抽象成函数。此层还将控制器的主机端口和主机通道抽象化，并提供操作它们的 API。
    * - HCD
      - ``hcd.h``， ``hcd.c``
      - HCD（主机控制器驱动程序）是一个与硬件无关的 API，适用于任何 USB 控制器（理论上，此 API 可以与任何 USB 控制器实现一同使用）。此层还抽象了根端口（根集线器）和 USB 管道。
    * - USBH 和集线器驱动程序
      - ``usbh.h``， ``usbh.c``
      - USBH（USB 主机驱动程序）层等效于 USB2.0 规范第 10 章中描述的 USBD 层。USBH 提供 USB 设备的抽象，在内部管理已连接设备的列表（即，设备池），还仲裁客户端之间的设备共享（即，跟踪正在使用的端点并提供一个共享端点 0）。
    * - 集线器驱动程序
      - ``hub.h``， ``hub.c``
      - 集线器驱动程序层是 USBH 的特殊客户端，负责处理设备的连接/断开，并通知 USBH 此类事件。对于设备的连接，集线器驱动程序还会处理枚举过程。
    * - USB 主机库
      - ``usb_host.h``， ``usb_host.c``
      - USB 主机库层是主机协议栈的最低公共 API 层，并呈现 USB 主机客户端的概念。客户端的抽象允许多个 class 驱动程序同时存在（其中每个类大致映射到一个单独的客户端），这也是一种分工机制（其中每个客户端负责各自的处理以及事件处理）。
    * - 主机 Class 驱动程序
      - 请参阅 `ESP-USB 存储库 <https://github.com/espressif/esp-usb>`_ 或是 ESP-IDF 中的 USB 主机示例 :example:`peripherals/usb/host`。
      - 主机 Class 驱动程序能实现特定设备类（例如，CDC、MSC、HID）的主机端。每个 class 驱动程序具有特定的公开 API 接口。

层依赖关系
----------

主机协议栈大致遵循自顶向下的层次结构，具有层间依赖关系。假设存在层 A（最高层）、B 、 C（最低层），则主机协议栈具有以下层间依赖规则：

- 特定层可以使用其直接下层的 API（层 A 可以使用层 B 的 API）。
- 特定层可以使用其间接下层的 API（层 A 可以使用层 C 的 API），即可以隔层调用。
- 特定层不能使用其任何上层的 API（层 C 无法使用层 A/B 的 API）。

.. note::

  允许跳过层次以避免在多个层次之间重复相同的抽象。例如，管道的抽象都位于 HCD 层，但被上面的多个层使用。

.. todo::

  添加图示以说明不同层之间的 API 依赖关系
