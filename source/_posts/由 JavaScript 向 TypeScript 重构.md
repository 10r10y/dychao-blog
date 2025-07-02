---
title: 由 JavaScript 向 TypeScript 重构
date: 2024-08-08 20:42:09
tags:
- 优化
categories: 
- JavaScript
---
## 由 JavaScript 向 TypeScript 重构

出于让我熟悉整个项目的考量，在接手一个新的项目时ld让我负责了老项目的 TypeScript 重构，记录一些思考和原则



#### 0. 迁移方式

首先要基于项目规模，判断采用全量迁移还是渐进迁移

- 【全量迁移】先将所有的 JS 文件改成 TS 文件，先享受到 TypeScript 的完整类型检查能力
- 【渐进迁移】每次完成一些文件的类型迁移，通过启用 `allowJs` 与 `checkJs` 配置，来先使用一部分类型检查能力

因为项目较为庞大，这里采用渐进迁移实现

1. 安装 TypeScript，并通过 `tsc --init` 初始化配置文件 `tsconfig.json`
2. 通过`"allowJs": true` 使 .js 与 .ts 可以共存
3. 优先修改的文件使用 `@ts-check` 启用该文件的类型检查
4. 使用 `@ts-ignore` 标记暂时无法解决的错误
5. 最后再开启 `"checkJs": true`  检查所有文件

```json
// tsconfig.json
{
  "compilerOptions": {
    "allowJs": true,    // 允许同时存在 .js 和 .ts 文件
    "checkJs": false,   // 暂时关闭对 JS 文件的全局类型检查
    "outDir": "./dist", // 确保构建输出目录配置正确
    // 其他必要配置...
  }
}
```



#### 1. 第一步：补充类型定义

编码工作的第一步，一定是基于项目的数据补充类型定义，即完善项目的整个类型体系，因为 TypeScript 再智能，也不可能自动补全这些类型，而没有这些类型，它的能力就是相当局限的。大部分地方会被推断为 any /  unknown  类型，导致类型检查基本失效了。所以需要先完善项目的整个类型体系，才能在后面的重构过程中，不断得到来自类型的提示与警告，从而“安全”地完成迁移流程。



#### 2. 先迁移通用工具方法，再迁移前端组件

完成类型定义的补充后，首先应该完善项目中的工具方法，大部分数据变化都发生在工具方法中，完成了工具方法的入参与出参的定义后，所有引用他们的组件都会收益。

当第一步的类型定义不断完善，再结合通用工具的类型重构完成，就会感觉到一张类型大网不断连结扩张，还是蛮有成就感的



#### 3. 避免逻辑改动，只做类型补充

避免一些多余的动作，比如逻辑看不顺眼向重写，npm包太老想更新，技术栈想同时重构等。

这样会导致不该有的bug出现，而定位问题再去回滚时又可能附带了无辜的类型重构代码，大大降低效率。

针对 TypeScript 的重构应该是纯粹的，如果实在想改可以先标记 TODO，之后再进行。



#### 4. 第三方库

如果流行且维护良好的库，大概率以及由对应的类型定义，只需要安装对应的 `@type/xx` 即可

如果是久远停止更新的库，需要创建 `.d.ts` 文件增补模块定义，可以创建一个 `types` 文件夹，加入自己的类型定义。然后就可以享受类型安全检查了。

```typescript
declare module 'old-http-client' {
  interface RequestOptions {
    timeout?: number;
    retries?: number;
  }

  export function get(url: string, options?: RequestOptions): Promise<string>;
}
```



#### 附.

- 其实调研时由看到一个自动重构为 TypeScript 的工具：`ts-migrate`，经过了解发现并不太适用于一个自己还不是很了解的项目，当bug出现时难以快速定位