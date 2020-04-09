---
title: "频谱中幅值的意义"
date: 2020-04-09T01:01:55+08:00
draft: false
tags: ["数字信号处理","FFT","频谱","Matlab"]
categories: ["信号处理"]
mathjax: true
lastmod: 2020-04-09 12:05:41
---

在使用FFT函数运算时，经常能够看到信号频谱的幅值非常大，而信号在时域中并没有这种数量级的参数，令人疑惑。
本文将结合理论和Matlab实验，说明各种频谱中幅值的实际意义，包含了幅度谱、功率谱在单、双边表示的情况。

本文关注的是一段确定信号的频谱，通常情况下随机信号才会有功率谱（密度）的分析，这里借用了类似的概念，可以将这段确定信号理解为随机信号的一个实例。

对于实信号来说，正负频谱为共轭关系，所以可以用单边谱来进行分析。而复信号的正负频谱没有关系，只能用双边谱表示。

## 测试信号
为同时说明单、双边谱，我们使用一个实正弦信号作为测试信号。
首先生成测试信号，设置采样率为100Hz，采样点数为1000。
设置正弦信号幅度为1，频率为20Hz。

$$
\begin{aligned}
signal & = sin(2\pi * 20 * t) \\\ 
& = \frac{1}{2j}(e^{j*2\pi *20 *t}-e^{-j *2\pi *20 *t})
\end{aligned}
$$

```
Fs = 100;
Ts = 1./Fs;
pointNum = 1000;
t = [0:pointNum-1] .* Ts;
signal = sin(2*pi*20*t);
```

## 累积幅度谱

`fftAmplitudeCumulation = abs(fftshift(fft(signal)))`

对信号求fft，并进行循环移位和频率轴映射，可以得到最简单的一个频谱，我称之为累积幅度谱。
其中每个点的值代表相应频率信号分量的幅度在采样点上的累积，或者说幅度对采样点的积分。
如果一个复指数信号幅度为1,持续了20个采样点，那么取值就是20；
或者信号幅度一直在改变，那么将每个采样点处的幅度累加，就是频谱中的幅值。

在例子中存在两个幅度都是500的峰值，
在测试信号中幅度恒定为1的正弦信号是由两个幅度恒为0.5的正负频率的复指数信号相加得到的，
而采样点数为1000。所以在累积幅度谱中正负20Hz的值应为1000×0.5=500，如图。
![双边累积幅度谱](/amplitude-of-spectrum/01-1.amplitude_cumulation_double.svg "双边累积幅度谱")

同样，单边累积幅度谱20Hz处的幅值就是20Hz正弦信号的幅度1乘以采样点数1000。
或者说双边谱正频率的两倍，而零频处不变，如图。
![单边累积幅度谱](/amplitude-of-spectrum/01-2.amplitude_cumulation_single.svg "单边累积幅度谱")

## 平均幅度谱

`fftAmplitudeAverage = fftAmplitudeCumulation ./ pointNum`

前面的累积幅度谱是与采样点数相关的，如果一个信号是功率信号，而我想要知道在这段时间中它平均的幅度，
就可以使用平均幅度谱，表示每个频率的基信号在这段时间的幅度。
简单地将累积幅度谱的幅值除以总采样点数，就可以得到平均幅度谱。

测试信号的双边和单边平均幅度谱如下，可以看到恒定的正弦信号的幅值为1，两个复指数信号幅值为0.5。
![双边平均幅度谱](/amplitude-of-spectrum/02-1.amplitude_average_double.svg "双边平均幅度谱")
![单边平均幅度谱](/amplitude-of-spectrum/02-2.amplitude_average_single.svg "单边平均幅度谱")

## 平均功率谱 - 线性

`fftPowerAverageLinear = fftAmplitudeAverage .^ 2`

由于FFT不能表示信号的变化过程，只能表示一段时间内的平均情况，因此在这里功率和能量是没有区别的。
正弦信号的功率为0.5Watt。
![双边平均功率谱-线性](/amplitude-of-spectrum/03-1.power_average_linear_double.svg "双边平均功率谱-线性")
![单边平均功率谱-线性](/amplitude-of-spectrum/03-2.power_average_linear_single.svg "单边平均功率谱-线性")

## 平均功率谱 - 对数

`fftPowerAverageLogarithm = 10*log(1000*fftPowerAverageLinear)/log(10)`

以dBm表示的信号功率。
![双边平均功率谱-对数](/amplitude-of-spectrum/04-1.power_average_logarithm_double.svg "双边平均功率谱-对数")
![单边平均功率谱-对数](/amplitude-of-spectrum/04-2.power_average_logarithm_single.svg "单边平均功率谱-对数")


## 其他

- 将累积幅度谱乘以采样间隔Ts，可以得到信号幅度在时间上的累积值，单位（V*m）。
- 对整个线性平均功率谱求和可以得到信号的总功率。


