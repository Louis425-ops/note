

## 一、box model

每个HTML元素都可以看作是一个矩形的盒子，HTML的布局就是对每个盒子的大小数值、边距等进行排布

### components(从外到内)

1. **Content（内容区）** - 实际的文本和图像内容
2. **Padding（内边距）** - 内容和边框之间的空间
3. **Border（边框）** - 围绕内容和内边距的线条
4. **Margin（外边距）** - 元素边框与其他元素边框之间的空间

### Width & Height 的不同设置方式

| 写法        | 含义                                           |
| ----------- | ---------------------------------------------- |
| 数值        | 用绝对值数值定义（带单位）                     |
| 百分数      | 按照外层容器的长宽百分比地为元素分配长宽       |
| auto        | 按浏览器的默认分配长宽                         |
| max-content | 内容有多宽，盒子就有多宽，不会顾及父级盒子宽度 |
| min-content | 装得下盒子内单个最大内容的最小宽度             |
| fit-content | 跟max比较像，但会顾及父级盒子的宽度尽量撑开    |

**代码示例：**

```css
.web {
    height: 50px;
    width: 10%;
    border: solid blue;
}

.text {
    width: max-content; /* 只管自己，不管父级 */
}
```

### Padding（内边距）

- 定义元素边框与元素内容之间的空间
- 可分别设置四个方向：`padding-top`、`padding-left`、`padding-right`、`padding-bottom`

### Border（边框）

- box model的边框，可设置宽度、颜色和样式
- **重要特性**：上下左右四类边框连接处是一条45度的斜线
- `border-style`必须不为默认值`none`，否则border将一直不可见
- `border-radius`属性可以设置边框的圆角

**创建三角形技巧：**

```css
.triangle {
    border-bottom: 100px solid pink;
    border-left: 100px solid transparent;
    width: 100px;
    height: 0; /* 关键：将height设为0 */
}
```

### Margin（外边距）

- 定义元素边框与其他元素边框之间的空间
- 逻辑和用法与padding完全相同

### IE vs 标准盒模型

- **标准盒模型**：`width/height = content`，不包含border和padding
- **IE盒模型**：`width/height = content + padding + border`，包含border和padding
- 可通过`box-sizing`属性来定义使用哪种盒模型

------

## 二、CSS Selector

CSS选择器规定了编写的CSS规则会被应用到哪些HTML元素上，被选择的元素称为"选择器的对象"

### 基础选择器类型

#### 1. Universal Selector（通用选择器）

```css
* {
    /* 这里的CSS规则对所有html元素生效 */
}
```

#### 2. ID Selector（ID选择器）

```css
#NAME {
    /* 这里的CSS规则对id="NAME"的元素生效 */
}
```

**注意：ID属性必须是唯一的**

#### 3. Class Selector（类选择器）

```css
.example {
    /* 这里的CSS规则对class中包含"example"的元素生效 */
}
```

**类选择器的高级用法：**

- **并集**：`.never,.gonna` - 对包含never或gonna的元素生效
- **交集**：`.never.gonna.give` - 对同时包含never、gonna和give的元素生效
- **元素限定**：`p.snow.fall` - 对同时满足`<p>`标签且class包含snow和fall的元素生效

#### 4. Type Selector（元素选择器）

```css
p {
    /* 这里的CSS规则对所有p元素生效 */
}
```

#### 5. Attribute Selector（属性选择器）

```css
*[lang] {
    /* 对拥有lang属性的所有元素生效 */
}

p[class="never gonna"] {
    /* 对class属性完全等于"never gonna"的p元素生效 */
}
```

**属性选择器表达式：**

- `[attribute]` - 选择拥有指定属性的元素
- `[attribute=value]` - 选择属性值完全匹配的元素
- `[attribute~=value]` - 选择属性值包含指定单词的元素
- `[attribute*=value]` - 选择属性值包含指定子串的元素
- `[attribute^=value]` - 选择属性值以指定子串开头的元素
- `[attribute$=value]` - 选择属性值以指定子串结尾的元素

### 组合器(Combinator)

| 组合器名       | 表现形式            | 用途                                         |
| -------------- | ------------------- | -------------------------------------------- |
| 后代选择器     | `element1 element2` | 选择element1内的所有element2元素             |
| 子选择器       | `element1>element2` | 选择element1作为父元素的所有element2子元素   |
| 通用兄弟选择器 | `element1~element2` | 选择前面有element1元素的所有element2元素     |
| 相邻兄弟选择器 | `element1+element2` | 选择所有紧随在element1元素后面的element2元素 |

**示例：**

```css
.out_frame p {
    color: green; /* 后代选择器：选择.out_frame内所有的p元素 */
}

.out_frame>p {
    color: green; /* 子选择器：仅选择.out_frame的直接子元素p */
}

p~.out_frame {
    color: green; /* 兄弟选择器：选择p元素之后的.out_frame兄弟元素 */
}

p+h1 {
    color: green; /* 相邻兄弟选择器：选择紧跟在p元素后的h1元素 */
}
```

### 伪类和伪元素

- **伪类**：用单冒号(`:`)，指定元素的特殊状态，如`:hover`
- **伪元素**：用双冒号(`::`)，修改元素的部分内容状态

### 选择器优先级

```
!important > style属性 > ID选择器 > Class选择器 > Type选择器 > Universal选择器
```

**权重计算原则：**

- ID远大于class远大于type远大于universal
- 100个下一级也比不过一个上一级
- 同等级情况下，越后面写的选择器优先级更高

------

## 三、Fetch API

Fetch API是XMLHttpRequest的理想替代方案，更易上手，使用Promise而非回调函数，支持现代异步写法

### fetch()方法

#### 参数

1. **input**: string - 请求的URL字符串
2. (可选参数) **init**- 配置项对象，常用参数：
   - `method`: 请求方法（GET、POST等），默认GET
   - `headers`: HTTP请求头
   - `body`: 请求体（GET请求无法携带body）
   - `credentials`: 请求的credentials（如需跨域cookie，设为"include"）

#### 返回值：Response对象

**Response对象常用属性：**

- `status`: number - 后端返回的状态码
- `statusText`: string - 状态信息
- `ok`: boolean - 状态码在200-299时为true
- `headers`: Headers对象 - 返回头信息

**Response对象常用方法：**

- `json()`: 将返回内容当作JSON解析
- `blob()`: 将返回数据解析为blo
- `text()`: 将返回数据解析为纯文本

### e.g

```javascript
// 通用请求函数封装
const request = async (url, options) => {
    const response = await fetch(prefix + url, {
        ...options,
        credentials: "include", // cookie--跨域携带
    });
    
    if (response.ok) {
        const res = await response.json();
        if (res.code === 0) {
            return res.data;
        } else {
            // 错误处理...
        }
    } else {
        // 也是处理错误
    }
};

export const post = async (url, data) => {
    return await request(url, {
        method: "POST",
        body: data ? JSON.stringify(data) : undefined,
    });
};
```

### important

- Response对象实现了ReadableStream接口，数据只能异步读取
- 需要适当的错误处理机制
- 在实际项目中将频繁使用fetch API进行网络请求