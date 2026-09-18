---
created_at: 2026-06-22
updated_at: 2026-09-18
tags:
    - 嵌入式
    - STM32
archived: false
---

[STM32](./STM32.md)

# 开发环境配置

1. 软件包下载

```bash
sudo pacman -S cmake  //提供构建工具
sudo pacman -S stlink //提供下载和调试功能
paru -S visual-studio-code-bin //下载vscode,提供代码编写,调试,提供交叉编译工具
paru -S stm32cubemx //图形化配置引脚功能
paru -S archlinux-java-run //为stm32cubemx提供运行环境
```

使用任意方式下载serial port assistant, 获得串口调试功能, 执行命令`sudo usermod -a -G uucp $USER`获得串口访问权限, 您可能需要重启以使该设置生效

2. vscode插件下载  
   搜索STM32CubeIDE for Visual Studio Code, 下载发布者为STMicroelectronics的插件包

## 相关笔记

- 下一篇:[STM32F103C8T6芯片引脚分布](STM32F103C8T6芯片引脚分布.md)  
- 返回索引:[STM32](./STM32.md)  
