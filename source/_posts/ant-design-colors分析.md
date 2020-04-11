---
title: ant-design-colors分析
date: 2020-04-11 13:16:16
tags:
categories: 色彩
---

# ant-design-colors

这是 antd 在实践中总结的配色的解决方案，现在已经沉淀出一个独立的库，作用是：根据给定的主题色生成 9 种衍生色，共 10 种颜色。来学习一下他是这么做的。

# 分析

核心在 generate.ts 这个文件中,代码不长

## 变量定义

首先它定义了一些常量

```js
const hueStep = 2; // 色相阶梯
const saturationStep = 16; // 饱和度阶梯，浅色部分
const saturationStep2 = 5; // 饱和度阶梯，深色部分
const brightnessStep1 = 5; // 亮度阶梯，浅色部分
const brightnessStep2 = 15; // 亮度阶梯，深色部分
const lightColorCount = 5; // 浅色数量，主色上
const darkColorCount = 4; // 深色数量，主色下
// 暗色主题颜色映射关系表
const darkColorMap = [
  { index: 7, opacity: 0.15 },
  { index: 6, opacity: 0.25 },
  { index: 5, opacity: 0.3 },
  { index: 5, opacity: 0.45 },
  { index: 5, opacity: 0.65 },
  { index: 5, opacity: 0.85 },
  { index: 4, opacity: 0.9 },
  { index: 3, opacity: 0.95 },
  { index: 2, opacity: 0.97 },
  { index: 1, opacity: 0.98 },
];
```

## 函数定义

定义了三个函数

```ts
interface HsvObject {
  h: number;
  s: number;
  v: number;
}
function getHue(hsv: HsvObject, i: number, light?: boolean): number{
    ...
}
function getSaturation(hsv: HsvObject, i: number, light?: boolean): number{
    ...
}
function getValue(hsv: HsvObject, i: number, light?: boolean): number{
    ...
}
```

这三个函数分别用于获取 HSB，也就是上一篇说到的色相，饱和度，明度。
开头说过，会根据主题色衍生，共 10 种颜色。最浅的设为编号 1，最深的设为编号 10，主题色为编号 6.参数 i 则代表编号

<!--- more --->

## 色相

```ts
function getHue(hsv: HsvObject, i: number, light?: boolean): number {
  let hue: number;
  // 根据色相不同，色相转向不同
  // 如果不理解“转向”可以了解一下HSB模型
  if (Math.round(hsv.h) >= 60 && Math.round(hsv.h) <= 240) {
    hue = light
      ? Math.round(hsv.h) - hueStep * i
      : Math.round(hsv.h) + hueStep * i;
  } else {
    hue = light
      ? Math.round(hsv.h) + hueStep * i
      : Math.round(hsv.h) - hueStep * i;
  }
  if (hue < 0) {
    hue += 360;
  } else if (hue >= 360) {
    hue -= 360;
  }
  return hue;
}
```

## 饱和度

```ts
function getSaturation(hsv: HsvObject, i: number, light?: boolean): number {
  // grey color don't change saturation

  if (hsv.h === 0 && hsv.s === 0) {
    // 这个条件就意味着，颜色在圆柱模型的中轴线上，最上边是白，最下面是黑
    return hsv.s;
  }
  let saturation: number;
  if (light) {
    saturation = Math.round(hsv.s * 100) - saturationStep * i;
  } else if (i === darkColorCount) {
    // 最深的颜色 + 16
    saturation = Math.round(hsv.s * 100) + saturationStep;
  } else {
    saturation = Math.round(hsv.s * 100) + saturationStep2 * i;
  }
  // 边界值修正
  if (saturation > 100) {
    saturation = 100;
  }
  // 第一格的 s 限制在 6-10 之间
  if (light && i === lightColorCount && saturation > 10) {
    saturation = 10;
  }
  if (saturation < 6) {
    saturation = 6;
  }
  return saturation;
}
```

## 明度

```ts
function getValue(hsv: HsvObject, i: number, light?: boolean): number {
  if (light) {
    return Math.round(hsv.v * 100) + brightnessStep1 * i;
  }
  return Math.round(hsv.v * 100) - brightnessStep2 * i;
}
```

## 合成

```ts
export default function generate(color: string, opts: Opts = {}): string[] {
  const patterns: Array<string> = []; // 存储颜色的数组

  const pColor = tinycolor(color); // tinycolor 是一个库，用于转换颜色的形式，及获取颜色属性
  // 对亮色处理
  for (let i = lightColorCount; i > 0; i -= 1) {
    const hsv = pColor.toHsv();
    const colorString: string = tinycolor({
      h: getHue(hsv, i, true),
      s: getSaturation(hsv, i, true),
      v: getValue(hsv, i, true),
    }).toHexString();
    patterns.push(colorString);
  }
  // push主题色
  patterns.push(pColor.toHexString());
  // 对暗色处理
  for (let i = 1; i <= darkColorCount; i += 1) {
    const hsv = pColor.toHsv();
    const colorString: string = tinycolor({
      h: getHue(hsv, i),
      s: getSaturation(hsv, i),
      v: getValue(hsv, i),
    }).toHexString();
    patterns.push(colorString);
  }

  // dark theme patterns
  // 这个位置，处理暗色主题，注意是主题，不是颜色
  if (opts.theme === "dark") {
    return darkColorMap.map(({ index, opacity }) => {
      // 将当前颜色与背景色按照一定比例混合
      const darkColorString: string = tinycolor
        .mix(opts.backgroundColor || "#141414", patterns[index], opacity * 100)
        .toHexString();
      return darkColorString;
    });
  }
  return patterns;
}
```

关于 mix 方法 的实现

```js
tinycolor.mix = function (color1, color2, amount) {
  amount = amount === 0 ? 0 : amount || 50;

  var rgb1 = tinycolor(color1).toRgb();
  var rgb2 = tinycolor(color2).toRgb();

  var p = amount / 100;

  var rgba = {
    r: (rgb2.r - rgb1.r) * p + rgb1.r,
    g: (rgb2.g - rgb1.g) * p + rgb1.g,
    b: (rgb2.b - rgb1.b) * p + rgb1.b,
    a: (rgb2.a - rgb1.a) * p + rgb1.a,
  };

  return tinycolor(rgba);
};
```

到这里可以很清楚的知道，antd 究竟是按什么规则来处理主题色

1. 对于亮色，暗色分开处理
2. 对于暗色主题，特殊处理

<iframe
     src="https://codesandbox.io/embed/nameless-http-7r250?autoresize=1&codemirror=1&eslint=1&fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:600px; border:0; border-radius: 4px; overflow:hidden;"
     title="nameless-http-7r250"
     allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb"
     sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"
   ></iframe>
