---
title: NPM-包升级
date: 2021-08-18 18:09:42
tags:
- NPM
- NodeJs
- Hexo
categories:
- NPM
---

```js
// 安装 npm-check-updates
npm install -g npm-check-updates
npm-check-updates // 运行检查
ncu -u // 更新package.json
npm install // 更新包到最新版
```

示例：

```ps1
PS E:\code\syxdevcode.github.io> npm install -g npm-check-updates
npm WARN deprecated request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142
npm WARN deprecated har-validator@5.1.5: this library is no longer supported
npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
C:\Users\ponytest\AppData\Roaming\npm\npm-check-updates -> C:\Users\ponytest\AppData\Roaming\npm\node_modules\npm-check-updates\bin\cli.js
C:\Users\ponytest\AppData\Roaming\npm\ncu -> C:\Users\ponytest\AppData\Roaming\npm\node_modules\npm-check-updates\bin\cli.js
+ npm-check-updates@11.8.3
added 311 packages from 167 contributors in 641.613s
[====================] 13/13 100%
PS E:\code\syxdevcode.github.io> npm-check-updates
 hexo                     ^5.3.0  →  ^5.4.0     n
 hexo-cli                 ^4.2.0  →  ^4.3.0
 hexo-deployer-git        ^2.1.0  →  ^3.0.0
 hexo-generator-index     ^1.0.0  →  ^2.0.0
 hexo-generator-searchdb  ^1.3.3  →  ^1.3.4
 hexo-renderer-marked     ^2.0.0  →  ^4.1.0
 hexo-renderer-stylus     ^1.1.0  →  ^2.0.1
 hexo-server              ^1.0.0  →  ^2.0.0

Run ncu -u to upgrade package.json
PS E:\code\syxdevcode.github.io> ncu -u
[====================] 13/13 100%

 hexo                     ^5.3.0  →  ^5.4.0
 hexo-cli                 ^4.2.0  →  ^4.3.0
 hexo-deployer-git        ^2.1.0  →  ^3.0.0
 hexo-generator-index     ^1.0.0  →  ^2.0.0
 hexo-generator-searchdb  ^1.3.3  →  ^1.3.4
 hexo-renderer-marked     ^2.0.0  →  ^4.1.0
 hexo-renderer-stylus     ^1.1.0  →  ^2.0.1
 hexo-server              ^1.0.0  →  ^2.0.0

Run npm install to install new versions.

PS E:\code\syxdevcode.github.io> npm install
npm WARN deprecated urix@0.1.0: Please see https://github.com/lydell/urix#deprecated
npm WARN deprecated resolve-url@0.2.1: https://github.com/lydell/resolve-url#deprecated
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@2.3.2 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@2.3.2: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: nice-napi@1.0.2 (node_modules\nice-napi):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for nice-napi@1.0.2: wanted {"os":"!win32","arch":"any"} (current: {"os":"win32","arch":"x64"})

added 3 packages from 3 contributors, removed 52 packages, updated 10 packages and audited 287 packages in 13.589s

22 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

参考：

[nodejs包高效升级插件npm-check-updates](https://www.cnblogs.com/xfcao/p/9436012.html)
