# 商品浏览实验页面

这是一个单页面应用，用于商品浏览实验研究。所有功能使用原生JavaScript实现，无需复杂框架。

## 📚 文档导航

- **[快速开始](#快速开始)** - 立即开始使用
- **[功能说明](#功能说明)** - 了解页面功能
- **[部署指南](部署指南.md)** - GitHub Pages部署步骤
- **[Credamo配置指南](Credamo配置指南.md)** - 在Credamo平台中配置和使用
- **[数据字段说明](数据字段说明.md)** - 完整的数据字段和查看方法

## 🚀 快速开始

### 本地测试
直接在浏览器中打开 `index.html` 即可。

### 实验条件设置
通过URL参数设置实验条件：

```
index.html?condition=rating              # 只显示星级和数量
index.html?condition=aigrs_neutral       # AIGRS中立版本
index.html?condition=aigrs_positive      # AIGRS正向版本
```

### 完整URL示例（用于Credamo）
```
https://你的用户名.github.io/product-experiment/index.html?survey_id={{survey_id}}&condition=rating&return_url={{next_page_url}}
```

## 功能说明

### ① 商品信息区（弱刺激）
- 左图右文布局
- 极简文本，使用bullet points
- 包含：商品名、价格区间、核心参数、平均评分
- 无颜色强调、无推荐语、无AI相关字样

### ② 评分 & 评论摘要区（唯一操纵）
- 灰色卡片设计
- 标题："用户评论摘要"
- 根据URL参数显示不同条件：
  - `condition=rating`: 只显示星级和数量
  - `condition=aigrs_neutral`: AIGRS中立文本
  - `condition=aigrs_positive`: AIGRS正向文本

### ③ 操作按钮区
- 单一按钮："查看原始评论"
- 中性灰色设计
- 记录 `view_reviews` 行为

### ④ 原始评论区
- **初始状态**：完全隐藏
- **首次展开**：显示9条评论
- **评论结构**：星级、标题、2-3行正文
- **深度搜索**：底部"加载更多评论"按钮
  - 每次加载9条
  - 最多加载5次（共约45条评论）
  - 记录每次点击时间戳和滚动位置

### ⑤ 返回问卷按钮
- 按钮文案："完成浏览，返回问卷"
- 位于页面底部
- 不高亮、不闪烁
- 永远可点击

## 技术实现

- 纯HTML + CSS + JavaScript
- 无外部依赖
- 响应式设计
- 数据存储在localStorage（可通过URL参数或postMessage传回Credamo）

## 自定义说明

### 修改商品信息
编辑 `index.html` 中的商品信息区（`.product-section`）部分（约236-249行）。

### 修改评论内容
编辑 `allReviews` 数组（约314-355行），包含40条预设评论，可根据需要调整。

### 修改加载次数
修改 `maxLoads` 变量（约459行，默认5次）。

### 修改每次加载数量
修改 `reviewsPerLoad` 变量（约458行，默认9条）。

## 数据记录概览

实验数据会自动记录，包含以下主要字段：

- **问卷标识**：`survey_id`, `user_id`, `respondent_id`
- **实验条件**：`condition` (rating/aigrs_neutral/aigrs_positive)
- **时间戳**：页面加载、DOM加载、交互行为等时间点
- **交互行为**：查看评论、加载更多、返回问卷等操作
- **停留时间**：总停留时间（毫秒和秒）

详细的数据字段说明请查看 [数据字段说明.md](数据字段说明.md)

### 快速查看数据
在浏览器控制台（F12）中执行：
```javascript
const data = JSON.parse(localStorage.getItem('experimentData'));
console.log(data);
```

## 下一步

1. **部署到GitHub Pages** → 查看 [部署指南.md](部署指南.md)
2. **在Credamo中配置** → 查看 [Credamo配置指南.md](Credamo配置指南.md)
3. **了解数据字段** → 查看 [数据字段说明.md](数据字段说明.md)

## 注意事项

1. 所有行为数据都会记录时间戳（ISO 8601格式，精确到毫秒）
2. 评论内容混合了不同情绪和属性（续航、功能、舒适度）
3. 页面设计遵循中性原则，避免过度引导
4. 数据保存在localStorage，同时可通过URL参数或postMessage传回Credamo
5. GitHub Pages 部署后，URL是公开的，任何人都可以访问

