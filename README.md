# WEBPACK

影片：https://www.youtube.com/watch?v=uP6KTupfyIw&t=2384s

### 安裝配置 Webpack，建立前端專案！

https://webpack.js.org/concepts/

1. 建立 `src` 作為專案的入口，並建立 index.js
   若沒有特別設定 `webpack.config.js` 的話，會自動使用 src 作為入口。

2. 設定 build 編譯指令

```javascript=
// package.json

"scripts": {
  "build": "webpack"
}
```

3. 安裝 webpack cli（執行 `npm run build` 時會被要求安裝 webpack cli）
   ![](https://i.imgur.com/AoskdOX.png)

4. 執行 `npm run build`，產生 dist 資料夾與 `dist/main.js`
   此處的 main.js 是編譯好的 `src/index.js`

### 配置 webpack.config.js 檔案

https://webpack.js.org/concepts/configuration/

- Entry
- Output
- Loaders 辨識、讀取閱讀檔案
- Plugins 讀檔案之外的事，用來加強設定
- Mode 開發（未壓縮）、上線的模式

1. 設定 entry 與 output

```javascript=
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'), // 在 output 這個資料夾的路徑
    filename: 'main.bundle.js', // 建立 main.bundle.js 這支檔案
  },
};
```

2. 設定 dev 編譯指令

```javascript=
  "scripts": {
    "build": "webpack",
    "dev": "webpack serve"
  },
```

3. 安裝 `webpack-dev-server`
   ![](https://i.imgur.com/dboDoJi.png)

4. 由於 `webpack` 版本問題，要加入 mode 和 devServer 的設定才能順利運行。

- https://stackoverflow.com/questions/69054930/cannot-get-localhost-8080-not-working-with-webpack-dev-server
- https://webpack.js.org/configuration/dev-server/ （影片中是使用 v4，目前官網是 v5 了，因此以 v5 為開發版本）

```javascript=
module.exports = {
  mode: 'development',
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist/'), // 在 output 這個資料夾的路徑
    filename: 'main.bundle.js', // 建立 main.bundle.js 這支檔案
  },
  devServer: {
    port: 8888,
    static: './dist', // 設定以此為根目錄，在 webpack v4 中對應的是 contentBase
  },
};
```

5. 在 dist 中建立 index.html 檔案，並且引入 bundle 好的 main.bundle.js 檔案即可執行

### css loader

```javascript=
// src/index.js
import './index.css'; // 此處的 css webpack 無法讀取，因為預設只能讀 JS
console.log('Helloooo');
```

webpack 無法看懂我們 CSS 的寫法，因此需要安裝 loader 來辨識 CSS 的內容。
![](https://i.imgur.com/oeUm3Cw.png)

https://webpack.js.org/loaders/css-loader/#root

1. 安裝 css-loader 以及 style-loader
2. 設定 module

```javascript=
module.exports = {
  // 略
  module: {
    rules: [
      {
        test: /\.css$/i, // 凡是 css 相關的檔案
        use: ['style-loader', 'css-loader'], // 都要使用 這些 loader 來進行解析
      },
    ],
  },
};
```

### hash 名稱、 HtmlWebpackPlugin 、 MiniCssExtractPlugin、source map...

#### hash 名稱

目前產生的 html 是自己寫死的，如果要使用 webpack 每次 bundle 都能自動生成 hash 亂數檔名的 js（`e.g. main.0a91deaed7cb88ba1fc2.bundle.js`），每次檔名都不同的話可以保證用戶每次載入都是全新的頁面，避免 catch 的問題(e.g. 上版修 bug)。

```javascript=
module.exports = {
  // 略
  output: {
    path: path.resolve(__dirname, 'dist'), // 在 output 這個資料夾的路徑
    filename: 'main.[hash].bundle.js', // hash
  },
};
```

#### HtmlWebpackPlugin

https://webpack.js.org/plugins/html-webpack-plugin/#root
之前使用而如果希望 webpack 自動生成 html 並載入上面 hash 的 JS 話 ，則要另外使用 HtmlWebpackPlugin 這個 plugin。

然而此時產生的 html 是空白的，因此需要載入一個 template 來設定 html 的 body 部分。

```javascript=
const HtmlWebpackPlugin = require('html-webpack-plugin');

plugins: [
  new HtmlWebpackPlugin({
    template: './base.html', // 依據 base.html 來產生內容
  }),
],
```

#### MiniCssExtractPlugin

https://webpack.js.org/plugins/mini-css-extract-plugin/#root
目前編譯好的 css 是寫在 html 中的，如果想要將 css 檔案抽離出來並在 dist 中建立獨立的 css 檔案的話，則要使用 MiniCssExtractPlugin。
![](https://i.imgur.com/bB4a5Zf.png)

```javascript=
// 引入
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  // 略
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [MiniCssExtractPlugin.loader, 'css-loader'], // 寫在這
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './base.html',
    }),
    new MiniCssExtractPlugin({
      filename: 'main.[hash].css',
    }),, // 加入 plugin 並生成 hash 檔名的 css
  ],
};
```

分離產生出的 main.css
![](https://i.imgur.com/Ncj0gI5.png)

#### source map

幫我們使用 devtool 時可以追溯到編譯前的檔案以方便 debug。
![](https://i.imgur.com/CmFSrhO.png)

```javascript=
devtool: 'source-map',
```

![](https://i.imgur.com/ECDhGh2.png)

### Babel 介紹與使用

將不同版本的語法編譯成通用瀏覽器能夠支援的語法。
照著步驟安裝設定即可：https://babeljs.io/setup#installation

1. 進官網選擇 webpack
2. 安裝 babel
3. 設定 webpack.config.js
4. 安裝 preset 編譯較新的語法
5. 建立 babel.config.js 來加入 presets（也可以直接寫在 webpack.config.json 中）

```javascript=
module: {
    rules: [
      {
        test: /\.m?js$/,
        // babel 不要編譯 node modules 的檔案
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          /* 這裡的設定可以直接獨立寫在 babel.config.js
          options: {
            presets: ['@babel/preset-env'],
          },
          */
        },
      },
    ],
  },
```

### Asset Modules 介紹

https://webpack.js.org/guides/asset-modules/#root

```javascript=
{
  test: /\.png/,
  type: 'asset/resource'
}
```

### Q&A

- 自動清除 dist 資料夾的 plugin
  https://webpack.js.org/guides/output-management/#cleaning-up-the-dist-folder
- 使用 env 切換 mode 模式
- 和 js 編譯有關的都去找 babel，例如 jsx
# webpack-practice
