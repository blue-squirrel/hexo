---
title: lerna实现Monorepo
---

lerna的方式实现Monorepo
<!-- more -->

### 1、全局安装lerna

```
npm i --g lerna
```

### 2、使用`git init`初始化一个项目仓库

```
git init lerna-learning && cd lerna-learning
```

### 3、执行lerna初始化

```
lerna init -i  // -i 表示各包 独立的版本控制
```

### 4、创建新的package

```
lerna create main
lerna create com-a
lerna create util
```

![image-20220814161658610](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220814161658610.png)

### 5、将其余包添加到main中，一并导出

```
lerna add xjh-test-lerna-com-a --scope=xjh-test-main
lerna add xjh-test--lerna-util --scope=xjh-test-main
```

```
// main/lib/index.js

var util = require('xjh-test-lerna-util');
var comA = require('xjh-test-lerna-com-a');

export default {
    util,
    comA
};
```



在根目录配置打包命令

```
// package.json

"scripts": {
   "build": "lerna run --stream --sort build",
  }
```

配置rollup.config.js

```
// rollup.config.js
import postcss from 'rollup-plugin-postcss';
import { nodeResolve } from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';

const fs = require('fs');
const pkJson = JSON.parse(fs.readFileSync('package.json', 'utf8'));

export default {
    input: 'lib',
    output: {
      file: 'dist/index.js',
      format: 'es',
      name: 'index'
      // name: pkJson._globalName
    },
    plugins: [
        nodeResolve(),
        postcss(),
        commonjs(),
    ]
}
```

npm run build打包各包文件

lerna publish发布各包文件即可



单独引入包或者按需引入

```
import util from 'xjh-test-lerna-util';

import {util} from 'xjh-test-lerna-main';
```
