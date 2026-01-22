# 商品浏览实验页面

这是一个单页面应用，用于商品浏览实验研究。所有功能使用原生JavaScript实现，无需复杂框架。

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
- **首次展开**：显示8-10条评论
- **评论结构**：星级、标题、2-3行正文
- **深度搜索**：底部"加载更多评论"按钮
  - 每次加载8-10条
  - 最多加载5次（共约45条评论）
  - 记录每次点击时间戳

### ⑤ 返回问卷按钮
- 按钮文案："完成浏览，返回问卷"
- 位于页面底部
- 不高亮、不闪烁
- 永远可点击

## 使用方法

### 基本使用
直接在浏览器中打开 `index.html` 即可。

### 实验条件设置
通过URL参数设置实验条件：

```
index.html?condition=rating              # 只显示星级和数量
index.html?condition=aigrs_neutral       # AIGRS中立版本
index.html?condition=aigrs_positive      # AIGRS正向版本
```

### 返回问卷设置
通过URL参数设置返回链接：

```
index.html?condition=rating&return_url=https://your-survey-url.com
```

## 数据记录

实验数据会自动记录到 `localStorage`，包含以下字段：

- `view_reviews`: 是否点击了"查看原始评论"
- `click_view_reviews`: 点击时间戳（ISO格式）
- `click_load_more`: 数组，记录每次"加载更多"的点击（次数和时间戳）
- `condition`: 实验条件（rating/aigrs_neutral/aigrs_positive）
- `return_timestamp`: 返回问卷的时间戳

### 获取数据
数据保存在浏览器的 `localStorage` 中，键名为 `experimentData`。可以通过以下方式获取：

```javascript
const data = JSON.parse(localStorage.getItem('experimentData'));
console.log(data);
```

## 技术实现

- 纯HTML + CSS + JavaScript
- 无外部依赖
- 响应式设计
- 数据存储在localStorage（可改为发送到服务器）

## 自定义说明

### 修改商品信息
编辑 `index.html` 中的商品信息区（`.product-section`）部分。

### 修改评论内容
编辑 `allReviews` 数组，包含40条预设评论，可根据需要调整。

### 修改加载次数
修改 `maxLoads` 变量（默认5次）。

### 修改每次加载数量
修改 `reviewsPerLoad` 变量（默认9条）。

## GitHub Pages 部署

### 快速部署步骤

1. **创建GitHub仓库**
   ```bash
   git init
   git add .
   git commit -m "Initial commit: 商品浏览实验页面"
   ```

2. **在GitHub上创建新仓库**
   - 访问 https://github.com/new
   - 创建新仓库（例如：`product-experiment`）
   - **不要**初始化README、.gitignore或license

3. **推送代码到GitHub**
   ```bash
   git remote add origin https://github.com/你的用户名/product-experiment.git
   git branch -M main
   git push -u origin main
   ```

4. **启用GitHub Pages**
   - 进入仓库的 Settings → Pages
   - Source 选择 `Deploy from a branch`
   - Branch 选择 `main`，文件夹选择 `/ (root)`
   - 点击 Save

5. **访问你的页面**
   - 几分钟后，你的页面将在以下URL可用：
   ```
   https://你的用户名.github.io/product-experiment/index.html
   ```

### 在Credamo中使用

部署完成后，在Credamo中使用以下链接：

```
https://你的用户名.github.io/product-experiment/index.html?condition=rating
https://你的用户名.github.io/product-experiment/index.html?condition=aigrs_neutral
https://你的用户名.github.io/product-experiment/index.html?condition=aigrs_positive
```

## 注意事项

1. 所有行为数据都会记录时间戳
2. 评论内容混合了不同情绪和属性（续航、功能、舒适度）
3. 页面设计遵循中性原则，避免过度引导
4. 数据保存在localStorage，如需发送到服务器，可修改返回按钮的事件处理
5. GitHub Pages 部署后，URL是公开的，任何人都可以访问

