---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
```javascript
// 1. 清除可能残留的旧弹窗
const oldOverlay = document.getElementById("my-custom-table-grid");
if (oldOverlay) oldOverlay.remove();

// 2. 创建全屏半透明遮罩层
const overlay = document.createElement("div");
overlay.id = "my-custom-table-grid";
overlay.style.position = "fixed";
overlay.style.top = "0";
overlay.style.left = "0";
overlay.style.width = "100vw";
overlay.style.height = "100vh";
overlay.style.backgroundColor = "rgba(0, 0, 0, 0.4)";
overlay.style.zIndex = "999999";
overlay.style.display = "flex";
overlay.style.alignItems = "center";
overlay.style.justifyContent = "center";

// 3. 创建白色弹窗主体
const modal = document.createElement("div");
modal.style.backgroundColor = "var(--background-primary, #ffffff)";
modal.style.padding = "20px";
modal.style.borderRadius = "10px";
modal.style.boxShadow = "0 4px 20px rgba(0,0,0,0.3)";
modal.style.display = "flex";
modal.style.flexDirection = "column";
modal.style.alignItems = "center";

const title = document.createElement("h3");
title.innerText = "插入 Word 风格表格";
title.style.margin = "0 0 10px 0";
title.style.color = "var(--text-normal, #333)";

const label = document.createElement("div");
label.innerText = "请在下方滑动选择";
label.style.marginBottom = "15px";
label.style.fontWeight = "bold";
label.style.color = "#ea580c";

const maxRows = 10;
const maxCols = 10;
const gridEl = document.createElement("div");
gridEl.style.display = "grid";
gridEl.style.gridTemplateColumns = "repeat(10, 26px)";
gridEl.style.gap = "4px";

let cells = [];

// 4. 画表格的核心逻辑
async function generateTable(rows, cols) {
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
    new ea.obsidian.Notice("✅ 成功插入 " + cols + "x" + rows + " 表格！");
}

// 5. 循环生成 10x10 的交互格子
for (let r = 1; r <= maxRows; r++) {
    for (let c = 1; c <= maxCols; c++) {
        const cell = document.createElement("div");
        cell.style.width = "26px";
        cell.style.height = "26px";
        cell.style.border = "1px solid var(--background-modifier-border, #ccc)";
        cell.style.borderRadius = "2px";
        cell.style.cursor = "pointer";
        cell.style.transition = "all 0.1s";
        
        cell.dataset.row = r;
        cell.dataset.col = c;

        // 鼠标悬停事件（橙色高亮）
        cell.addEventListener("mouseenter", () => {
            label.innerText = c + " 列 × " + r + " 行 表格";
            cells.forEach(item => {
                const ir = parseInt(item.dataset.row);
                const ic = parseInt(item.dataset.col);
                if (ir <= r && ic <= c) {
                    item.style.backgroundColor = "#fed7aa"; 
                    item.style.borderColor = "#ea580c"; 
                } else {
                    item.style.backgroundColor = "transparent";
                    item.style.borderColor = "var(--background-modifier-border, #ccc)";
                }
            });
        });

        // 鼠标点击事件（生成表格并关闭弹窗）
        cell.addEventListener("click", () => {
            overlay.remove(); 
            generateTable(r, c); 
        });

        cells.push(cell);
        gridEl.appendChild(cell);
    }
}

// 点击遮罩层空白处可取消关闭
overlay.addEventListener("click", (e) => {
    if (e.target === overlay) overlay.remove();
});

// 6. 把拼装好的界面显示到屏幕上
modal.appendChild(title);
modal.appendChild(label);
modal.appendChild(gridEl);
overlay.appendChild(modal);
document.body.appendChild(overlay);
```