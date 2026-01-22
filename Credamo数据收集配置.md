# Credamo 数据收集配置指南

## 📋 目标
将问卷ID和用户网页交互数据关联，统一记录在一张表中。

## 🔗 在 Credamo 中嵌入的链接格式

### 基础格式
```
https://linhaiting11111jenny.github.io/product-experiment/index.html?survey_id={问卷ID}&condition={实验条件}&return_url={返回链接}
```

### 参数说明

| 参数名 | 必需 | 说明 | 示例 |
|--------|------|------|------|
| `survey_id` | ✅ 是 | 问卷ID（用于关联数据） | `survey_id=12345` |
| `condition` | ✅ 是 | 实验条件 | `condition=rating` / `condition=aigrs_neutral` / `condition=aigrs_positive` |
| `return_url` | ⚠️ 建议 | 返回问卷的链接（用于数据回传） | `return_url=https://credamo.com/survey/next` |
| `user_id` | ❌ 可选 | 用户ID（如果需要） | `user_id=user123` |
| `respondent_id` | ❌ 可选 | 受访者ID（如果需要） | `respondent_id=resp456` |

### 完整示例链接

```
https://linhaiting11111jenny.github.io/product-experiment/index.html?survey_id=12345&condition=rating&return_url=https://credamo.com/survey/12345/next
```

## 📊 数据字段说明

实验数据现在包含以下字段：

```javascript
{
  // 问卷和用户标识
  survey_id: "12345",              // 问卷ID（必需）
  user_id: "user123",              // 用户ID（可选）
  respondent_id: "resp456",        // 受访者ID（可选）
  
  // 实验条件
  condition: "rating",             // 实验条件
  
  // 交互行为数据
  view_reviews: true,              // 是否查看评论
  click_view_reviews: "2024-01-15T10:30:45.123Z",  // 查看评论时间
  click_load_more: [               // 加载更多记录
    { count: 1, timestamp: "2024-01-15T10:31:20.456Z" },
    { count: 2, timestamp: "2024-01-15T10:32:10.789Z" }
  ],
  
  // 时间戳
  page_load_time: "2024-01-15T10:30:00.000Z",      // 页面加载时间
  return_timestamp: "2024-01-15T10:35:00.000Z"     // 返回问卷时间
}
```

## 🔧 在 Credamo 中的配置步骤

### 方法一：使用 URL 参数传递（推荐）

#### 步骤 1：在 Credamo 中设置变量

1. 在问卷开始处，创建一个**隐藏变量**来存储问卷ID：
   - 变量名：`survey_id`
   - 类型：文本
   - 值：使用 Credamo 的系统变量（如 `{{survey_id}}` 或 `{{respondent_id}}`）

#### 步骤 2：构建嵌入链接

在需要嵌入实验页面的题目中，使用 Credamo 的**HTML组件**或**嵌入外部页面**功能：

```html
<iframe 
    src="https://linhaiting11111jenny.github.io/product-experiment/index.html?survey_id={{survey_id}}&condition={{condition}}&return_url={{next_page_url}}" 
    width="100%" 
    height="800px" 
    frameborder="0">
</iframe>
```

或者使用 Credamo 的链接跳转功能：
```
https://linhaiting11111jenny.github.io/product-experiment/index.html?survey_id={{survey_id}}&condition={{condition}}&return_url={{next_page_url}}
```

#### 步骤 3：接收数据（在下一个页面）

在实验页面后的下一个页面中，添加 JavaScript 代码来接收数据：

```javascript
// 在 Credamo 的 JavaScript 代码区域添加
(function() {
    // 从 URL 参数获取实验数据
    const urlParams = new URLSearchParams(window.location.search);
    const experimentDataStr = urlParams.get('experiment_data');
    
    if (experimentDataStr) {
        try {
            const experimentData = JSON.parse(decodeURIComponent(experimentDataStr));
            
            // 将数据保存到 Credamo 变量中
            // 注意：根据 Credamo 的实际 API 调整以下代码
            
            // 方法1：保存到隐藏变量
            credamo.setVariable('experiment_view_reviews', experimentData.view_reviews);
            credamo.setVariable('experiment_condition', experimentData.condition);
            credamo.setVariable('experiment_click_view_reviews', experimentData.click_view_reviews);
            credamo.setVariable('experiment_click_load_more_count', experimentData.click_load_more.length);
            credamo.setVariable('experiment_page_load_time', experimentData.page_load_time);
            credamo.setVariable('experiment_return_timestamp', experimentData.return_timestamp);
            
            // 方法2：保存完整 JSON（如果 Credamo 支持）
            credamo.setVariable('experiment_data_json', JSON.stringify(experimentData));
            
            console.log('实验数据已保存：', experimentData);
        } catch (e) {
            console.error('解析实验数据失败：', e);
        }
    }
})();
```

### 方法二：使用 postMessage 接收（如果 Credamo 支持）

如果 Credamo 支持接收 postMessage，可以在父页面添加监听：

```javascript
// 在 Credamo 的父页面 JavaScript 中添加
window.addEventListener('message', function(event) {
    // 验证来源（可选，提高安全性）
    // if (event.origin !== 'https://linhaiting11111jenny.github.io') return;
    
    if (event.data && event.data.type === 'experiment_complete') {
        const experimentData = event.data.data;
        
        // 保存到 Credamo 变量
        credamo.setVariable('experiment_view_reviews', experimentData.view_reviews);
        credamo.setVariable('experiment_condition', experimentData.condition);
        credamo.setVariable('experiment_data_json', JSON.stringify(experimentData));
        
        console.log('通过 postMessage 接收数据：', experimentData);
    }
});
```

## 📝 数据表结构建议

在 Credamo 中，建议创建以下变量来存储数据：

| 变量名 | 类型 | 说明 |
|--------|------|------|
| `survey_id` | 文本 | 问卷ID（用于关联） |
| `experiment_condition` | 文本 | 实验条件 |
| `experiment_view_reviews` | 布尔/文本 | 是否查看评论 |
| `experiment_click_view_reviews` | 文本 | 查看评论时间戳 |
| `experiment_load_more_count` | 数字 | 加载更多次数 |
| `experiment_page_load_time` | 文本 | 页面加载时间 |
| `experiment_return_timestamp` | 文本 | 返回问卷时间 |
| `experiment_data_json` | 文本 | 完整数据（JSON格式） |

## 🔄 完整流程示例

### 1. 问卷开始
```
页面1：欢迎页
  ↓
页面2：随机分配实验条件（rating / aigrs_neutral / aigrs_positive）
  ↓
页面3：嵌入实验页面
  URL: https://linhaiting11111jenny.github.io/product-experiment/index.html?survey_id={{survey_id}}&condition={{condition}}&return_url={{next_page_url}}
  ↓
页面4：接收数据并继续问卷
  JavaScript: 从 URL 参数获取 experiment_data，保存到变量
  ↓
页面5-N：后续问卷题目
  ↓
结束：所有数据（问卷答案 + 实验数据）统一导出
```

### 2. 数据导出

在 Credamo 中导出数据时，你会得到一张包含以下列的表格：

| survey_id | experiment_condition | experiment_view_reviews | experiment_click_view_reviews | ... | 问卷题目1 | 问卷题目2 | ... |
|-----------|---------------------|------------------------|------------------------------|-----|----------|----------|-----|
| 12345 | rating | true | 2024-01-15T10:30:45.123Z | ... | 答案1 | 答案2 | ... |
| 12346 | aigrs_neutral | false | null | ... | 答案1 | 答案2 | ... |

## ⚠️ 注意事项

1. **问卷ID必须传递**：确保 `survey_id` 参数正确传递，这是关联数据的关键
2. **URL 长度限制**：如果数据很大，URL 参数可能过长，建议使用 postMessage
3. **数据验证**：在接收数据时，添加错误处理和数据验证
4. **测试**：在正式发布前，充分测试数据传递和保存功能

## 🧪 测试步骤

1. **测试链接构建**：
   ```
   https://linhaiting11111jenny.github.io/product-experiment/index.html?survey_id=TEST123&condition=rating&return_url=https://example.com/test
   ```

2. **测试数据传递**：
   - 打开实验页面
   - 进行一些操作（查看评论、加载更多）
   - 点击"完成浏览，返回问卷"
   - 检查返回 URL 中是否包含 `experiment_data` 参数

3. **测试数据接收**：
   - 在返回页面检查 Credamo 变量是否正确保存
   - 检查控制台是否有错误信息

## 📞 需要帮助？

如果遇到问题：
1. 检查浏览器控制台的错误信息
2. 检查 URL 参数是否正确传递
3. 检查 Credamo 变量是否正确保存
4. 查看 `数据保存说明.md` 了解数据存储位置

