在正式开始项目前, 先给出详细的流程和解决步骤。
实际开发中，就只记录一些值得记录的了。

## 🍕 项目搭建

环境:

- win10
- node 16.16.0
- npm 8.17.0

vscode 插件:

- dbaeumer.vscode-eslint
- vue.volar

### 创建项目

找好存放项目文件夹的位置, 执行初始化命令

```bash
npm create vite@latest
# 依次输入 项目名 - vue - typescript
```

### 初始化项目

先修改 `package.json` 的版本号

```js
// 之前
"version": "0.0.0",
// 之后
"version": "0.0.1",
```

然后安装依赖

```bash
npm i
```

安装完成后运行 vite, 页面成功载入

修剪改相关文件:

- `README.MD`
- `.gitignore`

**注**: 若 `vite.config.ts` 文件提示找不到模块, 尝试 `reload window` 重启一下

初始化 git 仓库, 然后 publish

### 初始化 ESLint

初始化

```bash
npm init @eslint/config
# 初始化, 依次选择 problems - esm - vue - ts - js
```

将自己积累的 `rules` 拷贝到 `.eslintrc.cjs` 配置文件中,
然后在 `.eslintrc.cjs` 文件调出 command palette ,
然后执行 `ESLint: fix all auto-fixable Problems` 命令。
（其他文件同理）

解决 eslint 在 `App.vue` 提示 `Parsing error: '>' expected.` 错误

```cjs
/* .eslintrc.cjs */
module.exports = {
  // 之前:
  'parser': '@typescript-eslint/parser',
  // 之后:
  'parser': 'vue-eslint-parser',
}
```

解决 eslint 在 `HelloWorld.vue` 提示 `Parsing error: Unexpected token )`

```cjs
/* .eslintrc.cjs */
// 在 parserOptions 参数中添加 ts 解析:
module.exports = {
  'parserOptions': {
    'parser': '@typescript-eslint/parser',
  },
}
```

eslint 在 `.eslintrc.cjs` 中提示 `'module' is not defined.eslintno-undef`

```cjs
/* .eslintrc.cjs */
// 原因是前面初始化时, 忘记添加 node 环境了, 补上 node 环境即可:
module.exports = {
  'env': {
    'node': true,
  },
}
```

eslint 在 `vite-env.d.ts` 中提示　`Don't use `{}` as a type.` 和

```ts
/* vite-env-d.ts */
// 先在报错语句上面添加一行注释, 忽略此错误。可以通过快捷键 ctrl+. 选择快捷修复, 后续开发中的一些报错也可以使用这种方式解决
// eslint-disable-next-line @typescript-eslint/ban-types, @typescript-eslint/no-explicit-any
const component: DefineComponent<{}, {}, any>
```

最后, 创建一个 `.eslintignore` 文件, 然后将要忽略的文件或文件夹写入, 比如 `dist` 文件夹

### 引入 element-plus

采用 "按需自动导入" 的方式

先安装

```bash
npm install element-plus --save # 安装 element-plus
npm install -D unplugin-vue-components unplugin-auto-import # 按需自动导入
```

安装后自动生成了两个声明文件: `components.d.ts` 和 `auto-imports.d.ts`

修改 `vite.config.ts` 文件:

- 之前:

  ```ts
  import { defineConfig } from 'vite'
  import vue from '@vitejs/plugin-vue'

  // https://vitejs.dev/config/
  export default defineConfig({
    plugins: [vue()]
  })
  ```

- 之后:
  ```ts
  import { defineConfig } from 'vite'
  import vue from '@vitejs/plugin-vue'
  import AutoImport from 'unplugin-auto-import/vite'
  import Components from 'unplugin-vue-components/vite'
  import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

  // https://vitejs.dev/config/
  export default defineConfig({
    plugins: [
      vue(),
      AutoImport({
        resolvers: [ElementPlusResolver()],
      }),
      Components({
        resolvers: [ElementPlusResolver()],
      }),
    ],
  })
  ```

<details>
<summary>插入一项不属于 element-plus 的内容</summary><br>

点击 App.vue 时突然提示一个 `issue`, 内容是:

```markdown
❗ Different vue-tsc version

The Vue Language Features (Volar)'s version is 1.0.9, but the workspace's vue-tsc version is 0.40.13. This may produce different type checking behavior.
```

暂时没有遇到因为这个 issue 造成的错误, 如果有, 可以参考一下 [这篇博客](https://www.xiaoboy.com/topic/202108302255.html)

修改 package.json 中的 build 脚本: `"build": "vue-tsc --noEmit --skipLibCheck && vite build",`

[`--skipLibCheck`](https://github.com/johnsoncodehk/vue-tsc/issues/16) 参数的含义是, 不检查 `node_modules` 文件夹

<br></details>

测试一下 **组件** 和 **组件API** 是否正常

新建一个 **测试组件** vue 文件, 写入 [内容](https://raw.githubusercontent.com/element-plus/element-plus/dev/docs/examples/date-picker/enter-date.vue)

<details>
<summary><b>src/components/DatePicker.vue</b></summary>

```vue
<template>
  <div>
    <el-radio-group v-model="size" label="size control">
      <el-radio-button label="large">large</el-radio-button>
      <el-radio-button label="default">default</el-radio-button>
      <el-radio-button label="small">small</el-radio-button>
    </el-radio-group>
  </div>
  <div class="demo-date-picker">
    <div class="block">
      <span class="demonstration">Default</span>
      <el-date-picker
        v-model="value1"
        type="date"
        placeholder="Pick a day"
        :size="size"
      />
    </div>
    <div class="block">
      <span class="demonstration">Picker with quick options</span>
      <el-date-picker
        v-model="value2"
        type="date"
        placeholder="Pick a day"
        :disabled-date="disabledDate"
        :shortcuts="shortcuts"
        :size="size"
      />
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'

const size = ref<'' | 'large' | 'small'>('')

const value1 = ref('')
const value2 = ref('')

const shortcuts = [
  {
    text: 'Today',
    value: new Date(),
  },
  {
    text: 'Yesterday',
    value: () => {
      const date = new Date()
      date.setTime(date.getTime() - 3600 * 1000 * 24)
      return date
    },
  },
  {
    text: 'A week ago',
    value: () => {
      const date = new Date()
      date.setTime(date.getTime() - 3600 * 1000 * 24 * 7)
      return date
    },
  },
]

const disabledDate = (time: Date) => {
  return time.getTime() > Date.now()
}
</script>
<style scoped>
.demo-date-picker {
  display: flex;
  width: 100%;
  padding: 0;
  flex-wrap: wrap;
}
.demo-date-picker .block {
  padding: 30px 0;
  text-align: center;
  border-right: solid 1px var(--el-border-color);
  flex: 1;
}
.demo-date-picker .block:last-child {
  border-right: none;
}
.demo-date-picker .demonstration {
  display: block;
  color: var(--el-text-color-secondary);
  font-size: 14px;
  margin-bottom: 20px;
}
</style>
```

</details><br>

再新建一个 **测试组件API** vue 文件:

```vue
<!-- src/components/MessageBox.vue -->
<template>
  <el-button text @click="open">Click to open the Message Box</el-button>
</template>

<script lang="ts" setup>
import { ElMessage, ElMessageBox, Action } from 'element-plus'

const open = () => {
  ElMessageBox.alert('This is a message', 'Title', {
    // if you want to disable its autofocus
    // autofocus: false,
    confirmButtonText: 'OK',
    callback: (action: Action) => {
      ElMessage({
        type: 'info',
        message: `action: ${action}`,
      })
    },
  })
}
</script>

```


然后调用两个 vue 组件:

```vue
<!-- src/components/HelloWorld.vue -->
<script setup lang="ts">
import DatePicker from './DatePicker.vue'
import MessageBox from './MessageBox.vue'
</script>
<template>
  <DatePicker />
  <MessageBox />
</template>
```

测试结果为: **element-plus组件** 没问题, 但是 **element-plus组件API** 的样式出现问题,
查看官网, 可以看到这么一句话:

> 如果您使用 unplugin-element-plus 并且只使用组件 API，您需要手动导入样式。

在 `MessageBox.vue` 中, 共用到了 `message` 和 `message-box` 这两个组件, 故需要导入这两个组件的样式:

```vue
<!-- src/components/MessageBox.vue -->
<script lang="ts" setup>
import 'element-plus/es/components/message/style/css'
import 'element-plus/es/components/message-box/style/css'
</script>
```

最终完美的实现了, 并且打包后也没问题.

在 vite 运行过程中, `vite.config.ts` 中的 `unplugin-vue-components`
帮助我们实现了自动创建 `components.d.ts` 文件,
该文件会自动声明我们导入的 element-plus 组件和其他 vue 组件,
通过 `dts` 参数可以很方便的修改此声明文件的默认位置:(`auto-imports.d.ts` 文件同理)

```ts
// vite.config.ts
export default defineConfig({
  plugins: [
    AutoImport({
      dts: 'src/types/auto-imports.d.ts',
    }),
    Components({
      dts: 'src/types/components.d.ts',
    }),
  ],
})
```

### 为 `src` 路径设置一个别名

修改 `vite.config.ts` 文件夹:

```ts
import path from 'path' /* 这里要安装一下 @types/node */
const pathSrc = path.resolve(__dirname, 'src')
export default defineConfig({
  resolve: {
    alias: {
      '~/': `${pathSrc}/`,
    },
  },
})
```

然后在在 `main.ts` 中测试一下:

```ts
// src/main.ts
// 之前
import './style.css'
import App from './App.vue'
// 之后
import '~/style.css'
import App from '~/App.vue'
```

测试成功!

最后, 再配置一下 `tsconfig.json`, 让 vscode 能够将 `~` 识别为路径别名

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "~/*": ["src/*"]
    },
  },
}
```

### 使用 sass

使用 sass 特别简单, 直接安装 `sass` 就可以使用

```bash
npm install --save-dev sass
```

然后创建几个 .scss 文件

- `styles/index.scss`

  ```scss
  @import './variables.scss';
  @import './mixin.scss';

  body {
      @include scrollBar;

      height          : 100%;
      background-color: $theme;
  }
  ```

- `styles/variables.scss`

  ```scss
  $theme :skyblue;
  $font-color: green;
  ```

- `styles/mixin.scss`

  ```scss
  @mixin scrollBar {
      &::-webkit-scrollbar-track-piece {
          background: #d3dce6;
      }

      &::-webkit-scrollbar {
          width: 6px;
      }

      &::-webkit-scrollbar-thumb {
          background   : #99a9bf;
          border-radius: 20px;
      }
  }
  ```

接着在 `main.ts` 中引入:

```ts
// main.ts
import '~/styles/index.scss' // global css
```

成功看到效果!

配置 `vite.config.ts` 文件, 让所有组件都能使用全局变量

```ts
// vite.config.ts
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `
          @use "~/styles/variables.scss" as *;
        `,
      },
    },
  },
})
```

然后在 `HelloWorld.vue` 中使用全局变量

```vue
<!-- HelloWorld.vue -->
<style scoped lang="scss">
.read-the-docs {
  color: $font-color;
}
</style>
```

成功!

