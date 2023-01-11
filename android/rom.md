-   BootLoader
    -   引导手机开机的一个自动加载器、通过fastboot协议可以与电脑通信
    -   未解锁的BootLoader会拒绝进行刷机操作以及拒绝非官方rom的启动
        -   厂商出于安全会对BootLoader加锁

-   fastboot

    -   BootLoader所支持的一种协议，进入后可以通过执行fastboot指令引导系统

    -   在高版本Android中被拆分成两部分

        -   bootloader：解锁、刷入recovery等
        -   fastbootd：负责大规模传输数据的功能，如刷gsi

    -   刷入recovery镜像

        ```sh
        fastboot flash recovery ***.img
        ```


-   使用adb进入

    -   进入bootloader

        ```sh
        adb reboot bootloader
        ```

    -   进入fastbootd

        ```sh
        adb reboot fastboot
        ```
        
    -   进入recovery
    
        ```sh
        adb reboot recovery
        ```



官改/gsi

