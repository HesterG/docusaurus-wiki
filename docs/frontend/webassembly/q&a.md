# Q & A

## **Q: JS 导入 WAT 文件并运行？**

**A**: 你不能直接在 JavaScript 中导入 WAT（WebAssembly Text Format），但可以将 WAT 转换为 Wasm（WebAssembly 的二进制格式），然后在 JavaScript 中导入并运行。

### 实现步骤

1. **将 WAT 转换为 Wasm**:

   - 首先需要将 WAT（文本格式）转换为 Wasm（二进制格式）。可以使用 `wat2wasm` 工具。
   - 安装 `wabt` 工具集，它包含 `wat2wasm`：
     - [WebAssembly/wabt](https://github.com/WebAssembly/wabt) 项目页面提供安装说明。
     - 使用 macOS 命令安装：

       ```bash
       brew install wabt
       ```

   - 假设你有一个名为 `example.wat` 的文件，内容如下：

     ```wat
     (module
       (func (export "add") (param i32 i32) (result i32)
         get_local 0
         get_local 1
         i32.add))
     ```

     使用以下命令将其转换为 Wasm：

     ```bash
     wat2wasm example.wat -o example.wasm
     ```

2. **在 JavaScript 中加载和运行 Wasm**:
   - 一旦有了 `.wasm` 文件，可以通过 JavaScript 使用 `WebAssembly` API 来加载并运行。
   - 例如：

     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>WebAssembly Example</title>
     </head>
     <body>
         <script>
             fetch('example.wasm')
                 .then(response => response.arrayBuffer())
                 .then(bytes => WebAssembly.instantiate(bytes))
                 .then(results => {
                     const add = results.instance.exports.add;
                     console.log("5 + 3 =", add(5, 3));
                 })
                 .catch(console.error);
         </script>
     </body>
     </html>
     ```

### 解释

1. `fetch('example.wasm')`: 从服务器获取 Wasm 文件。
2. `WebAssembly.instantiate(bytes)`: 将字节数组编译成 WebAssembly 模块并实例化。
3. `results.instance.exports.add`: 从模块中提取 `add` 函数并运行。

---

## **Q: 然后可以让浏览器运行？**

**A**: 对的！浏览器无法直接识别 WAT，但可以识别和执行 Wasm（二进制 WebAssembly 文件）。通过将 WAT 转换为 Wasm，浏览器可以通过 JavaScript 使用 WebAssembly API 加载和运行。

### 步骤

1. **将 WAT 转换为 Wasm**: 首先，使用 `wat2wasm` 工具将 WAT（WebAssembly 的文本格式）转换为 Wasm（二进制格式）。浏览器只能识别和运行 Wasm 文件，而不能直接处理 WAT。

2. **在 JavaScript 中加载 Wasm**: 使用浏览器的 `WebAssembly` API 在 JavaScript 中加载和执行生成的 Wasm 文件。通过 `fetch` 加载 Wasm 文件并使用 `WebAssembly.instantiate()` 方法实例化并运行它。

3. **浏览器支持**: 主流浏览器（如 Chrome、Firefox、Safari 和 Edge）都原生支持 WebAssembly，因此它们能够识别、加载和执行 Wasm 文件。

### 总结

- **WAT 文件** 需要通过工具转换为 **Wasm 文件**，因为浏览器只能识别二进制格式的 Wasm。
- **Wasm 文件** 可以通过 **JavaScript** 代码在浏览器中加载并运行。
- 通过浏览器的 **WebAssembly API**，浏览器可以直接处理 Wasm 并高效运行。
