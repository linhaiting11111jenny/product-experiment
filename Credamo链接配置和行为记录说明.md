# Credamo 链接配置和行为记录说明

## 📋 在 Credamo 上需要配置的链接

### 三个实验条件对应的链接

在 Credamo 中，你需要配置 **3个不同的链接**，对应三个实验条件：

#### 1. Rating 条件（只显示星级和数量）
```
https://linhaiting11111jenny.github.io/product-experiment/index.html?survey_id={{survey_id}}&condition=rating&return_url={{next_page_url}}
```

#### 2. AIGRS 中立条件
```
https://linhaiting11111jenny.github.io/product-experiment/index.html?survey_id={{survey_id}}&condition=aigrs_neutral&return_url={{next_page_url}}
```

#### 3. AIGRS 正向条件
```
https://linhaiting11111jenny.github.io/product-experiment/index.html?survey_id={{survey_id}}&condition=aigrs_positive&return_url={{next_page_url}}
```

### 参数说明

| 参数 | 说明 | 示例值 |
|------|------|--------|
| `survey_id` | 问卷ID（Credamo变量） | `{{survey_id}}` |
| `condition` | 实验条件（固定值） | `rating` / `aigrs_neutral` / `aigrs_positive` |
| `return_url` | 返回链接（Credamo变量） | `{{next_page_url}}` |

## 🔧 AIGRS.html 修改对应关系

### 不同链接对应在 AIGRS.html 的哪个部分修改

**注意**：`AIGRS.html` 是一个简化版本，主要用于参考。实际使用的是 `index.html`。

如果你想修改 `AIGRS.html`，以下是各部分的对应关系：

#### ① 商品信息区（弱刺激）
**位置**：`AIGRS.html` 第 84-98 行
```html
<section class="product-section">
    <div class="product-image">...</div>
    <div class="product-info">
        <h2>X-Series 无线降噪耳机</h2>
        <p class="price">¥899 - ¥1,099</p>
        <ul class="attr-list">...</ul>
    </div>
</section>
```
**修改内容**：商品名称、价格、属性列表

#### ② 评分 & 评论摘要区（唯一操纵）
**位置**：`AIGRS.html` 第 100-105 行
```html
<section class="summary-card">
    <div class="summary-title">用户评论摘要</div>
    <div id="summary-text" class="summary-content">
        <!-- 这里的内容会根据 condition 参数变化 -->
    </div>
</section>
```
**修改内容**：根据 `condition` 参数显示不同内容
- `rating`: 显示星级和数量
- `aigrs_neutral`: 显示中立AIGRS文本
- `aigrs_positive`: 显示正向AIGRS文本

**在 index.html 中的实现**：第 337-365 行的 `initSummary()` 函数

#### ③ 操作按钮区
**位置**：`AIGRS.html` 第 107-109 行
```html
<div class="action-area">
    <button id="view-reviews-btn" class="btn-neutral">查看原始评论</button>
</div>
```
**修改内容**：按钮文案、样式

#### ④ 原始评论区
**位置**：`AIGRS.html` 第 111-117 行
```html
<div id="review-module">
    <div id="review-list"></div>
    <button id="load-more-btn">加载更多评论</button>
</div>
```
**修改内容**：评论数据、加载逻辑

#### ⑤ 返回问卷按钮
**位置**：`AIGRS.html` 第 119-121 行
```html
<button id="complete-btn">完成浏览，返回问卷</button>
```
**修改内容**：按钮文案、返回逻辑

## 📊 现有实验记录的行为

### ✅ 已记录的行为（index.html）

#### 1. 时间戳记录
- ✅ **link_click_time**: 点击链接进入页面的时间戳
- ✅ **page_load_time**: 页面开始加载的时间戳
- ✅ **dom_content_loaded_time**: DOM内容加载完成的时间戳
- ✅ **page_load_complete_time**: 页面完全加载完成的时间戳

#### 2. 交互行为记录
- ✅ **view_reviews**: 是否点击了"查看原始评论"（布尔值）
- ✅ **click_view_reviews**: 点击"查看原始评论"的时间戳
- ✅ **reviews_expanded_time**: 评论展开完成的时间戳

#### 3. 加载更多行为记录
- ✅ **click_load_more**: 每次点击"加载更多评论"的详细记录数组
  - `count`: 第几次点击
  - `timestamp`: 点击时间戳
  - `before_scroll_position`: 点击前的滚动位置
  - `after_scroll_position`: 加载完成后的滚动位置
  - `load_complete_time`: 加载完成的时间戳
- ✅ **total_load_more_clicks**: 总共点击"加载更多"的次数

#### 4. 页面停留时间
- ✅ **return_click_time**: 点击"完成浏览，返回问卷"的时间戳
- ✅ **return_timestamp**: 返回时间戳（同 return_click_time）
- ✅ **total_stay_time_ms**: 总停留时间（毫秒）
- ✅ **total_stay_time_seconds**: 总停留时间（秒）

#### 5. 问卷和用户标识
- ✅ **survey_id**: 问卷ID
- ✅ **user_id**: 用户ID（可选）
- ✅ **respondent_id**: 受访者ID（可选）
- ✅ **condition**: 实验条件

#### 6. 其他记录
- ✅ **page_leave_time**: 离开页面的时间戳（如果可检测）

### 📝 完整数据字段列表

```javascript
{
  // 问卷和用户标识
  survey_id: "12345",
  user_id: "user123",           // 可选
  respondent_id: "resp456",     // 可选
  
  // 实验条件
  condition: "rating",           // "rating" / "aigrs_neutral" / "aigrs_positive"
  
  // 时间戳记录
  link_click_time: "2024-01-15T10:30:00.000Z",           // 点击链接时间
  page_load_time: "2024-01-15T10:30:00.100Z",           // 页面开始加载
  dom_content_loaded_time: "2024-01-15T10:30:00.500Z",  // DOM加载完成
  page_load_complete_time: "2024-01-15T10:30:01.000Z",  // 页面完全加载
  
  // 交互行为
  view_reviews: true,                                    // 是否查看评论
  click_view_reviews: "2024-01-15T10:31:00.000Z",       // 点击查看评论时间
  reviews_expanded_time: "2024-01-15T10:31:00.100Z",     // 评论展开完成时间
  
  // 加载更多行为
  click_load_more: [
    {
      count: 1,
      timestamp: "2024-01-15T10:32:00.000Z",
      before_scroll_position: 500,
      after_scroll_position: 1200,
      load_complete_time: "2024-01-15T10:32:00.150Z"
    },
    {
      count: 2,
      timestamp: "2024-01-15T10:33:00.000Z",
      before_scroll_position: 1200,
      after_scroll_position: 2000,
      load_complete_time: "2024-01-15T10:33:00.120Z"
    }
  ],
  total_load_more_clicks: 2,
  
  // 页面停留时间
  return_click_time: "2024-01-15T10:35:00.000Z",        // 点击返回时间
  return_timestamp: "2024-01-15T10:35:00.000Z",          // 返回时间戳
  total_stay_time_ms: 300000,                           // 总停留时间（毫秒）
  total_stay_time_seconds: 300,                         // 总停留时间（秒）
  
  // 其他
  page_leave_time: "2024-01-15T10:35:00.100Z"           // 离开页面时间（如果可检测）
}
```

## 🎯 行为记录时间线

```
时间轴示例：
10:30:00.000 - link_click_time (点击链接)
10:30:00.100 - page_load_time (页面开始加载)
10:30:00.500 - dom_content_loaded_time (DOM加载完成)
10:30:01.000 - page_load_complete_time (页面完全加载)
10:31:00.000 - click_view_reviews (点击查看评论)
10:31:00.100 - reviews_expanded_time (评论展开完成)
10:32:00.000 - click_load_more[0] (第1次加载更多)
10:32:00.150 - click_load_more[0].load_complete_time (第1次加载完成)
10:33:00.000 - click_load_more[1] (第2次加载更多)
10:33:00.120 - click_load_more[1].load_complete_time (第2次加载完成)
10:35:00.000 - return_click_time (点击返回)
10:35:00.100 - page_leave_time (离开页面)
```

## 📋 数据导出表格结构

在 Credamo 中导出数据时，建议的表格列：

| survey_id | condition | link_click_time | page_load_time | page_load_complete_time | view_reviews | click_view_reviews | reviews_expanded_time | total_load_more_clicks | return_click_time | total_stay_time_seconds | ... |
|-----------|-----------|-----------------|----------------|-------------------------|--------------|---------------------|----------------------|------------------------|-------------------|-------------------------|-----|
| 12345 | rating | 2024-01-15T10:30:00.000Z | 2024-01-15T10:30:00.100Z | 2024-01-15T10:30:01.000Z | true | 2024-01-15T10:31:00.000Z | 2024-01-15T10:31:00.100Z | 2 | 2024-01-15T10:35:00.000Z | 300 | ... |

## 🔍 如何验证行为记录

### 方法1：浏览器控制台
```javascript
// 在实验页面打开控制台（F12），执行：
const data = JSON.parse(localStorage.getItem('experimentData'));
console.log(data);
```

### 方法2：查看网络请求
如果数据通过 URL 参数传递，可以在返回页面的 URL 中查看 `experiment_data` 参数。

### 方法3：Credamo 变量检查
在 Credamo 的下一个页面，检查保存的变量值。

## ⚠️ 注意事项

1. **所有时间戳使用 ISO 8601 格式**：便于后续分析和处理
2. **时间精度**：时间戳精确到毫秒
3. **数据完整性**：即使被试不进行任何操作，也会记录页面加载相关的时间戳
4. **数据关联**：所有数据都通过 `survey_id` 与问卷答案关联

## 📞 需要更多行为记录？

如果需要记录其他行为（如滚动深度、鼠标移动、页面可见性变化等），可以进一步扩展代码。告诉我你需要记录哪些行为即可。

