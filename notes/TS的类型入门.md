# TypeScript 类型入门

> 本文面向有一定 JS/Vue/Node.js 开发经验，并熟悉 C++/Python 的开发者，快速理解 TypeScript 的类型系统、工具链与最佳实践。

---

## 一、TypeScript 是什么？

TypeScript 是 JavaScript 的超集，增加了静态类型系统。其目标是让 JS 项目具备更强的可维护性、可读性和工程可控性。

```ts
function add(a: number, b: number): number {
  return a + b;
}
```

## 二、前端如何使用 TS：以 Vue + Vite 为例

Vue 3 **强烈推荐**使用 TypeScript，结合 Vite 构建工具，体验非常丝滑。

### 1. 启用 TypeScript 项目

```bash
pnpm create vite@latest my-vue-app --template vue-ts
cd my-vue-app
pnpm install
pnpm run dev
```

在 tsconfig.json 中启用严格模式：

```js
{  
  "compilerOptions": {  
    "strict": true, // 强制类型检查  
    "types": ["vite/client"] // Vite 环境类型支持[6](@ref)  
  }  
}  
```

### 2. Vue SFC 中使用 TS

```vue
<script lang="ts" setup>
import { ref } from 'vue'
// 这里可以自动推导出类型，知识为了方便演示
const count = ref<number>(0)
function increment(): void {
  count.value++
}
</script>
```

TS 帮助你获得 IDE 自动补全、重构提示、组件 props 校验等。

---

## 三、后端如何使用 TS：以 Express 为例

### 1. 初始化 TS 项目

```bash
pnpm init
pnpm add express
pnpm add -D typescript @types/node @types/express tsx nodemon
npx tsc --init
```

### 2. 创建一个基本服务

```ts
// src/index.ts
import express, { Request, Response } from 'express';

const app = express();
app.use(express.json());

app.get('/', (req: Request, res: Response) => {
  res.send('Hello TypeScript!');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

---

## 四、TypeScript 工具链

| 工具                           | 用途                         |
| ---------------------------- | -------------------------- |
| `tsc`                        | TypeScript 编译器             |
| `tsx`                        | 更快的 TS/JS 执行器，支持 ESM 与 HMR |
| `vite`                       | 快速前端打包器，支持 TS              |
| `eslint + typescript-eslint` | 代码规范检查                     |
| `prettier`                   | 代码格式化                      |
| `typedoc`                    | 自动生成文档                     |
| `ts-node`                   | 直接运行 TS 代码（开发时）         |

> 提示：VS Code 提供实时类型错误提示和定义跳转​（Ctrl+点击）。

---

## 五、TypeScript 的类型系统（对比 C++）

### 1. 基本类型

| TS 类型     | C++ 类比          | 示例                         |
| --------- | --------------- | -------------------------- |
| `number`  | `int`, `double` | `let a: number = 42;`      |
| `string`  | `std::string`   | `let name: string = "TS";` |
| `boolean` | `bool`          | `let ok: boolean = true;`  |
| `any`     | `void*`         | `let x: any = 123;`        |
| `void`    | `void`          | `function f(): void {}`    |

举例：

```ts
let count: number = 5;      // ≈ C++ int/double  
let name: string = "John";  // ≈ std::string  
let isDone: boolean = true; // ≈ bool  
let nothing: void = null;   // ≈ void  
```

### 2. 复合类型

| TS 类型                 | C++ 类比                           | 示例                                         |
| ---------------------- | ---------------------------------- | ----------------------------------------------- |
| 数组 `number[]`         | `std::vector<int>`                 | `let arr: number[] = [1, 2];`                   |
| 元组 `[string, number]` | `std::tuple<std::string, int>`     | `let t: [string, number] = ["hi", 1];`          |
| 枚举 `enum`             | `enum class`                       | `enum Direction { Up, Down }`                   |
| 对象 `interface`        | `struct` / `class`                 | `interface User { name: string; age: number; }` |
| Generic `Array<T>`        | `template<typename T> class Array` | `let nums: Array<number> = [1, 2, 3];`          |

### 3. 函数类型

入参返回值有类型:

```ts
function greet(name: string): string {
  return `Hello, ${name}`
}
```

### 4. 泛型（Templates）

```ts
function identity<T>(arg: T): T {
  return arg;
}
```

---

## 六、TS vs JS 开发效率与收益对比

| 维度         | JS 项目              | TS 项目                  |
|--------------|----------------------|--------------------------|
| 学习曲线     | 快                   | 需要了解类型系统          |
| 初始开发效率 | 高                   | 稍低                     |
| 重构安全性   | 低，靠测试和经验      | 高，类型系统护航          |
| 大型项目维护性 | 难                   | 易于维护和协作            |

> 通常项目规模达到 3k+ 行代码，多人协作时，TS 的收益就开始显著放大。

---

## 七、从软件工程角度看 TS 的优势

* **类型是 API 的文档**：无需翻源码，一看类型定义即可理解用法；
* **类型是重构的安全网**：删除、改名、参数调整都能快速反馈；
* **类型是团队协作的契约**：接口规范统一，减少沟通成本；
* **配合 ESLint、Prettier、CI 检查**：构建严谨的工程体系。

TS 是现代 JavaScript 工程实践的基石。一些最佳实践：

​**避免 any**​ → 优先用 unknown + 类型守卫：

```ts
function isError(err: unknown): err is Error {  
  return err instanceof Error;  
}

function handle(err: unknown) {
  // 安全地检查某个未知类型是否是 Error
  // 在处理 try/catch 的错误时常用的技巧
  if (isError(err)) {
    console.error(err.message);  // 在这里 err 被推断为 Error 类型
  } else {
    console.error('Unknown error:', err);
  }
}
```

泛型替代重复类型​（类似 C++ 模板），如：

```ts
type ApiResponse<T> = { data: T; error?: string };  
```

**使用场景：**

2025年，除了一次性脚本或超小型工具，其他都建议用TypeScript.

现代集成开发环境（IDE），如VS Code，提供了卓越的TypeScript支持，带来了高级的IntelliSense功能，包括自动补全、实时错误检测、智能导航工具（如跳转到定义、查找所有引用）和强大的重构能力 。这意味着开发者在编写代码时能获得实时反馈，编辑器会建议方法、标记错误并提供自动修复选项，从而显著减少了工程师识别和修复简单错误或查找文档的时间 。

对生产力的影响是可衡量的：研究表明，将Visual Studio Code等工具与TypeScript集成可以显著减少调试时间，将生产力水平提高约20% 。此外，通过采用静态代码分析措施，企业可以节省约25%的开发人员效率，而TypeScript的类型系统本身就极大地增强了静态代码分析的能力 。

而且还有AI的提示，也是使用ts更好。基本可以理解500行以上的代码，ts的优势就十分显著，配合vscode以及AI工具的辅助，效率就会超过js，而且项目越大和维护周一越长，收益越显著。
