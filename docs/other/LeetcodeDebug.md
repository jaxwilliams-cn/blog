# Leetcode x vscode x typescript

最近闲来无事，捡起了 leetcode 复习复习，突然发现 leetcode 可以直接提交 typescript 代码了，
毫无疑问，作为前端工程师的我 pick 了 ts，
众所周知，leetcode 提交代码 debug 非常的慢，所以在本地配置一下开发环境和 debug 环境还是很有必要的
我使用的 vscode 编辑器，因此本文即是 vscode debug leetcode(typescript)的配置分享

# 需求清单

1. node, npm/yarn
2. git(可选的)
3. vscode
4. leetcode 账号

# 配置清单

`package.json`

```json
"devDependencies": {
  "@types/node": "^14.11.8",
  "ts-node": "^9.0.0",
  "typescript": "^4.0.3"
},
```

`tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2018",
    "module": "CommonJS",
    "strict": true,
    "outDir": "./build",
    "sourceMap": true
  },
  "include": ["src/**/*"]
}
```

`.vscode/launch.json`

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Current TS File",
      "skipFiles": ["<node_internals>/**"],
      "program": "${file}",
      "preLaunchTask": "tsc: build - tsconfig.json",
      "outFiles": ["${workspaceFolder}/build/**/*.js"]
    }
  ]
}
```

.gitignore

```
build
node_modules
```

`目录结构`

- src // 我习惯放在 src 目录下，如果希望放在其他目录，可以修改 tsconfig.json 的 include 配置
  - 2020-10 // 我喜欢按日期分类，也可以按题型分类
    - 2020-10-01.ts
- build // 自动生成的配置

# 如何 debug

直接在 ts 文件中打断点，按 F5 即可开始调试当前文件

# tips

## 题目贴在文件顶部

从 leetcodecopy 题目，且会自动附上连接，对于以后二刷很有帮助。

## 自己编写测试用例

使用 nodejs`assert`模块中的`strictEqual`, `deepStrictEqual`就可以快速的对代码做单元测试

如

```javascript
// https://leetcode-cn.com/problems/count-binary-substrings
strictEqual(countBinarySubstrings('00110011'), 6)
strictEqual(countBinarySubstrings('10101'), 4)
strictEqual(countBinarySubstrings('00111'), 2)
strictEqual(countBinarySubstrings('0100110'), 5)
```

## 常用数据结构整理成库

如树的结构

```typescript
// src/libs/tree.ts
class TreeNode {
  val: number
  left: TreeNode | null
  right: TreeNode | null
  constructor(val?: number, left?: TreeNode | null, right?: TreeNode | null) {
    this.val = val === undefined ? 0 : val
    this.left = left === undefined ? null : left
    this.right = right === undefined ? null : right
  }
  // 完全二叉树层序遍历还原二叉树
  static fromLevelOrderTraversal(arr: (number | null)[]): TreeNode | null {}

  toString() {
    // 中序遍历
    function _convertTree(root: TreeNode | null): TreeNode[] {
      if (root === null) return []
      return [..._convertTree(root.left), root, ..._convertTree(root.right)]
    }
    return _convertTree(this)
  }
}
```

## 关于坚持 ✊

少做一题不要紧，松懈一天没关系，没必要自我谴责，只要别放弃，慢慢的就成习惯了。
