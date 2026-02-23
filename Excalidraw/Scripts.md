---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
```javascript
// 获取用户输入的行数和列数
let rowsStr = await utils.inputPrompt("请输入行数 (Rows):", "例如: 4", "4");
if (!rowsStr) return;
let colsStr = await utils.inputPrompt("请输入列数 (Columns):", "例如: 3", "3");
if (!colsStr) return;

const rows = parseInt(rowsStr);
const cols = parseInt(colsStr);

if (isNaN(rows) || isNaN(cols)) {
    new Notice("输入无效，请输入数字！");
    return;
}

// 设置表格单元格的默认宽和高
const cellWidth = 160;
const cellHeight = 50;

// 设置 Excalidraw 的画笔样式
ea.style.roughness = 0;           // 设置为 0，画出笔直的线条，更像标准表格
ea.style.strokeWidth = 1;         // 线条粗细
ea.style.fillStyle = "solid";     // 填充样式
ea.style.backgroundColor = "transparent"; // 背景透明

// 循环生成矩形单元格
for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
        // 计算每个单元格的 X 和 Y 坐标
        let x = c * cellWidth;
        let y = r * cellHeight;
        ea.addRect(x, y, cellWidth, cellHeight);
    }
}

// 将生成的元素添加到当前画布中，并自动居中选中
await ea.addElementsToView(true, true, true);
```