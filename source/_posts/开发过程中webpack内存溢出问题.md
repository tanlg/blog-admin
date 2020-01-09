---
title: 开发过程中webpack内存溢出问题
date: 2019-10-24 18:55:21
tags: moka
---


mage项目比较大，启动起来保存几下就崩溃了，如图：
![mage崩溃图](/image/20191024185854.jpg)
google查阅相关资料，整个插件分析一下 webpack-bundle-analyzer
```javascript
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

new BundleAnalyzerPlugin()
```
启动后发现，嚯，这么大。怪不得妹纸崩溃了。
![mage崩溃原因图](/image/20191024190619.jpg)

分析一下原因：
1. 项目本身太大了，开发过程中把所有入口的项目都打包了（hr pc端、hr移动端、 hm pc端、hm移动端等等）。
2. 使用的其他仓库的东西没有做按需加载（比如 moka UI）
   ![开发环境打包的js每个都含有 1M+ 大小的moka UI](/image/20191024192752.jpg)
3. 三方库用的太多，挖掘机炒菜。（猜测很多库只用到了很小到一部分功能，对应的功能完全可以手动实现？）

展望一下未来的优化思路以及解决方法：
1. 爷有钱，加内存！
![炫富](/image/20191024193632.jpg)
4096不行，整上40960
很捞的方法，也是我正在使用的方法，优点是省事，缺点是除了省事没别的了，作为一位程序员还是要有不断优化项目的觉悟。
2. 曾经用过的一个方法，webpack按照路由编译，
```javascript
entry: {
  app: [
    require.resolve('./polyfills'),
    'react-hot-loader/patch',
    config.startRoutes ? paths.appDevIndexJs : paths.appIndexJs
  ]
},
```
具体思路是两个参数(根目录整个config.js配置一下)，一个用来控制是否按路由启动， 第二个参数是传入的路由文件（组件），这样就把需要调试的地方渲染出来了，很快。（缺点是对路由文件的管理比较严格，改一改应该可以应用到mage项目里）
```javascript
module.exports = {
  port: 3000,
  proxy: {
    '/p': 'http://test.mokahr.com'
  },
  env: {},
  publicPath: '/',
  eslint: true,
  bundleAnalyzer: false,
  autoOpenUrl: port => `http://test.mokahr.com:${port}`,
  startRoutes: false,
  routePath: 'xxxxxx'
  useTs: false
}

```
3. 把不同的入口拆成不同的仓库，这样就能单独维护某一个端了。
4. 新的suger design 已经做按需加载了，很棒，应该就跟antd design + babel-plugin-import 的效果一样 引入对应的文件。
5. 网上找了个方法，还没来得及看，先放这儿了
使用webbpack-dev-serve钩子进行单独编译：在webpack-dev-serve中，有一个钩子before，在访问页面的时候我们能够拿到页面信息的路径，下面是实现
```javascript
// vue.config.js
const compiledPages = [];
before(app) {
      app.get('*.html', (req, res, next) => {
        const result = req.url.match(/[^/]+?(?=\.)/);
        const pageName = result && result[0];
        const pagesName = Object.keys(multiPageConfig);

        if (pageName) {
          if (pagesName.includes(pageName)) {
            if (!compiledPages.includes(pageName)) {
              const page = multiPageConfig[pageName];
              fs.writeFileSync(`dev-entries/${pageName}.js`, `import '../${page.tempEntry}'; // eslint-disable-line`);
              compiledPages.push(pageName);
            }
          } else {
            // 没这个入口
            res.writeHead(200, { 'content-type': 'text/html; charset=utf-8' });
            res.end('<p style="font-size: 50px;">不存在的入口</p>');
          }
        }
        next();
      });
    },

```
```javascript
const fs = require('fs');
const util = require('util');

const outputFile = util.promisify(fs.writeFile);
async function main() {
  const tasks = [];
  if (!fs.existsSync('dev-entries')) {
    fs.mkdirSync('dev-entries');
  }
  Object.keys(pages).forEach((key) => {
    const entry = `dev-entries/${key}.js`;
    pages[key].tempEntry = pages[key].entry; // 暂存真正的入口文件地址
    pages[key].entry = entry;
    tasks.push(outputFile(entry, ''));
  });
  await Promise.all(tasks);
}

if (process.env.NODE_ENV === 'development') {
  main();
}

module.exports = pages;

```



 

