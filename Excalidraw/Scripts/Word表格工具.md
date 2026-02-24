---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
// === Excalidraw Word表格生成与双向整理工具 ===

// 1. 核心整理算法（升级为：支持高度与宽度的双向自动对齐）
async function autoLayoutTable(tableElements) {
    if (!tableElements || tableElements.length === 0) return;
    ea.copyViewElementsToEAforEditing(tableElements);
    const elements = ea.getElements();
    const rects = elements.filter(e => e.type === "rectangle");
    if (rects.length === 0) return;

    // 按 Y 轴坐标将格子分行 (容差 10px)
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

    // 强制每一行内部严格按 X 轴从左到右排序
    rows.forEach(row => row.sort((a, b) => a.x - b.x));

    // 计算每一列的“最大宽度”
    let colCount = Math.max(...rows.map(r => r.length));
    let colWidths = new Array(colCount).fill(0);
    for (let r = 0; r < rows.length; r++) {
        for (let c = 0; c < rows[r].length; c++) {
            if (rows[r][c].width > colWidths[c]) {
                colWidths[c] = rows[r][c].width;
            }
        }
    }

    // 计算每一行的“最大高度”
    let rowHeights = rows.map(row => Math.max(...row.map(cell => cell.height)));

    // 重新应用所有格子的 X, Y, Width, Height
    let currentY = rows[0][0].y; // 整个表格的起始 Y
    let startX = Math.min(...rows.map(r => r[0].x)); // 整个表格的起始 X

    for (let r = 0; r < rows.length; r++) {
        let currentX = startX;
        for (let c = 0; c < rows[r].length; c++) {
            let cell = rows[r][c];
            cell.x = currentX;           // 重置 X 坐标
            cell.y = currentY;           // 重置 Y 坐标
            cell.width = colWidths[c];   // 统一同列宽度
            cell.height = rowHeights[r]; // 统一同行高度
            currentX += colWidths[c];    // 下一个格子的起始 X 向右推移
        }
        currentY += rowHeights[r];       // 下一行的起始 Y 向下推移
    }

    await ea.addElementsToView(false, false);
}

// 2. 生成网格算法
async function startProcess(rows, cols) {
    const cellWidth = 150;
    const cellHeight = 45;
    ea.style.roughness = 0;
    ea.style.strokeWidth = 1;
    ea.style.strokeColor = "#000000";
    ea.style.roundness = null;
    ea.style.fontFamily = 2;

    let tableIds = [];
    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            let x = c * cellWidth;
            let y = r * cellHeight;
            ea.style.backgroundColor = (r === 0) ? "#f3f4f6" : "#ffffff";
            ea.style.fillStyle = "solid";
            
            let rectId = ea.addRect(x, y, cellWidth, cellHeight);
            tableIds.push(rectId);
        }
    }
    ea.addToGroup(tableIds);
    await ea.addElementsToView(true, true, true);
    
    setTimeout(() => {
        const elements = ea.getViewElements().filter(el => tableIds.includes(el.id));
        autoLayoutTable(elements);
    }, 500);
}

// 3. 交互与入口
const selectedElements = ea.getViewSelectedElements();

if (selectedElements.length > 0) {
    autoLayoutTable(selectedElements).then(() => {
        new ea.obsidian.Notice("✨ 表格的长宽已全面自适应对齐！");
    });
} else {
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
            cell.dataset.row = r; cell.dataset.col = c;
            cell.addEventListener("mouseenter", () => {
                label.innerText = `${c} 列 × ${r} 行 表格`;
                cells.forEach(item => {
                    const isSel = parseInt(item.dataset.row) <= r && parseInt(item.dataset.col) <= c;
                    item.style.backgroundColor = isSel ? "#fed7aa" : "transparent";
                    item.style.borderColor = isSel ? "#ea580c" : "var(--background-modifier-border, #ccc)";
                });
            });
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