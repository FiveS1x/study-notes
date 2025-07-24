# 从 JavaScript 迁移到 TypeScript

[迁移官方文档](https://typescript.bootcss.com/tutorials/migrating-from-javascript.html)



# 接口相关

## 不一定存在的参数

1. **可选属性标记（`?`）**

   在类型定义中使用 `?` 标记属性为可选

   ```ts
   type User = {
     id: number;
     name: string;
     // 可选参数：不一定存在
     email?: string; 
     age?: number;
   };
   ```

   

2. **联合类型（`| undefined`）**

   ```ts
   type Product = {
     id: string;
     price: number;
     // 明确表示可能不存在
     discount?: number | undefined; 
   };
   ```

   

3. **索引签名（Index Signature）**

   索引签名是 TypeScript 中定义对象动态属性的方式。它表示对象**可以有任意数量的额外属性**，只要键名类型匹配

   ```ts
   interface MyObject {
     // 索引签名：允许任意字符串键，值为任意类型
     [key: string]: any;
     
     // 具体属性
     id: number;
     name: string;
   }
   ```

   

4. **工具类型（`Partial`）**

   使用 `Partial<T>` 快速生成所有属性可选的新类型

   ```ts
   interface Settings {
     theme: string;
     fontSize: number;
     darkMode: boolean;
   }
   
   // 所有属性变为可选
   type PartialSettings = Partial<Settings>; 
   
   const tempSettings: PartialSettings = {
     theme: "light" // ✅ 其他属性可省略
   };
   ```

   

5. **可选参数（函数中使用）**

   ```ts
   function sendMessage(
     content: string,
     // 可选参数
     recipient?: string, 
     // 显式允许 undefined
     priority: "high" | "low" | undefined = undefined 
   ) {
     // recipient 类型为 string | undefined
   }
   ```

   

### 使用建议

- **优先用 `?`**：表示逻辑上的可选属性
- **需要显式 `undefined` 时**：用 `prop: T | undefined`
- **动态属性**：使用索引签名 `[key: string]: T | undefined`
- **函数参数**：用 `param?: T` 或默认值 `param: T = defaultValue`



## any类型和unknow类型的区别

在 TypeScript 中，`any` 和 `unknown` 都是用于处理不确定类型的值，但它们之间存在一些关键性差异，主要体现在类型安全性和对这些类型的值允许的操作上。

### any类型

- `any` 类型表示可以是任何类型的值，使用它时，TypeScript 的类型系统将不会对这个值进行任何类型检查。这意味着你可以对一个 `any` 类型的变量进行任何操作，包括调用方法、访问属性等，而编译器不会报错。
- 使用场景通常是在与非TypeScript代码交互或者当你明确知道一个变量的类型但TypeScript无法推断时。
- 使用过多的any类型会使类型系统的保护减少，可能会导致出现运行错误。



### unknow类型

- unknown 类型也是表示可以是任何类型，但与 any 不同，它是一个更为严格的类型。当你声明一个变量为 unknown 类型时，TypeScript 的类型系统**==要求你在使用这个变量之前，必须先通过类型断言或类型守卫来明确其具体类型==**，否则你不能直接对其执行大部分操作，如调用方法或访问属性。
- `unknown` 适合在你不了解一个值的确切类型，但又不想完全放弃类型检查的时候使用。



# 变量声明相关

## 约定俗成的占位符

- 在编程中，`_` 前缀或单独的 `_` 是广泛接受的约定，表示"这个变量是必需的，但我故意不使用它"。下方案例中的from定义了但未使用，所以前缀加 `_` 。

  ```ts
  router.beforeEach((to, _from, next) => {
    const store = useMainStore();
  
    // 设置页面标题
    const title = (to.meta.title as string) || store.appName;
    document.title = `${title} | ${store.appName}`;
  
    // 认证检查
    if (to.meta.requiresAuth && !store.isAuthenticated) {
      next({ name: "Login" });
    } else {
      next();
    }
  });
  ```

  

- 这常见于函数参数、解构赋值等场景