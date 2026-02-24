---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
// --- 核心排版算法（自动对齐并撑开行高） ---
async function autoLayoutTable(tableElements) {
    if (!tableElements || tableElements.length === 0) return;
    ea.copyViewElementsToEAforEditing(tableElements);
    const elements = ea.getElements();
    const rects = elements.filter(e => e.type === "rectangle");
    if (rects.length === 0) return;

    const TOLERANCE = 10;
    rects.sort((a, b) => {
        if (Math.abs(a.y - b.y) > TOLERANCE) return a.y - b.y;
        return a.x - b.x;
    });

    let rows = [];
    let currentRow = [rects[0]];
    for (let i = 1; i < rects.length; i++) {
        if (Math.abs(rects[i].y - currentRow[0].y) <= TOLERANCE) {
            currentRow.push(rects[i]);
        } else {
            rows.push(currentRow);
            currentRow = [rects[i]];
        }
    }
    rows.push(currentRow);

    let currentY = rows[0][0].y;
    for (let row of rows) {
        let maxHeight = Math.max(...row.map(r => r.height));
        for (let rect of row) {
            rect.y = currentY;
            rect.height = maxHeight;
        }
        currentY += maxHeight;
    }
    await ea.addElementsToView(false, false);
}

// --- 核心生成算法（高兼容版） ---
async function startProcess(rows, cols) {
    const cellWidth = 150;
    const cellHeight = 45;
    ea.style.roughness = 0;           
    ea.style.strokeWidth = 1;         
    ea.style.strokeColor = "#000000"; 
    ea.style.roundness = null;        
    ea.style.fontFamily = 2;
    ea.style.textAlign = "center";
    ea.style.verticalAlign = "middle";

    let tableIds = [];
    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            let x = c * cellWidth;
            let y = r * cellHeight;

            ea.style.backgroundColor = (r === 0) ? "#f3f4f6" : "#ffffff";
            ea.style.fillStyle = "solid";
            
            // 1. 生成矩形
            let rectId = ea.addRect(x, y, cellWidth, cellHeight);
            tableIds.push(rectId);

            // 2. 生成绑定的文本（使用兼容性最高的 addText）
            let textId = ea.addText(x + cellWidth/2, y + cellHeight/2, " ", {
                width: cellWidth - 10,
                textAlign: "center",
                verticalAlign: "middle",
                box: true
            });
            tableIds.push(textId);
        }
    }
    ea.addToGroup(tableIds);
    await ea.addElementsToView(true, true, true);
    
    // 生成后延迟0.5秒，自动整理一次坐标
    setTimeout(() => {
        const elements = ea.getViewElements().filter(el => tableIds.includes(el.id));
        autoLayoutTable(elements);
    }, 500);
}

// ==========================================
// 🚀 智能启动逻辑：根据是否选中了元素，决定执行什么功能
// ==========================================
const selectedElements = ea.getViewSelectedElements();

if (selectedElements.length > 0) {
    // 【模式A】如果你已经选中了一个表格，直接执行“一键对齐排版”，不弹窗！
    autoLayoutTable(selectedElements).then(() => {
        new ea.obsidian.Notice("✨ 散架的表格已重新对齐排版！");
    });
} else {
    // 【模式B】如果没有选中任何东西，弹出 Word 风格网格，让你新建表格
    const oldOverlay = document.getElementById("integrated-table-grid");
    if (oldOverlay) oldOverlay.remove();

    const overlay = document.createElement("div");
    overlay.id = "integrated-table-grid";
    Object.assign(overlay.style, {
        position: "fixed", top: "0", left: "0", width: "100vw", height: "100vh",
        backgroundColor: "rgba(0, 0, 0, 0.4)", zIndex: "999999",
        display: "flex", alignItems: "center", justifyContent: "center"
    });

    const modal = document.createElement("div");
    Object.assign(modal.style, {
        backgroundColor: "var(--background-primary, #ffffff)", padding: "20px",
        borderRadius: "10px", boxShadow: "0 4px 20px rgba(0,0,0,0.3)",
        display: "flex", flexDirection: "column", alignItems: "center"
    });

    const label = document.createElement("div");
    label.innerText = "请滑动选择表格大小";
    label.style.marginBottom = "15px";
    label.style.fontWeight = "bold";
    label.style.color = "#ea580c";

    const gridEl = document.createElement("div");
    gridEl.style.display = "grid";
    gridEl.style.gridTemplateColumns = "repeat(10, 26px)";
    gridEl.style.gap = "4px";

    let cells = [];

    for (let r = 1; r <= 10; r++) {
        for (let c = 1; c <= 10; c++) {
            const cell = document.createElement("div");
            Object.assign(cell.style, {
                width: "26px", height: "26px", cursor: "pointer",
                border: "1px solid var(--background-modifier-border, #ccc)", borderRadius: "2px"
            });
            cell.addEventListener("mouseenter", () => {
                label.innerText = `${c} 列 × ${r} 行 表格`;
                cells.forEach(item => {
                    const isSel = parseInt(item.dataset.row) <= r && parseInt(item.dataset.col) <= c;
                    item.style.backgroundColor = isSel ? "#fed7aa" : "transparent";
                    item.style.borderColor = isSel ? "#ea580c" : "var(--background-modifier-border, #ccc)";
                });
            });
            cell.dataset.row = r; cell.dataset.col = c;
            cell.addEventListener("click", () => { overlay.remove(); startProcess(r, c); });
            cells.push(cell);
            gridEl.appendChild(cell);
        }
    }

    overlay.addEventListener("click", (e) => { if (e.target === overlay) overlay.remove(); });
    modal.appendChild(label);
    modal.appendChild(gridEl);
    overlay.appendChild(modal);
    document.body.appendChild(overlay);
}