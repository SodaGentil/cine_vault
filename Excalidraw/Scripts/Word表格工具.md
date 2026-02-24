---
excalidraw-plugin: parsed
tags: [excalidraw/script]
---
// ==========================================
// 1. 核心排版算法：自动对齐并撑开行高
// ==========================================
async function autoLayoutTable(tableElements) {
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

// ==========================================
// 2. 交互界面逻辑：Word 风格 10x10 网格
// ==========================================
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

// 生成逻辑
async function startProcess(rows, cols) {
    const cellWidth = 150;
    const cellHeight = 45;
    ea.style.roughness = 0;           
    ea.style.strokeWidth = 1;         
    ea.style.strokeColor = "#000000";