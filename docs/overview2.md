# 软件包封装： 如何发布兼容多种 JS 模块标准的软件包？



### **支持哪些模块规范**

IFFE -立即执行函数

AMD - require

CMD - SeaJS 

ESM - ES  标准的模块化方案 

UMD - 兼容 CJS 与 AMD、IFFE 规范



### **代码的压缩和混淆问题**

代码压缩是指去除代码中的空格、制表符、换行符等内容，将代码压缩至几行内容甚至一行，这样可以提高网站的加载速度。混淆是将代码转换成一种功能上等价，但是难以阅读和理解的形式。混淆的主要目的是增加反向工程的难度，同时也可以相对减少代码的体积，比如将变量名缩短就会减少代码的体积。



### **SourceMap 配置**

SourceMap 就是一个信息文件，里面存储了代码打包转换后的位置信息，实质是一个 json 描述文件，维护了打包前后的代码映射关系。



通常输出的模块不会提供 SourceMap，因为通过 sourcemap 就很容易还原原始代码。但是如果你想在浏览器中断点调试你的代码，或者希望在异常监控工具中定位出错位置，SourceMap 就非常有必要。





# **代码打包压缩**

## 配置 Vite 的打包方案

```
pnpm i terser@"5.4.0" -D
```



```ts
build: {
    rollupOptions,
    minify: 'terser', // terser | esbuild 压缩代码使用的库
    sourcemap: true, // 输出单独 source 文件
    // cssCodeSplit: true,
    lib: {
      entry: './src/entry.ts',
      name: 'SmartyUI', //  生成包的名字，在 iife/umd 包，同一页上的其他脚本可以访问它
      fileName: 'smarty-ui-vite', // 一个输出文件名的前缀，默认情况下会和模块类型配合组成最终的文件名
      formats: ['es', 'umd', 'iife']
    }
}
```



# monorepo方式管理组件库生态



## 传统 Mutirepo 方式的不足

遇到多个软件包的场景，使用多个 Repo 仓库的方式组织代码

多个项目间切换开发会非常不方便

npm link 方式把几个项目的本地目录链接起来。但是这种方法依然有弊端，比如在团队开发的时候，你必须随时同步所有的代码仓库。另外如果你的代码不希望公开到 Npm 上，你还需要建立私有的 Npm 仓库



## Monorepo 的优势

Monorepo 其实就是将多个项目 （pacakage 软件包）放到同一个仓库 （Repo） 中进行管理

- 可见性 （Visibility）: 每个开发者都可以方便地查看多个包的代码，方便修改跨 Package 的 Bug。比如开发 Admin 的时候发现UI 有问题，随手就可以修改。

- 更简单的包管理方式（Simpler dependency management）： 由于共享依赖简单，因此所有模块都托管在同一个存储库中，因此都不需要私有包管理器。

- 唯一依赖源（Single source of truth）： 每个依赖只有一个版本，可以防止版本冲突，没有依赖地狱。

- 原子提交： 方便大规模重构，开发者可以一次提交多个包（package）



### 方案选型

目前 JS 中常见的 Monorepo 大概有两种选择：Lerna、Pnpm workspace。



lernaJS 是由 Babel 团队编写的多包管理工具。因为 Babel 体系的规模庞大后有很多子包需要管理，放在多个仓库管理起来比较困难



Pnpm workspace 有性能优势



### 调整目录结构

![image-20230214171551607](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20230214171551607.png)



解决格式化校验时，文件不匹配的问题



```json
  "lint-staged": {
    "*.{js,ts,vue,tsx,jsx}": [
      "eslint --fix",
      "git add"
    ],
    "*.{json,md,yml,css}": [
      "prettier --write"
    ]
  },
```



仅使用 pnpm  进行模块管理

```json
"scripts": {
    "preinstall": "npx only-allow pnpm"
}
```





### 初始化工作空间

首先需要创建一个 pnpm-workspace.yaml，这个文件用于声明所有软件包全部存放在 packages 目录之中

```yaml
packages:
  # all packages in subdirs of packages/ and components/
  - 'packages/**'
```



### 依赖包安装位置选择

```shell
# 安装 workspace 中
pnpm i vite -w
```



如果只安装在子 package 里面，可以使用 -r 

```shell
# 子package安装
pnpm i vue -r --filter smarty-ui-vite

# 或者 直接在 docs-vite 目录下
pnpm i vue
```



