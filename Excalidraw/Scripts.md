---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
```javascript
// 修复点：必须从 ea.obsidian 中解构出 Modal 和 Notice
const { Modal, Notice } = ea.obsidian;

class TableGridModal extends Modal {
    constructor(app, callback) {
        super(app);
        this.callback = callback;
    }

    onOpen() {
        const { contentEl } = this;
        contentEl.empty();
        
        contentEl.createEl("h3", { text: "插入 Word 风格表格", attr: { style: "margin-top: 0; margin-bottom: 10px;" } });
        const labelEl = contentEl.createEl("div", { 
            text: "请在下方滑动选择", 
            attr: { style: "margin-bottom: 15px; font-weight: bold; color: #ea580c;" } 
        });

        const maxRows = 10;
        const maxCols = 10;
        const gridEl = contentEl.createEl("div", {
            attr: { style: `display: grid; grid-template-columns: repeat(${maxCols}, 28px); gap: