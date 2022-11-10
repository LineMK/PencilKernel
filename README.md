# Pencil_Kernel
#### 介绍
支持UEFI引导的Pencil-Kernel
#### 编译环境
##### 编译器
使用GCC编译<br />
Windows下,可以用`Mingw`编译`UEFI`程序,使用`x86_64-elf64-gcc`编译内核.<br />
Linux下编译UEFI需要用交叉编译器.<br />
```bash
sudo apt install gcc-mingw-w64-x86-64
```
##### EDK2
只需要`EDK2`中的`MedPkg/Include`下的文件.
#### 编译命令
##### UEFI
```bash
# $(CC)为编译器,$(INCLUDE_DIR)是uefi所用头文件目录,$(ENTRY_POINT)为UEFI入口地址(UefiMain)
# $(OUTPUT)是编译后的efi程序(BootX64.efi),$(SRC)为源代码
$(CC) -Wall -Wextra -I$(INCLUDE_DIR) -e $(ENTRY_POINT) -nostdinc -nostdlib \
        -finput-charset=UTF-8 -fexec-charset=UTF-16 \
        -fno-builtin -Wl,--subsystem,10 -m64 -mcmodel=large -o $(OUTPUT) $(SRC)
```
##### Kernel
```bash
# Kernel
# $(CC)为编译器,$(INCLUDE_DIR)是所用头文件的目录,
# $(OBJ)是编译后的内核程序,$(SRC)为源代码
# -c :只编译,不链接
# -fPIC :使生成的程序可以被加载到任意地址
$(CC) $(SRC) -c -o $(OBJ) -Wall -ffreestanding -fPIC -I$(UEFI_INCLUDE_DIR) \
        -nostdlib -nostdinc -mcmodel=large -fno-builtin -fno-strict-aliasing \
        -Wall -m64 -march=x86-64 -O0

# $(LD)为连接器, $(OBJ_FORMAT)为内核文件格式(elf64-x86-64)
# $(LDS)为连接脚本(kernel/kernel.lds)
$(LD) -b $(OBJ_FORMAT) -o $(OBJ).tmp $(OBJ) \
        -T $(LDS) -Map $(OBJ).map 

# 制作为纯二进制格式
# $(OUTPUT_FORMAT) 是最终输出文件的格式(binary)
$(OBJCOPY) -I $(OBJ_FORMAT) -S -j ".text" -j ".data" -j ".rodata" \
        -O $(OUTPUT_FORMAT) $(OBJ).tmp $(OUTPUT)
```
#### 运行(以U盘为例)
1.先将格式化为FAT32/16/12文件系统.(如果原来就是FAT32/16/12文件系统,则可以跳过此步骤)<br />
2.在根目录下创建EFI目录,然后在EFI文件夹中创建Boot目录<br />
3.将编译后的程序命名为`BootX64.efi`,复制进(2)中创建的Boot文件夹中<br />
4.从U盘启动即可.<br />