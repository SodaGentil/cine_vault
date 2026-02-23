---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
```javascript
const { Modal, Notice } = ea.obsidian;

class TableGridModal extends Modal {
    constructor(app, callback) {
        super(app);
        this.callback = callback;
    }

    onOpen() {
        const { contentEl } = this;
        contentEl.empty();
        
        const title = contentEl.createEl("h3", { text: "插入 Word 风格表格" });
        title.style.marginTop = "0px";
        title.style.marginBottom = "10px";

        const labelEl = contentEl.createEl("div", { text: "请在下方滑动选择" });
        labelEl.style.marginBottom = "15px";
        labelEl.style.fontWeight = "bold";
        labelEl.style.color = "#ea580c";

        const maxRows = 10;
        const maxCols = 10;
        const gridEl = contentEl.createEl("div");
        gridEl.style.display = "grid";
        gridEl.style.gridTemplateColumns = "repeat(10, 28px)";
        gridEl.style.gap = "4px";
        gridEl.style.justifyContent = "center";
        gridEl.style.marginBottom = "10px";

        let cells = [];

        for (let r = 1; r <= maxRows; r++) {
            for (let c = 1; c <= maxCols; c++) {
                const cell = gridEl.createEl("div");
                cell.style.width = "28px";
                cell.style.height = "28px";
                cell.style.border = "1px solid #ccc";
                cell.style.borderRadius = "2px";
                cell.style.cursor = "pointer";
                cell.style.background = "transparent";
                
                cell.setAttribute("data-row", r.toString());
                cell.setAttribute("data-col", c.toString());

                cell.addEventListener("mouseenter", () => {
                    labelEl.innerText = c + " 列 × " + r + " 行 表格";
                    cells.forEach(item => {
                        const ir = parseInt(item.getAttribute("data-row"));
                        const ic = parseInt(item.getAttribute("data-col"));
                        if (ir <= r && ic <= c) {
                            item.style.background = "#fed7aa"; 
                            item.style.borderColor = "#ea580c"; 
                        } else {
                            item.style.background = "transparent";
                            item.style.borderColor = "#ccc";
                        }
                    });
                });

                cell.addEventListener("click", () => {
                    this.close(); 
                    this.callback(r, c); 
                });

                cells.push(cell);
            }
        }
    }

    onClose() {
        this.contentEl.empty();
    }
}

new TableGridModal(app, async (rows, cols) => {
    if (rows === 0 || cols === 0) return;

    const cellWidth = 150;
    const cellHeight = 40;

    ea.style.roughness = 0;           
    ea.style.strokeWidth = 1;         
    ea.style.strokeColor = "#000000"; 
    ea.style.roundness = null;        
    ea.style.fontFamily = 2;          

    let tableElements = [];

    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            let x = c * cellWidth;
            let y = r * cellHeight;

            if (r === 0) {
                ea.style.backgroundColor = "#f3f4f6"; 
                ea.style.fillStyle = "solid";
            } else {
                ea.style.backgroundColor = "#ffffff"; 
                ea.style.fillStyle = "solid"; 
            }

            let rectId = ea.addRect(x, y, cellWidth, cellHeight);
            tableElements.push(rectId);
        }
    }

    ea.addToGroup(tableElements);
    await ea.addElementsToView(true, true, true);
    new Notice("✅ 成功插入 " + cols + "x" + rows + " 表格！");
}).open();
```