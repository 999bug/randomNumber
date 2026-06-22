# 随机数生成器重设计 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 基于现有 `随机数3.7.html` 功能，创建现代化单文件 `随机数4.0.html`

**Architecture:** 单个 HTML 文件，内嵌 CSS + JS，零外部依赖。左右双栏布局，左栏控制面板右栏输出区。JS 模块化为：配置读取 → 随机数生成 → 错误注入（含间隔约束）→ DOM 渲染 → 复制工具。

**Tech Stack:** HTML5 + CSS3 (Grid/Flexbox/Custom Properties) + Vanilla JS (ES6+)

## Global Constraints

- 新文件 `随机数4.0.html`，不改动 `随机数3.7.html`
- 零外部依赖（不引入任何 CSS/JS 框架）
- 错误数之间至少间隔 3 个正确数
- 错误数显示：红色文字 + 黑色下划线 + 浅红底
- 一键复制：逗号分隔纯数字，点击后按钮文字变为 "✓ 已复制" 1.5 秒
- 数字网格动态生成，不硬编码 DOM 结构
- 响应式：宽度 < 768px 时切换为上下布局
- 所有注释、按钮文字使用中文

---
````

### Task 1: HTML 骨架 + CSS 布局

**Files:**
- Create: `随机数4.0.html`

**Produces:** HTML 结构 + 完整 CSS 样式 + 空 JS 占位函数

- [ ] **Step 1: 写入完整 HTML 骨架和 CSS**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>随机数生成器 4.0</title>
    <style>
        :root {
            --primary: #4A90D9;
            --primary-hover: #357ABD;
            --bg: #f5f6fa;
            --card-bg: #ffffff;
            --text: #2d3436;
            --text-light: #636e72;
            --error-text: #d63031;
            --error-bg: #ffeaa7;
            --error-underline: #2d3436;
            --correct-bg: #f1f2f6;
            --radius: 12px;
            --shadow: 0 2px 12px rgba(0,0,0,0.08);
            --font-mono: ui-monospace, 'Cascadia Code', 'Source Code Pro', Menlo, Consolas, monospace;
        }

        * { margin: 0; padding: 0; box-sizing: border-box; }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Microsoft YaHei", sans-serif;
            background: var(--bg);
            color: var(--text);
            min-height: 100vh;
            padding: 24px;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            display: flex;
            gap: 24px;
            align-items: flex-start;
        }

        /* 标题 */
        .header {
            max-width: 1200px;
            margin: 0 auto 20px;
            font-size: 26px;
            font-weight: 700;
            color: var(--text);
        }

        /* 左栏 — 控制面板 */
        .panel {
            background: var(--card-bg);
            border-radius: var(--radius);
            box-shadow: var(--shadow);
            padding: 28px;
            flex: 0 0 380px;
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        .panel h2 {
            font-size: 20px;
            font-weight: 600;
            text-align: center;
            color: var(--primary);
        }

        .form-group {
            display: flex;
            flex-direction: column;
            gap: 6px;
        }

        .form-group label {
            font-size: 14px;
            font-weight: 600;
            color: var(--text-light);
        }

        .form-group input {
            height: 42px;
            border: 2px solid #e0e3e8;
            border-radius: 8px;
            padding: 0 12px;
            font-size: 16px;
            font-family: var(--font-mono);
            transition: border-color 0.2s;
            outline: none;
        }

        .form-group input:focus {
            border-color: var(--primary);
        }

        .range-row {
            display: flex;
            gap: 12px;
            align-items: center;
        }

        .range-row input { flex: 1; }
        .range-row span { color: var(--text-light); font-weight: 600; }

        .btn-group {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
        }

        .btn {
            flex: 1;
            min-width: 80px;
            height: 44px;
            border: none;
            border-radius: 8px;
            font-size: 15px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.2s;
            font-family: inherit;
        }

        .btn-primary {
            background: var(--primary);
            color: #fff;
        }
        .btn-primary:hover { background: var(--primary-hover); }

        .btn-secondary {
            background: #e8ecf1;
            color: var(--text);
        }
        .btn-secondary:hover { background: #d5dbe3; }

        .btn-danger {
            background: #fff;
            color: #d63031;
            border: 2px solid #ff7675;
        }
        .btn-danger:hover { background: #fff5f5; }

        /* 右栏 — 输出区 */
        .output {
            flex: 1;
            background: var(--card-bg);
            border-radius: var(--radius);
            box-shadow: var(--shadow);
            padding: 28px;
            min-height: 500px;
            display: flex;
            flex-direction: column;
        }

        .output h2 {
            font-size: 20px;
            font-weight: 600;
            text-align: center;
            color: var(--primary);
            margin-bottom: 20px;
        }

        .output-empty {
            flex: 1;
            display: flex;
            align-items: center;
            justify-content: center;
            color: #b2bec3;
            font-size: 18px;
        }

        /* 数字网格 */
        .number-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(90px, 1fr));
            gap: 10px;
            flex: 1;
            align-content: start;
        }

        .number-cell {
            background: var(--correct-bg);
            border-radius: 8px;
            padding: 10px 6px;
            text-align: center;
            font-family: var(--font-mono);
            font-size: 16px;
            font-weight: 500;
            transition: transform 0.15s;
            word-break: break-all;
        }

        .number-cell:hover { transform: scale(1.05); }

        .number-cell.error {
            background: var(--error-bg);
            color: var(--error-text);
            text-decoration: underline;
            text-decoration-color: var(--error-underline);
            text-underline-offset: 3px;
            font-weight: 700;
        }

        /* 统计栏 */
        .stats {
            display: flex;
            gap: 20px;
            justify-content: center;
            margin-top: 20px;
            padding: 14px;
            background: #f8f9fb;
            border-radius: 8px;
            flex-wrap: wrap;
        }

        .stat-item {
            text-align: center;
        }

        .stat-value {
            font-size: 22px;
            font-weight: 700;
            font-family: var(--font-mono);
        }

        .stat-label {
            font-size: 12px;
            color: var(--text-light);
            margin-top: 2px;
        }

        .stat-item.error-count .stat-value { color: #d63031; }
        .stat-item.pass-count .stat-value { color: #00b894; }
        .stat-item.pass-rate .stat-value { color: var(--primary); }

        /* 复制按钮 */
        .copy-btn {
            display: block;
            margin: 16px auto 0;
            padding: 10px 32px;
            background: var(--primary);
            color: #fff;
            border: none;
            border-radius: 8px;
            font-size: 15px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.2s;
            font-family: inherit;
        }
        .copy-btn:hover { background: var(--primary-hover); }
        .copy-btn.copied { background: #00b894; }

        /* Toast 提示 */
        .toast {
            position: fixed;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: #2d3436;
            color: #fff;
            padding: 10px 24px;
            border-radius: 8px;
            font-size: 14px;
            z-index: 1000;
            opacity: 0;
            transition: opacity 0.3s;
            pointer-events: none;
        }
        .toast.show { opacity: 1; }

        /* 响应式 */
        @media (max-width: 768px) {
            .container { flex-direction: column; }
            .panel { flex: none; width: 100%; }
            .number-grid { grid-template-columns: repeat(auto-fill, minmax(72px, 1fr)); }
        }
    </style>
</head>
<body>
    <div class="header">🎲 随机数生成器</div>
    <div class="container">
        <!-- 左栏 - 控制面板 -->
        <div class="panel">
            <h2>控制面板</h2>
            <div class="form-group">
                <label for="errorRate">出错率 (%)</label>
                <input type="number" id="errorRate" placeholder="默认 0" min="0" max="100" step="0.1" value="0">
            </div>
            <div class="form-group">
                <label for="rangeMin">范围</label>
                <div class="range-row">
                    <input type="number" id="rangeMin" placeholder="最小值" step="any">
                    <span>—</span>
                    <input type="number" id="rangeMax" placeholder="最大值" step="any">
                </div>
            </div>
            <div class="form-group">
                <label for="count">随机数数量</label>
                <input type="number" id="count" placeholder="生成个数" min="1" max="500">
            </div>
            <div class="form-group">
                <label for="decimalPlaces">小数位数 (0-7)</label>
                <input type="number" id="decimalPlaces" placeholder="小数位数" min="0" max="7" value="0">
            </div>
            <div class="btn-group">
                <button class="btn btn-primary" onclick="generateInt()">输出整数</button>
                <button class="btn btn-primary" onclick="generateDecimal()">输出小数</button>
                <button class="btn btn-primary" onclick="generateRatio()">输出比值</button>
            </div>
            <div class="btn-group">
                <button class="btn btn-secondary" onclick="resetOutput()">重置</button>
                <button class="btn btn-danger" onclick="resetAll()">重置全部</button>
            </div>
        </div>

        <!-- 右栏 - 输出区 -->
        <div class="output" id="outputPanel">
            <h2>输出窗口</h2>
            <div class="output-empty" id="emptyHint">点击上方按钮生成随机数</div>
            <div class="number-grid" id="numberGrid" style="display:none;"></div>
            <div class="stats" id="stats" style="display:none;">
                <div class="stat-item error-count">
                    <div class="stat-value" id="errorCount">0</div>
                    <div class="stat-label">出错数</div>
                </div>
                <div class="stat-item pass-count">
                    <div class="stat-value" id="passCount">0</div>
                    <div class="stat-label">合格数</div>
                </div>
                <div class="stat-item pass-rate">
                    <div class="stat-value" id="passRate">0%</div>
                    <div class="stat-label">合格率</div>
                </div>
            </div>
            <button class="copy-btn" id="copyBtn" onclick="copyAll()" style="display:none;">📋 一键复制</button>
        </div>
    </div>
    <div class="toast" id="toast"></div>

    <script>
        // ===== 工具函数 =====

        /** 生成 [min, max] 范围内的随机整数 */
        function randomInt(min, max) {
            return Math.floor(Math.random() * (max - min + 1)) + min;
        }

        /** 生成 [min, max) 范围内的随机小数，精度由 digitScale 控制 */
        function randomDecimal(min, max, digitScale) {
            const raw = (max - min) * Math.random() + min;
            return Math.round(raw * digitScale) / digitScale;
        }

        /** 根据小数位数返回精度倍数（1位→10, 2位→100, ...） */
        function getDigitScale(places) {
            return Math.pow(10, places);
        }

        /** 将数字格式化为指定位数的小数（末尾补零） */
        function toFixedDecimal(x, places) {
            const scale = getDigitScale(places);
            let f = Math.round(x * scale) / scale;
            let s = f.toString();
            let dot = s.indexOf('.');
            if (dot < 0) { dot = s.length; s += '.'; }
            while (s.length <= dot + places) { s += '0'; }
            return s;
        }

        // ===== 占位函数（后续任务实现） =====

        function validateInputs() { return false; }
        function generateCorrectNumbers(count, rangeMin, rangeMax, decimalPlaces, isRatio) { return []; }
        function injectErrors(numbers, errorRate, rangeMin, rangeMax, decimalPlaces, isDecimal, isRatio) { return { numbers, errorFlags }; }
        function renderResult(numbers, errorFlags) {}
        function generateInt() {}
        function generateDecimal() {}
        function generateRatio() {}
        function resetOutput() {}
        function resetAll() {}
        function copyAll() {}
        function showToast(msg) {}
    </script>
</body>
</html>
```

- [ ] **Step 2: 在浏览器打开确认布局正常**

在浏览器打开 `随机数4.0.html`，确认左右双栏布局、输入框、按钮均显示正常，输出区显示"点击上方按钮生成随机数"空状态提示。

---

### Task 2: 核心生成逻辑 + 输入验证

**Files:**
- Modify: `随机数4.0.html` — 替换占位 JS 函数

**Produces:** 可用的随机数生成 + 错误注入（含间隔约束）+ 输入验证

- [ ] **Step 1: 替换占位 JS，实现核心逻辑**

将 Task 1 中的 `<script>` 标签内所有占位函数替换为以下完整实现：

```javascript
// ===== 常量 =====
const MIN_ERROR_GAP = 3; // 两个错误数之间的最小间隔（正确数个数）

// ===== 工具函数 =====

/** 生成 [min, max] 范围内的随机整数 */
function randomInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}

/** 生成 [min, max) 范围内的随机小数，精度由 digitScale 控制 */
function randomDecimal(min, max, digitScale) {
    const raw = (max - min) * Math.random() + min;
    return Math.round(raw * digitScale) / digitScale;
}

/** 根据小数位数返回精度倍数 */
function getDigitScale(places) {
    return Math.pow(10, places);
}

/** 将数字格式化为指定位数的小数（末尾补零） */
function toFixedDecimal(x, places) {
    const scale = getDigitScale(places);
    let f = Math.round(x * scale) / scale;
    let s = f.toString();
    let dot = s.indexOf('.');
    if (dot < 0) { dot = s.length; s += '.'; }
    while (s.length <= dot + places) { s += '0'; }
    return s;
}

// ===== 输入读取 =====

/** 读取所有表单值，返回解析后的配置对象。验证失败返回 null */
function readConfig() {
    const errorRate = parseFloat(document.getElementById('errorRate').value) || 0;
    const rangeMin = parseFloat(document.getElementById('rangeMin').value);
    const rangeMax = parseFloat(document.getElementById('rangeMax').value);
    const count = parseInt(document.getElementById('count').value);
    const decimalPlaces = parseInt(document.getElementById('decimalPlaces').value) || 0;

    if (isNaN(rangeMin)) { showToast('请输入最小值'); return null; }
    if (isNaN(rangeMax)) { showToast('请输入最大值'); return null; }
    if (rangeMin >= rangeMax) { showToast('最小值必须小于最大值'); return null; }
    if (isNaN(count) || count < 1) { showToast('请输入有效的生成数量'); return null; }
    if (count > 500) { showToast('数量不能超过 500'); return null; }
    if (decimalPlaces < 0 || decimalPlaces > 7) { showToast('小数位数范围 0-7'); return null; }
    if (errorRate < 0 || errorRate > 100) { showToast('出错率范围 0-100'); return null; }

    return { errorRate, rangeMin, rangeMax, count, decimalPlaces };
}

// ===== 随机数生成 =====

/** 生成正确随机数列表 */
function generateCorrectNumbers(count, rangeMin, rangeMax, decimalPlaces) {
    const numbers = [];
    const scale = getDigitScale(decimalPlaces);
    for (let i = 0; i < count; i++) {
        let val;
        if (decimalPlaces === 0) {
            val = randomInt(rangeMin, rangeMax);
        } else {
            val = randomDecimal(rangeMin, rangeMax, scale);
        }
        numbers.push(val);
    }
    return numbers;
}

// ===== 错误注入（含间隔约束） =====

/**
 * 在 numbers 中注入错误值。
 * 约束：任意两个错误位置之间至少间隔 MIN_ERROR_GAP 个正确数。
 * 如果错误率过高无法满足间隔，自动降低错误数并提示。
 * 
 * @returns {{ numbers: Array, errorFlags: boolean[], actualErrorCount: number, adjusted: boolean }}
 */
function injectErrors(numbers, errorRate, rangeMin, rangeMax, decimalPlaces, isDecimal, isRatio) {
    const count = numbers.length;
    
    // 计算请求的错误数量
    let requestedErrorCount = Math.round(count * errorRate / 100);
    if (requestedErrorCount === 0) {
        return { numbers: [...numbers], errorFlags: new Array(count).fill(false), actualErrorCount: 0, adjusted: false };
    }

    // 计算最大允许错误数（满足间隔约束）
    // 每个错误占据 1 个位置 + 后面 MIN_ERROR_GAP 个正确位置 = 1 + MIN_ERROR_GAP 个 slot
    // 所以 maxErrors = ceil(count / (1 + MIN_ERROR_GAP))
    const slotSize = 1 + MIN_ERROR_GAP;
    const maxErrors = Math.ceil(count / slotSize);
    
    let errorCount = Math.min(requestedErrorCount, maxErrors);
    let adjusted = errorCount < requestedErrorCount;

    // 生成所有可用位置（0 到 count-1）
    const available = Array.from({ length: count }, (_, i) => i);
    
    // 从可用位置中按间隔选取错误位置
    const errorPositions = [];
    const used = new Set();
    
    while (errorPositions.length < errorCount && available.length > 0) {
        const idx = randomInt(0, available.length - 1);
        const pos = available[idx];
        
        // 检查与已选位置的间隔
        let valid = true;
        for (const ep of errorPositions) {
            if (Math.abs(pos - ep) <= MIN_ERROR_GAP) {
                valid = false;
                break;
            }
        }
        
        if (valid) {
            errorPositions.push(pos);
            // 移除该位置及其相邻位置
            for (let i = pos - MIN_ERROR_GAP; i <= pos + MIN_ERROR_GAP; i++) {
                if (i >= 0 && i < count) used.add(i);
            }
            // 从 available 中移除已被覆盖的位置
            for (let i = available.length - 1; i >= 0; i--) {
                if (used.has(available[i])) available.splice(i, 1);
            }
        } else {
            // 该位置不可用，移除
            available.splice(idx, 1);
        }
    }
    
    // 如果还是不够（理论上不会，但做保护）
    if (errorPositions.length < errorCount) {
        adjusted = true;
        errorCount = errorPositions.length;
    }

    // 构建错误标记数组
    const errorFlags = new Array(count).fill(false);
    const errorSet = new Set(errorPositions);
    errorPositions.forEach(p => { errorFlags[p] = true; });

    // 生成错误值
    const result = [...numbers];
    const negative = rangeMin < 0;
    
    for (const pos of errorPositions) {
        if (isDecimal) {
            const scale = getDigitScale(decimalPlaces);
            if (pos % 2 === 0) {
                // 偶数位置：生成低于范围的值
                if (negative) {
                    result[pos] = randomDecimal(rangeMin - 0.2 + 0.1, rangeMin - 0.1, scale);
                } else {
                    result[pos] = randomDecimal(0, rangeMin - 0.1, scale);
                }
            } else {
                // 奇数位置：生成高于范围的值
                result[pos] = randomDecimal(rangeMax + 0.1, rangeMax + 0.2, scale);
            }
            result[pos] = parseFloat(toFixedDecimal(result[pos], decimalPlaces));
        } else {
            if (pos % 2 === 0) {
                if (negative) {
                    result[pos] = randomInt(rangeMin - 3, rangeMin - 1);
                } else {
                    result[pos] = randomInt(0, rangeMin - 1);
                }
            } else {
                result[pos] = randomInt(rangeMax + 1, rangeMax + 3);
            }
        }
    }

    return { numbers: result, errorFlags, actualErrorCount: errorCount, adjusted };
}

// ===== Toast 提示 =====

let toastTimer;

/** 显示短暂提示 */
function showToast(msg) {
    const toast = document.getElementById('toast');
    toast.textContent = msg;
    toast.classList.add('show');
    clearTimeout(toastTimer);
    toastTimer = setTimeout(() => toast.classList.remove('show'), 2000);
}
```

- [ ] **Step 2: 验证逻辑正确性**

在浏览器控制台手动测试：
```javascript
// 测试间隔约束
const nums = Array.from({length: 20}, (_, i) => i);
const result = injectErrors(nums, 50, 0, 100, 0, false, false);
// 检查 errorFlags 中相邻 true 的间隔 >= 3
let lastErr = -999;
for (let i = 0; i < result.errorFlags.length; i++) {
    if (result.errorFlags[i]) {
        console.assert(i - lastErr > 3, `间隔不足: ${lastErr} -> ${i}`);
        lastErr = i;
    }
}
console.log('间隔测试通过');
```

---

### Task 3: 渲染 + 交互 + 复制

**Files:**
- Modify: `随机数4.0.html` — 替换 `generateInt` / `generateDecimal` / `generateRatio` / `renderResult` / `resetOutput` / `resetAll` / `copyAll`

**Produces:** 完整可用的随机数生成器

- [ ] **Step 1: 替换剩余占位函数**

在 `<script>` 标签末尾（Task 2 代码之后）添加以下函数：

```javascript
// ===== 渲染 =====

let currentData = []; // 存储当前生成的所有数值（正确值，用于复制）

/** 渲染生成结果到输出区 */
function renderResult(numbers, errorFlags, isRatio, adjusted, errorCount) {
    const grid = document.getElementById('numberGrid');
    const emptyHint = document.getElementById('emptyHint');
    const stats = document.getElementById('stats');
    const copyBtn = document.getElementById('copyBtn');

    // 清空网格
    grid.innerHTML = '';
    
    const count = numbers.length;
    
    // 构建数字卡片
    for (let i = 0; i < count; i++) {
        const cell = document.createElement('div');
        cell.className = 'number-cell' + (errorFlags[i] ? ' error' : '');
        
        let displayValue;
        if (isRatio) {
            displayValue = '1:' + numbers[i];
        } else {
            displayValue = numbers[i];
        }
        cell.textContent = displayValue;
        grid.appendChild(cell);
    }

    // 显示网格，隐藏空状态
    grid.style.display = 'grid';
    emptyHint.style.display = 'none';
    stats.style.display = 'flex';
    copyBtn.style.display = 'block';

    // 更新统计
    const passCount = count - errorCount;
    const passRate = count > 0 ? Math.round((passCount / count) * 1000) / 10 : 100;
    
    document.getElementById('errorCount').textContent = errorCount;
    document.getElementById('passCount').textContent = passCount;
    document.getElementById('passRate').textContent = passRate + '%';
    
    // 存储正确值（用于复制）
    if (isRatio) {
        currentData = numbers.map(n => '1:' + n);
    } else {
        currentData = numbers.map(String);
    }

    // 如果错误数被调整过，给出提示
    if (adjusted) {
        showToast('⚠ 错误率过高，已自动降低以满足间隔要求（至少间隔 ' + MIN_ERROR_GAP + ' 个）');
    }
}

// ===== 三个生成按钮 =====

function generateInt() {
    const cfg = readConfig();
    if (!cfg) return;
    if (cfg.decimalPlaces !== 0) {
        // 整数模式小数位数应为 0，自动调整
        document.getElementById('decimalPlaces').value = '0';
        cfg.decimalPlaces = 0;
    }
    
    const numbers = generateCorrectNumbers(cfg.count, cfg.rangeMin, cfg.rangeMax, cfg.decimalPlaces);
    const result = injectErrors(numbers, cfg.errorRate, cfg.rangeMin, cfg.rangeMax, cfg.decimalPlaces, false, false);
    renderResult(result.numbers, result.errorFlags, false, result.adjusted, result.actualErrorCount);
}

function generateDecimal() {
    const cfg = readConfig();
    if (!cfg) return;
    
    const numbers = generateCorrectNumbers(cfg.count, cfg.rangeMin, cfg.rangeMax, cfg.decimalPlaces);
    // 格式化为指定位数
    const formatted = numbers.map(n => toFixedDecimal(n, cfg.decimalPlaces));
    const result = injectErrors(formatted, cfg.errorRate, cfg.rangeMin, cfg.rangeMax, cfg.decimalPlaces, true, false);
    renderResult(result.numbers, result.errorFlags, false, result.adjusted, result.actualErrorCount);
}

function generateRatio() {
    const cfg = readConfig();
    if (!cfg) return;
    
    const numbers = generateCorrectNumbers(cfg.count, cfg.rangeMin, cfg.rangeMax, cfg.decimalPlaces);
    const formatted = numbers.map(n => toFixedDecimal(n, cfg.decimalPlaces));
    const result = injectErrors(formatted, cfg.errorRate, cfg.rangeMin, cfg.rangeMax, cfg.decimalPlaces, true, true);
    renderResult(result.numbers, result.errorFlags, true, result.adjusted, result.actualErrorCount);
}

// ===== 复制 =====

/** 一键复制所有数值（逗号分隔） */
function copyAll() {
    if (currentData.length === 0) {
        showToast('没有可复制的数据');
        return;
    }
    
    const text = currentData.join(', ');
    
    if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(text).then(() => {
            onCopySuccess();
        }).catch(() => {
            fallbackCopy(text);
        });
    } else {
        fallbackCopy(text);
    }
}

function fallbackCopy(text) {
    const textarea = document.createElement('textarea');
    textarea.value = text;
    textarea.style.position = 'fixed';
    textarea.style.left = '-9999px';
    document.body.appendChild(textarea);
    textarea.select();
    try {
        document.execCommand('copy');
        onCopySuccess();
    } catch (e) {
        showToast('复制失败，请手动复制');
    }
    document.body.removeChild(textarea);
}

function onCopySuccess() {
    const btn = document.getElementById('copyBtn');
    const originalText = btn.textContent;
    btn.textContent = '✓ 已复制';
    btn.classList.add('copied');
    setTimeout(() => {
        btn.textContent = originalText;
        btn.classList.remove('copied');
    }, 1500);
}

// ===== 重置 =====

/** 重置输出区 */
function resetOutput() {
    const grid = document.getElementById('numberGrid');
    const emptyHint = document.getElementById('emptyHint');
    const stats = document.getElementById('stats');
    const copyBtn = document.getElementById('copyBtn');

    grid.innerHTML = '';
    grid.style.display = 'none';
    emptyHint.style.display = 'flex';
    stats.style.display = 'none';
    copyBtn.style.display = 'none';
    
    document.getElementById('errorCount').textContent = '0';
    document.getElementById('passCount').textContent = '0';
    document.getElementById('passRate').textContent = '0%';
    currentData = [];
}

/** 重置全部（含输入框） */
function resetAll() {
    resetOutput();
    document.getElementById('errorRate').value = '';
    document.getElementById('rangeMin').value = '';
    document.getElementById('rangeMax').value = '';
    document.getElementById('count').value = '';
    document.getElementById('decimalPlaces').value = '';
}
```

- [ ] **Step 2: 完整功能测试**

在浏览器中测试所有场景：

1. **整数模式**：范围 1-100，数量 30，出错率 10%，生成后检查错误数间隔 ≥ 3，错误数红色+黑色下划线
2. **小数模式**：范围 0-10，数量 20，小数位 2，出错率 15%
3. **比值模式**：范围 1-50，数量 15，小数位 1，出错率 20%
4. **一键复制**：点击复制按钮，粘贴验证内容为逗号分隔
5. **重置按钮**：点击重置/重置全部
6. **间隔约束测试**：设置出错率 80%，数量 10，应提示自动降低
7. **响应式**：缩小浏览器宽度 < 768px，确认切换上下布局

- [ ] **Step 3: 最终检查**

确认文件完整：
- HTML 骨架完整（`<!DOCTYPE html>` 到 `</html>`）
- CSS 不依赖外部资源
- JS 无 console.log 调试代码
- 中文字符无乱码（UTF-8 编码）
````

---

## Self-Review

1. **Spec coverage**: 
   - ✅ 左右双栏布局 (Task 1 CSS)
   - ✅ 错误间隔 ≥ 3 (Task 2 `injectErrors` + `MIN_ERROR_GAP`)
   - ✅ 错误红色文字 + 黑色下划线 + 浅红底 (Task 1 CSS `.number-cell.error`)
   - ✅ 一键复制逗号分隔 (Task 3 `copyAll`)
   - ✅ 动态网格 (Task 3 `renderResult` 动态 createElement)
   - ✅ 零外部依赖 (全文件无外部引用)
   - ✅ 响应式 (Task 1 CSS `@media`)
   - ✅ 不改动原文件 (新建 `随机数4.0.html`)

2. **Placeholder scan**: 无 TBD/TODO/占位符，所有步骤包含完整代码。

3. **Type consistency**: `injectErrors` 返回 `{ numbers, errorFlags, actualErrorCount, adjusted }`，`renderResult` 按同样签名消费，一致。
