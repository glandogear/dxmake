---
title: "在FPGA仿真时从文件中读取激励信号"
date: 2020-09-03T17:28:28+08:00
draft: false
tags: [FPGA,verilog,testbench,文件读写]
categories: [FPGA]
---

在Verilog中使用`$readmemh()`命令从文件中读取数据，并作为FPGA仿真时的激励输入。

<!--more-->

`$readmemh()`读取的文件是以文本形式存储的数据，文件中可以包含十六进制或者二进制字符、空白符（包括空格，换行，制表符等）以及注释，注释支持`//`和`/*……*/`两种格式，文件中还可以用@符号指定某些数据的地址。如下是一个文件的例子：

```plain
// data.mem

1 00 00000001   // spi BASE+0 clock divider setting
0 00 00000000
1 01 A0000100   // 10 100000 000000000000000000000010
1 02 FFFF5555   // 1111 1111 1111 1111 0101 0101 0101 0101
```

这个文件中包含三个数据，位宽分别为1、8、32，还有许多注释。

在testbench文件中，我们定义一段内存`memory`用来从文件中读取数据，它的位宽为最宽的数据的宽度，存储单元数为总的数据量，例子中为四行乘每行三个数据。从文件中得到数据之后，使用for循环，并在循环中加入长度为一个时钟周期的延迟，之后将memory中的数据按赋值给相应的变量。这样一个简单的激励信号设置就完成了。

```verilog
// testbench.v
// ==== stimulating signals from FPGA side ====================================
localparam DATA_SEGS = 3;
integer i;
reg [32:0] memory [0:4*DATA_SEGS-1];
initial begin
    $readmemh("data.mem",memory);
	i = 0;
    for (i=0;i<4;i=i+1) begin
		#FULL_CYCLE;
		var_a = memory[i*DATA_SEGS+0][0:0];
		var_b = memory[i*DATA_SEGS+1][7:0];
		var_c = memory[i*DATA_SEGS+2][31:0];
	end
end
```

## 其他

- `$readmemh()`还有两个可选参数分别用来指定将数据存入memory中的哪一段地址。
- 在mem文件中可以使用`@`指定数据在memory中的地址，例如`@5 FF`将把0xFF存入`memory[5]`中。
- 另有一个类似的命令`$readmemb()`，它会以二进制格式解析文件。
- 其他的都文件命令有`$fscanf()`。

