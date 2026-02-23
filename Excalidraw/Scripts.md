---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
```javascript
const { Modal } = obsidian;

// 1. 定义一个自定义弹窗界面类
class TableGridModal extends Modal {
    constructor(app, callback) {
        super(app);
        this.callback = callback;
    }

    onOpen() {
        const { contentEl } = this;
        contentEl.empty();
        
        // 设置弹窗标题和动态标签
        contentEl.createEl("h3", { text: "插入 Word 风格表格", attr: { style: "margin-top: 0;" } });
        const labelEl = contentEl.createEl("div", { 
            text: "请在下方滑动选择", 
            attr: { style: "margin-bottom: 10px; font-weight: bold; color: #d97706;" } 
        });

        // 创建网格容器 (默认 10x10)
        const maxRows = 10;
        const maxCols = 10;
        const gridEl = contentEl.createEl("div", {
            attr: { style: `display: grid; grid-template-columns: repeat(${maxCols}, 25px); gap: 2px; justify-content: center;` }
        });

        let cells = [];

        // 绘制交互格子
        for (let r = 1; r <= maxRows; r++) {
            for (let c = 1; c <= maxCols; c++) {
                const cell = gridEl.createEl("div", {
                    attr: { 
                        style: "width: