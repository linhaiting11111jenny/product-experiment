# Credamo（见数）平台嵌入使用说明

## 方案一：通过URL嵌入（推荐）

### 步骤1：部署HTML文件
将 `index.html` 文件部署到一个可公开访问的URL，例如：
- GitHub Pages
- Netlify
- Vercel
- 自己的服务器

**示例部署URL：**
```
https://your-domain.com/experiment/index.html
```

### 步骤2：在Credamo中嵌入

在Credamo问卷编辑器中，找到"嵌入外部页面"或"HTML组件"功能，使用以下格式的URL：

#### 基础URL（默认rating条件）
```
https://your-domain.com/experiment/index.html
```

#### 带实验条件的URL
```
https://your-domain.com/experiment/index.html?condition=rating
https://your-domain.com/experiment/index.html?condition=aigrs_neutral
https://your-domain.com/experiment/index.html?condition=aigrs_positive
```

#### 带返回链接的URL（数据会通过URL参数传回）
```
https://your-domain.com/experiment/index.html?condition=rating&return_url=https://credamo.com/your-survey/next-page
```

### 步骤3：接收数据

实验数据会通过以下方式传递：

1. **postMessage方式**（如果Credamo支持）
   - 数据会通过 `window.postMessage` 发送给父窗口
   - 消息格式：
     ```javascript
     {
         type: 'experiment_complete',
         data: {
             view_reviews: true/false,
             click_view_reviews: "2024-01-01T12:00:00.000Z",
             click_load_more: [...],
             condition: "rating",
             return_timestamp: "2024-01-01T12:05:00.000Z",
             page_load_time: "2024-01-01T12:00:00.000Z"
         }
     }
     ```

2. **URL参数方式**（如果设置了return_url）
   - 点击"完成浏览，返回问卷"后，会跳转到：
     ```
     https://credamo.com/your-survey/next-page?experiment_data={"view_reviews":true,...}
     ```
   - 在Credamo的下一个页面中，可以通过URL参数获取数据

## 方案二：内联HTML代码（如果Credamo支持）

如果Credamo支持直接嵌入HTML代码，可以将 `index.html` 的 `<body>` 和 `<script>` 部分的内容复制到Credamo的HTML组件中。

**注意：** 需要确保CSS样式也一并复制。

## 方案三：通过iframe嵌入

如果Credamo支持iframe嵌入，可以使用：

```html
<iframe 
    src="https://your-domain.com/experiment/index.html?condition=rating" 
    width="100%" 
    height="800px" 
    frameborder="0"
    style="border: none;">
</iframe>
```

## 数据字段说明

实验数据包含以下字段：

| 字段名 | 类型 | 说明 |
|--------|------|------|
| `view_reviews` | Boolean | 是否点击了"查看原始评论"按钮 |
| `click_view_reviews` | String (ISO时间戳) | 点击"查看原始评论"的时间 |
| `click_load_more` | Array | 每次点击"加载更多评论"的记录数组 |
| `condition` | String | 实验条件：`rating` / `aigrs_neutral` / `aigrs_positive` |
| `return_timestamp` | String (ISO时间戳) | 点击"完成浏览，返回问卷"的时间 |
| `page_load_time` | String (ISO时间戳) | 页面首次加载的时间 |

### click_load_more 数组格式
```javascript
[
    { count: 1, timestamp: "2024-01-01T12:01:00.000Z" },
    { count: 2, timestamp: "2024-01-01T12:02:00.000Z" },
    ...
]
```

## 在Credamo中获取数据的方法

### 方法1：通过URL参数获取
如果使用 `return_url` 参数，在Credamo的下一个页面中：

```javascript
// 在Credamo的JavaScript代码中
const urlParams = new URLSearchParams(window.location.search);
const experimentData = JSON.parse(urlParams.get('experiment_data') || '{}');
console.log(experimentData);
```

### 方法2：通过postMessage监听
如果Credamo支持接收postMessage：

```javascript
// 在Credamo的父页面中
window.addEventListener('message', function(event) {
    if (event.data && event.data.type === 'experiment_complete') {
        const experimentData = event.data.data;
        console.log('实验数据：', experimentData);
        // 将数据保存到Credamo的变量中
    }
});
```

### 方法3：通过localStorage获取
实验数据也会保存在浏览器的localStorage中，键名为 `experimentData`：

```javascript
const data = JSON.parse(localStorage.getItem('experimentData') || '{}');
```

## 快速测试

1. **本地测试**：直接在浏览器中打开 `index.html`
2. **测试不同条件**：
   - `index.html?condition=rating`
   - `index.html?condition=aigrs_neutral`
   - `index.html?condition=aigrs_positive`
3. **测试数据传递**：打开浏览器控制台，查看localStorage中的 `experimentData`

## 注意事项

1. **跨域问题**：如果部署在不同域名，确保Credamo允许iframe嵌入（X-Frame-Options设置）
2. **数据格式**：所有时间戳使用ISO 8601格式
3. **数据完整性**：即使被试不点击任何按钮，`page_load_time` 也会被记录
4. **条件随机化**：建议在Credamo中通过随机化功能分配不同的 `condition` 参数

## 推荐配置

在Credamo问卷中，建议这样设置：

1. **随机化条件**：使用Credamo的随机化功能，将被试分配到三个条件之一
2. **URL构建**：根据随机化结果，构建对应的URL
3. **数据收集**：在下一个页面中接收并保存实验数据
4. **数据导出**：实验结束后，从Credamo导出数据，包含所有行为记录

## 示例Credamo问卷流程

```
开始页面
  ↓
随机化分配条件（rating / aigrs_neutral / aigrs_positive）
  ↓
嵌入实验页面（index.html?condition=xxx）
  ↓
被试浏览和操作
  ↓
点击"完成浏览，返回问卷"
  ↓
数据传递到下一个页面
  ↓
后续问卷题目
  ↓
结束
```

