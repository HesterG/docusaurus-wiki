# Typescript

## 笔记

[ts notes](https://docusaurus-wiki-three.vercel.app/frontend/ts-notes)

## 类型

[any vs unknown](https://www.jianshu.com/p/124787a71561)

tuple就是规定了长度的array

enum

```typescript
enum Color {
  Red = 1,
  Green = 2,
  Blue = 4,
}
```

enum转译成js后

```typescript
var Color;
(function (Color) {
    Color[Color["Red"] = 1] = "Red";
    Color[Color["Green"] = 2] = "Green";
    Color[Color["Blue"] = 4] = "Blue";
})(Color || (Color = {}));
```

相当于`Color["Red"] = 1`, 且`Color[1] = "Red"`

`console.log(Color)`输出

```js
{1: 'Red', 2: 'Green', 4: 'Blue', Red: 1, Green: 2, Blue: 4}
```

## 非空断言

[TypeScript 非空断言](https://cloud.tencent.com/developer/article/1610693)

## 编译

编译单个文件`tsc xxx.ts -w`

编译整个项目, 创建`tsconfig.json`

使用`webpack`编译

**note** 因为从下到上使用loader，所以`ts-loader`要放在最下面，先转成`js`，上面放`babel-loader`和他的`options`来编译

```js
use: [
  // 配置babel
  {
    // 指定加载器
    loader:"babel-loader",
    // 设置babel
    options: {
    // 设置预定义的环境
      presets:[
        [
          // 指定环境的插件
          "@babel/preset-env",
          // 配置信息
          {
            // 要兼容的目标浏览器
            targets:{
            "chrome":"58",
            "ie":"11"
          },
          // 指定corejs的版本
          "corejs":"3",
          // 使用corejs的方式 "usage" 表示按需加载
          "useBuiltIns":"usage"
      	}]
  		]
  	}
  },
  'ts-loader'
]
```

## 更多

[课件链接](https://drive.weixin.qq.com/s?k=AJEAIQdfAAoee5725k)

[typescript-features-to-avoid](https://www.executeprogram.com/blog/typescript-features-to-avoid)
