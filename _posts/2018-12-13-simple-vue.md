---
title: '用 webpack 单独编译 vue 组件'
layout: post
subtitle: ''
tags:
  - vue
category: vue
---
最近由于需要做一些管理用的界面，所以在研究 vue，这是一个很不错的一个前端框架，但问题是我早已有了不少页面，并不想全体重写开个新工程，但又想使用 vue 来写一些组件，就有了下面看起来有点奇怪的需求：

* 我希望能单独编译一个个 *.vue 文件，然后导入到某个页面中去。页面使用 script 引入 vue 的方式，而不使用 vue_cli 之类的来编译整个工程。

参考了 webpack 的文档和 [编译vue单文件组件](https://github.com/yangjunlong/compile-vue-demo)  等文章以后，初步实现了目标。

创建一个工程，用 npm init 初始化生成 package.json 文件，然后配置 webpack。

webpack.config.js 用的 vue-loader 的官方配置：

    // webpack.config.js
    const path = require('path')
    const VueLoaderPlugin = require('vue-loader/lib/plugin')

    module.exports = {
      mode: 'development',
      resolve: {
        // 自动补全的扩展名
        extensions: ['.js', '.vue', '.json', '.ts']
      },
      module: {
        rules: [
          {
            test: /\.vue$/,
            loader: 'vue-loader'
          },
          // 它会应用到普通的 `.js` 文件
          // 以及 `.vue` 文件中的 `<script>` 块
          {
            test: /\.js$/,
            loader: 'babel-loader'
          },
          // 它会应用到普通的 `.css` 文件
          // 以及 `.vue` 文件中的 `<style>` 块
          {
            test: /\.css$/,
            use: [
              'vue-style-loader',
              'css-loader'
            ]
          }
        ]
      },
      plugins: [
        // 请确保引入这个插件来施展魔法
        new VueLoaderPlugin()
      ]
    }

然后在这个目录下，执行 webpack 命令就可以单独编译某个 vue 文件：

    webpack mod1.vue -p --output-filename mod1.vue.js --output-library mod1

页面上使用下面的语法来导入相关的组件：

    <script src="https://unpkg.com/vue"></script>
    <script src="../dist/mod1.vue.js"></script>
    <script src="../dist/mod2.vue.js"></script>

    <script lang="javascript">
      let vue=new Vue({
        el: "#app",
        components: {
          'mod1': mod1.default,
          'mod2': mod2.default
        }
      });
    </script>

为了方便起见，我写了个 build.js 枚举目录并执行 webpack 命令：

    // build.js
    var fs = require('fs');
    var path = require('path');
    const { exec } = require('child_process');

    //解析需要遍历的文件夹，我这以E盘根目录为例
    var filePath = path.resolve('.');
    fileDisplay(filePath);

    function fileDisplay(filePath){
    //根据文件路径读取文件，返回文件列表
    fs.readdir(filePath,function(err,files){
        if(err){
            console.warn(err)
        }else{
            //遍历读取到的文件列表
            files.forEach(function(filename){
                //获取当前文件的绝对路径
                var filedir = path.join(filePath,filename);
                //根据文件路径获取文件信息，返回一个fs.Stats对象
                fs.stat(filedir,function(eror,stats){
                    if(eror){
                        console.warn('获取文件stats失败');
                    }else{
                        var isFile = stats.isFile();//是文件
                        var isDir = stats.isDirectory();//是文件夹
                        if(isFile){
                            var ext=path.extname(filename);
                            if(ext==".vue"){
                                console.info("Complate file: " + filedir);
                                var basename=path.basename(filename, ext);
                                
                                let cmd='webpack ' + filedir + " -p --output-filename " + filename + ".js --output-library " + basename;
                                console.info(cmd);
                                exec(cmd, { }, (err, stdout, stderr) => {
                                    if(err) {
                                        console.log(err);
                                        return;
                                    }
                                    console.log(`stdout: ${stdout}`);
                                });
                            }
                        }
                        if(isDir){
                            fileDisplay(filedir);//递归，如果是文件夹，就继续遍历该文件夹下面的文件
                        }
                    }
                })
            });
        }
    });
    }

所以修改 package.json 文件，把 build.js 配进去，就可以通过 npm run build 编译所有组件了。

    "scripts": {
      "build": "node build.js"
    },

