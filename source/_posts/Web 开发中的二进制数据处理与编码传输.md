---
title: File、Blob、ArrayBuffer、Base64、DataURL
date: 2024-07-16 19:35:26
categories: 
- JavaScript
---

# File、Blob、ArrayBuffer、Base64、DataURL

## 0. 简单概念

1. **二进制数据容器**
   - **Blob**：表示不可变的原始二进制数据（如文件片段、流数据），是浏览器处理二进制的基础类型
   - **File**：继承自 Blob，扩展了文件名、类型等元信息，代表用户系统中的文件（如 `<input type="file">` 上传的文件）
2. **底层数据操作**
   - **ArrayBuffer**：表示原始二进制缓冲区，需通过 `TypedArray`（如 `Uint8Array`）或 `DataView` 进行读写，用于直接操作内存中的二进制数据（如解析文件内容）
3. **编码与传输格式**
   - **Base64**：将二进制数据编码为 ASCII 字符串，便于在文本协议（如 JSON、URL）中安全传输（体积增大约 33%）
   - **DataURL**：通过 `data:[MIME];base64,<Base64数据>` 格式内嵌资源（如图片），可直接嵌入 HTML/CSS，无需额外请求（例如 `src="data:image/png;base64,iVBORw..."`）

## 1. File

**本质：** 继承自Blob，表示用户文件系统中的文件

**特性：**

```
const file = new File(['content'], 'demo.txt', {
  type: 'text/plain',
  lastModified: Date.now()
});

console.log(file.name); // 'demo.txt'
console.log(file.size); // 7 (字节)
console.log(file.type); // 'text/plain'
```



## 2. Blob (Binary Large Object)

**作用：** 存储二进制数据的容器

**创建方法：**

```
// 从字符串创建
const blob1 = new Blob(['Hello World'], { type: 'text/plain' });

// 从ArrayBuffer创建
const buffer = new ArrayBuffer(8);
const blob2 = new Blob([buffer]);

// 合并Blob
const combined = new Blob([blob1, blob2]);
```



## 3. FileReader

**核心方法：**

```
const reader = new FileReader();

reader.onload = (e) => {
  console.log(e.target.result);
};

reader.readAsText(blob);    // 读取为文本
reader.readAsArrayBuffer(blob); // 读取为ArrayBuffer
reader.readAsDataURL(blob); // 读取为DataURL
```



## 4. ArrayBuffer

**特性：** 特性：固定长度的原始二进制缓冲区

**操作方式：**

```
const buffer = new ArrayBuffer(16);
const view = new Uint8Array(buffer);
view[0] = 255;
console.log(view); // Uint8Array [255, 0, 0, ...]
```



## 5. Base64

**编码原理：** 每3字节转换为4个ASCII字符

**转换示例：**

```
// 文本转Base64
btoa('Hello'); // 'SGVsbG8='

// Base64转文本
atob('SGVsbG8='); // 'Hello'
```



## 6. DataURL

**格式：** `data:[<mediatype>][;base64],<data>`

```
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA...
```



## 转换关系

### 1. File ↔ Blob

```
// File → Blob 
const blob = file.slice(0, file.size, file.type);

// Blob → File
const file = new File([blob], 'filename', { type: blob.type });
```



### 2. Blob ↔ ArrayBuffer

```
// Blob → ArrayBuffer
const reader = new FileReader();
reader.readAsArrayBuffer(blob);
reader.onload = () => {
  const buffer = reader.result;
};

// ArrayBuffer → Blob
const newBlob = new Blob([buffer]);
```



### 3. Blob ↔ DataURL

```
// Blob → DataURL
const reader = new FileReader();
reader.readAsDataURL(blob);

// DataURL → Blob
function dataURLtoBlob(dataurl) {
const arr = dataurl.split(','),
        mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]),
        n = bstr.length,
        u8arr = newUint8Array(n);
while(n--) u8arr[n] = bstr.charCodeAt(n);
returnnew Blob([u8arr], { type: mime });
}
```



### 4. ArrayBuffer ↔ Base64

```
// ArrayBuffer → Base64
function arrayBufferToBase64(buffer) {
let binary = '';
const bytes = newUint8Array(buffer);
for (let i = 0; i < bytes.byteLength; i++) {
    binary += String.fromCharCode(bytes[i]);
  }
return btoa(binary);
}

// Base64 → ArrayBuffer
function base64ToArrayBuffer(base64) {
const binaryString = atob(base64);
const len = binaryString.length;
const bytes = newUint8Array(len);
for (let i = 0; i < len; i++) {
    bytes[i] = binaryString.charCodeAt(i);
  }
return bytes.buffer;
}
```



## 小结

| 类型        | 内存占用 | 内存占用 | 适用场景                 |
| :---------- | :------- | :------- | :----------------------- |
| Blob        | 低       | 快       | 文件存储/切片上传        |
| ArrayBuffer | 最低     | 最快     | 二进制操作/WebSocket传输 |
| DataURL     | 高       | 慢       | 文件存储/切片上传        |
| Object URL  | 中       | 快       | 大文件预览               |