## 路径提示插件 path-intellisense

![](./img/01.webp)

```json
"path-intellisense.mappings": {
  "@": "${workspaceRoot}/src"
}
```

## Ctrl + 鼠标左键跳转

`package.json`文件同目录下编辑`jsconfig.json`或`tsconfig.json`

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "exclude": [
    "node_modules"
  ]
}
```