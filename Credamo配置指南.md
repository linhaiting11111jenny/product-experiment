# Credamo（见数）平台配置指南

本指南详细说明如何在Credamo平台中配置和使用商品浏览实验页面。

## 📋 目录

- [链接配置](#链接配置)
- [数据收集配置](#数据收集配置)
- [数据接收方法](#数据接收方法)
- [完整流程示例](#完整流程示例)

## 🔗 链接配置

### 三个实验条件对应的链接

在Credamo中，你需要配置 **3个不同的链接**，对应三个实验条件：

#### 1. Rating 条件（只显示星级和数量）
```
https://你的用户名.github.io/product-experiment/index.html?survey_id={{survey_id}}&condition=rating&return_url={{next_page_url}}
```

#### 2. AIGRS 中立条件
```
https://你的用户名.github.io/product-experiment/index.html?survey_id={{survey_id}}&condition=aigrs_neutral&return_url={{next_page_url}}
```

#### 3. AIGRS 正向条件
```
https://你的用户名.github.io/product-experiment/index.html?survey_id={{survey_id}}&condition=aigrs_positive&return_url={{next_page_url}}
```

### URL参数说明

| 参数名 | 必需 | 说明 | 示例 |
|--------|------|------|------|
| `survey_id` | ✅ 是 | 问卷ID（用于关联数据） | `survey_id={{survey_id}}` |
| `condition` | ✅ 是 | 实验条件 | `condition=rating` / `condition=aigrs_neutral` / `condition=aigrs_positive` |
| `return_url` | ⚠️ 建议 | 返回问卷的链接（用于数据回传） | `return_url={{next_page_url}}` |
| `user_id` | ❌ 可选 | 用户ID（如果需要） | `user_id={{user_id}}` |
| `respondent_id` | ❌ 可选 | 受访者ID（如果需要） | `respondent_id={{respondent_id}}` |

### 在Credamo中嵌入的方法

#### 方法一：使用iframe嵌入（推荐）

在Credamo的HTML组件中使用：

```html
<iframe 
    src="https://你的用户名.github.io/product-experiment/index.html?survey_id={{survey_id}}&condition={{condition}}&return_url={{next_page_url}}" 
    width="100%" 
    height="800px" 
    frameborder="0"
    style="border: none;">
</iframe>
```

#### 方法二：使用链接跳转

在Credamo中使用"跳转到外部链接"功能，直接使用完整URL。

## 📊 数据收集配置

### 目标
将问卷ID和用户网页交互数据关联，统一记录在一张表中。

### 数据传递方式

实验数据会通过以下方式传递回Credamo：

1. **postMessage方式**（如果Credamo支持iframe嵌入）
   - 数据会通过 `window.postMessage` 发送给父窗口
   - 消息格式：
     ```javascript
     {
         type: 'experiment_complete',
         data: {
             // 完整的实验数据对象
         }
     }
     ```

2. **URL参数方式**（如果设置了return_url）
   - 点击"完成浏览，返回问卷"后，会跳转到：
     ```
     https://credamo.com/your-survey/next-page?experiment_data={"view_reviews":true,...}
     ```
   - 在Credamo的下一个页面中，可以通过URL参数获取数据

3. **localStorage方式**（备用）
   - 数据也会保存在浏览器的localStorage中
   - 键名：`experimentData`

## 🔧 数据接收方法

### 方法一：通过URL参数获取（推荐）

如果使用 `return_url` 参数，在Credamo的下一个页面中添加JavaScript代码：

```javascript
// 在Credamo的JavaScript代码区域添加
(function() {
    // 从URL参数获取实验数据
    const urlParams = new URLSearchParams(window.location.search);
    const experimentDataStr = urlParams.get('experiment_data');
    
    if (experimentDataStr) {
        try {
            const experimentData = JSON.parse(decodeURIComponent(experimentDataStr));
            
            // 将数据保存到Credamo变量中
            // 注意：根据Credamo的实际API调整以下代码
            
            // 方法1：保存到隐藏变量
            credamo.setVariable('experiment_view_reviews', experimentData.view_reviews);
            credamo.setVariable('experiment_condition', experimentData.condition);
            credamo.setVariable('experiment_click_view_reviews', experimentData.click_view_reviews);
            credamo.setVariable('experiment_click_load_more_count', experimentData.click_load_more.length);
            credamo.setVariable('experiment_page_load_time', experimentData.page_load_time);
            credamo.setVariable('experiment_return_timestamp', experimentData.return_timestamp);
            credamo.setVariable('experiment_total_stay_time_seconds', experimentData.total_stay_time_seconds);
            
            // 方法2：保存完整JSON（如果Credamo支持）
            credamo.setVariable('experiment_data_json', JSON.stringify(experimentData));
            
            console.log('实验数据已保存：', experimentData);
        } catch (e) {
            console.error('解析实验数据失败：', e);
        }
    }
})();
```

### 方法二：通过postMessage接收

如果Credamo支持接收postMessage，可以在父页面添加监听：

```javascript
// 在Credamo的父页面JavaScript中添加
window.addEventListener('message', function(event) {
    // 验证来源（可选，提高安全性）
    // if (event.origin !== 'https://你的用户名.github.io') return;
    
    if (event.data && event.data.type === 'experiment_complete') {
        const experimentData = event.data.data;
        
        // 保存到Credamo变量
        credamo.setVariable('experiment_view_reviews', experimentData.view_reviews);
        credamo.setVariable('experiment_condition', experimentData.condition);
        credamo.setVariable('experiment_data_json', JSON.stringify(experimentData));
        
        console.log('通过postMessage接收数据：', experimentData);
    }
});
```

### 方法三：通过localStorage获取（备用）

如果以上方法都不可用，可以在Credamo的下一个页面中从localStorage读取：

```javascript
// 在Credamo的JavaScript代码中
const data = JSON.parse(localStorage.getItem('experimentData') || '{}');
if (data && data.survey_id) {
    // 保存到Credamo变量
    credamo.setVariable('experiment_data_json', JSON.stringify(data));
}
```

## 📝 Credamo变量配置建议

在Credamo中，建议创建以下变量来存储数据：

| 变量名 | 类型 | 说明 |
|--------|------|------|
| `survey_id` | 文本 | 问卷ID（用于关联） |
| `experiment_condition` | 文本 | 实验条件 |
| `experiment_view_reviews` | 布尔/文本 | 是否查看评论 |
| `experiment_click_view_reviews` | 文本 | 查看评论时间戳 |
| `experiment_load_more_count` | 数字 | 加载更多次数 |
| `experiment_page_load_time` | 文本 | 页面加载时间 |
| `experiment_return_timestamp` | 文本 | 返回问卷时间 |
| `experiment_total_stay_time_seconds` | 数字 | 总停留时间（秒） |
| `experiment_data_json` | 文本 | 完整数据（JSON格式） |

## 🔄 完整流程示例

### 1. 问卷流程设计

```
页面1：欢迎页
  ↓
页面2：随机分配实验条件（rating / aigrs_neutral / aigrs_positive）
  ↓
页面3：嵌入实验页面
  URL: https://你的用户名.github.io/product-experiment/index.html?survey_id={{survey_id}}&condition={{condition}}&return_url={{next_page_url}}
  ↓
页面4：接收数据并继续问卷
  JavaScript: 从URL参数获取experiment_data，保存到变量
  ↓
页面5-N：后续问卷题目
  ↓
结束：所有数据（问卷答案 + 实验数据）统一导出
```

### 2. 数据导出

在Credamo中导出数据时，你会得到一张包含以下列的表格：

| survey_id | experiment_condition | experiment_view_reviews | experiment_click_view_reviews | ... | 问卷题目1 | 问卷题目2 | ... |
|-----------|---------------------|------------------------|------------------------------|-----|----------|----------|-----|
| 12345 | rating | true | 2024-01-15T10:30:45.123Z | ... | 答案1 | 答案2 | ... |
| 12346 | aigrs_neutral | false | null | ... | 答案1 | 答案2 | ... |

## 🧪 测试步骤

### 1. 测试链接构建

```
https://你的用户名.github.io/product-experiment/index.html?survey_id=TEST123&condition=rating&return_url=https://example.com/test
```

### 2. 测试数据传递

- 打开实验页面
- 进行一些操作（查看评论、加载更多）
- 点击"完成浏览，返回问卷"
- 检查返回URL中是否包含 `experiment_data` 参数

### 3. 测试数据接收

- 在返回页面检查Credamo变量是否正确保存
- 检查控制台是否有错误信息
- 验证数据格式是否正确

## ⚠️ 注意事项

1. **问卷ID必须传递**：确保 `survey_id` 参数正确传递，这是关联数据的关键
2. **URL长度限制**：如果数据很大，URL参数可能过长，建议使用postMessage
3. **数据验证**：在接收数据时，添加错误处理和数据验证
4. **测试**：在正式发布前，充分测试数据传递和保存功能
5. **跨域问题**：如果部署在不同域名，确保Credamo允许iframe嵌入
6. **return_url参数**：确保 `{{next_page_url}}` 在Credamo中正确解析为有效的URL，否则会导致404错误

## ❗ 常见问题排查

### 问题1：点击"完成浏览，返回问卷"后出现404错误

**可能原因：**
- `{{next_page_url}}` 变量在Credamo中没有正确解析
- return_url参数传递了无效的URL或模板变量字符串

**解决方案：**

1. **检查URL参数是否正确解析**
   - 在浏览器开发者工具中检查iframe的src属性
   - 确认 `{{next_page_url}}` 是否被替换为实际URL
   - 如果看到 `return_url={{next_page_url}}` 这样的原始模板，说明变量未解析

2. **使用Credamo的正确变量**
   - 确认Credamo平台支持 `{{next_page_url}}` 变量
   - 如果不支持，尝试使用其他变量，如 `{{next_url}}` 或 `{{page_url}}`
   - 或者手动设置返回URL

3. **测试方法**
   - 在iframe的src中直接使用完整URL测试：
     ```html
     <iframe 
         src="https://你的用户名.github.io/product-experiment/index.html?survey_id=123&condition=rating&return_url=https://credamo.com/your-survey/next" 
         ...>
     ```
   - 如果直接URL可以工作，说明是变量解析问题

4. **替代方案：使用postMessage**
   - 如果URL跳转有问题，可以只使用postMessage传递数据
   - 在Credamo的下一个页面中通过JavaScript接收数据
   - 这样不需要return_url参数

## 📞 需要帮助？

如果遇到问题：

1. 检查浏览器控制台的错误信息
2. 检查URL参数是否正确传递
3. 检查Credamo变量是否正确保存
4. 查看 [数据字段说明.md](数据字段说明.md) 了解完整的数据结构

## 📚 相关文档

- [README.md](README.md) - 项目总览和快速开始
- [部署指南.md](部署指南.md) - GitHub Pages部署步骤
- [数据字段说明.md](数据字段说明.md) - 完整的数据字段说明

