---
name: stock-fund-flow
description: >
  查询 A 股行业板块和概念板块的资金流向，包括主力净流入/流出排名。
  使用东方财富数据中心，通过页面数据组件直接获取完整 JSON 数据。
  当用户说"板块资金"、"资金流向"、"主力净流入"、"板块排行"时触发。
version: 1.0.0
---

# A 股板块资金流向查询

## 数据源

| 类型 | 页面 |
|------|------|
| 行业板块 | https://data.eastmoney.com/bkzj/hy.html |
| 概念板块 | https://data.eastmoney.com/bkzj/gn.html |

## 必须执行的步骤

### 1. 切换桌面版 UA

browser_use set_user_agent desktop_chrome

手机版只显示前 20 名且无搜索功能，必须桌面版 UA

### 2. 导航到页面

navigate: https://data.eastmoney.com/bkzj/hy.html
wait_for_dom_stable

### 3. 从 dataview 组件获取完整数据

使用 execute_js 从 jQuery dataview 组件取数据：

const dv = jQuery('#dataview').data('dataview');
const rows = dv.data; // 当前页数据（每页 50 条）

### 4. 数据字段

f14: 板块名称
f2: 板块最新价
f3: 涨跌幅（%）
f62: 主力净流入（元）
f66: 超大单净流入（元）
f78: 大单净流入（元）
f84: 中单净流入（元）
f69: 小单净流入（元）

### 5. 排序和筛选

按涨跌幅排序: rows.sort((a, b) => b.f3 - a.f3)
筛选资金流入: rows.filter(r => r.f62 > 0)
筛选资金流出: rows.filter(r => r.f62 < 0)
取前 N 名: rows.slice(0, 10)

### 6. 数据解读

主力净流入 > 0: 大资金买入，板块可能走强
主力净流入 < 0: 大资金卖出，板块可能走弱
大跌 + 主力净流入: 洗盘（有承接）
大跌 + 主力净流出: 出货，需警惕

## 汇报格式

### 行业板块资金流入 TOP 5
| 板块 | 涨跌幅 | 主力净流入 |
|------|--------|-----------|
| 电力 | +2.46% | +55.6亿 |
| 银行Ⅱ | +1.60% | +24.5亿 |

### 行业板块资金流出 TOP 5
| 板块 | 涨跌幅 | 主力净流入 |
|------|--------|-----------|

## 注意事项

- 不要尝试手机版 UA 或 get_readable/get_text 读取表格
- 数据盘中实时更新
- 主力净流入 = 超大单 + 大单
- 行业板块共约 128 个，每页 50 条
