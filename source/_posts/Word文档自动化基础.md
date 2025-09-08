---
title: Word文档自动化基础
date: 2025-07-13 21:54:21
excerpt: 第三方库python-docx可以实现word的简单自动化，面对Excel数据需要按照一定格式大量转换为Word文档的需求时，掌握此项技术可以大大提高效率！
tags:
  - 办公自动化
categories:
  - [学习,Python第三方库]
---

# 1. 基本介绍

> `python-docx`是用于创建和修改 Microsoft Word（.docx）文件的 Python 第三方库。通过它，你能够以编程方式生成 Word 文档，添加文本、表格、图像、样式等元素

```python
from docx import Document
# 我们通过这一方法创建文档对象，后续操作皆以该对象为出发点
doc = Document()
# 此时是读取path路径现有的文档对象
doc = Document(path)
# 保存文档
doc.save(path)
```

word中常见的操作：添加标题（add_heading），添加正文（add_paragraph），添加表格（add_table）

# 2. 文本块Run

文本块Run对象是通过add_run()返回的对象，如下：

```python
head = doc.add_heading()
# 得到run对象，paragraph同理
run = head.add_run()

# 表格的run对象以单元格为单位
table = doc.add_table(rows=2, cols=2,style='Table Grid')
# 访问第一行第一列单元格的第一个段落的第一个 Run
cell = table.cell(0, 0)
# 得到单元格内容首段的run对象
run = cell.paragraphs[0].add_run()
```

通过Run对象，我们可以对文本进行多种定制化操作，例如字体样式，字体大小/颜色，对齐方式等等，每使用一次add_run()就在原先模块的基础上追加了新的文本

## 字体样式

> 确保文本是`add_run`添加的

```python
from docx.oxml.ns import qn
# 英文字体设置
run.font.name = '黑体'
# 中文字体设置
run._element.rPr.rFonts.set(qn('w:eastAsia'), '微软雅黑') 
```

## 字体大小

```python
from docx.shared import Pt
# 12磅字体
run.font.size = Pt(12)  
```

## 字体颜色

```python
from docx.shared import RGBColor
# 红色
run.font.color.rgb = RGBColor(255, 0, 0)  
```

## 粗体、斜体、下划线

```python
# 粗体
run.bold = True     
# 斜体
run.italic = True     
# 下划线
run.underline = True   
```

# 2. 标题相关操作

```python
# 添加标题，默认为一级标题，text推荐在run中添加，这样可以自定义样式
heading = doc.add_heading(text,level=1)
```

**标题对齐**

```python
from docx.enum.text import WD_ALIGN_PARAGRAPH
# 居中对齐
heading.alignment = WD_ALIGN_PARAGRAPH.CENTER  
```

**标题前后间距**

> 借助段落相关API实现

```python
# 段前间距12磅
heading.paragraph_format.space_before = Pt(12)  
# 段后间距6磅
heading.paragraph_format.space_after = Pt(6)   
```

# 3. 段落相关操作

```python
# 创建空段落，后续通过run添加内容
p = doc.add_paragraph()
```

## 段落对齐方式

```python
# 设置段落对齐方式，JUSTIFY是两端对齐
from docx.enum.text import WD_ALIGN_PARAGRAPH
p.alignment = WD_ALIGN_PARAGRAPH.CENTER  # 居中对齐
```

## 缩进设置

```python
from docx.shared import Cm
# 左缩进1厘米
p.paragraph_format.left_indent = Cm(1.0)  
# 首行缩进0.5厘米【负数为悬挂缩进】
p.paragraph_format.first_line_indent = Cm(0.5)  
```

## 间距

```python
# 1.5倍行距
p.paragraph_format.line_spacing_rule = WD_LINE_SPACING.ONE_POINT_FIVE 
# 设置固定行距为21磅
p.line_spacing_rule = WD_LINE_SPACING.EXACTLY 
p.paragraph_format.line_spacing = Pt(21)
# 首行缩进（两个中文字符大约42磅）
p.paragraph_format.first_line_indent = Pt(42)
# 段前间距6磅
p.paragraph_format.space_before = Pt(6)  
# 段后间距6磅
p.paragraph_format.space_after = Pt(6)   
```

## 分页

```python
# 段落前分页
p.paragraph_format.page_break_before = True  
```

# 4. 表格相关操作

```python
# 3行3列带网格的表格
table = doc.add_table(rows=3, cols=3, style='Table Grid')
```

## 表格尺寸

```python
# 行数
rows_count = len(table.rows)  
# 列数
cols_count = len(table.columns)  
```

## 添加新行

```python
# 表格底部添加新行
new_row = table.add_row().cells
```

## 访问单元格cell

```python
# 第1行第2列的单元格
cell = table.cell(0, 1)  
# 第2列第1行的单元格（可通过迭代访问所有单元格）
cell = table.columns[1].cells[0] 
# 第1行第2列的单元格（可通过迭代访问所有单元格）
cell = table.rows[0].cells[1]
```

## 合并单元格

```python
# 合并第1行的前两列
table.cell(0, 0).merge(table.cell(0, 1))
# 合并多行多列（2x2区域）
table.cell(0, 0).merge(table.cell(1, 1))  
```

## 单元格对齐

```python
from docx.enum.text import WD_ALIGN_PARAGRAPH, WD_ALIGN_VERTICAL
# 水平对齐（左、中、右）
cell.paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.CENTER 
# 垂直对齐（上、中、下）
cell.vertical_alignment = WD_ALIGN_VERTICAL.CENTER  
```

## 宽度与高度

> 高度通常是按行为单位

**宽度**

```python
# 表格总宽度15厘米
table.width = Cm(15)  
# 第1列宽度为3厘米
table.columns[0].width = Cm(3)  
# 第1个单元格宽度为5厘米
table.cell(0, 0).width = Cm(5)  
```

**高度**

```python
from docx.enum.text import WD_ROW_HEIGHT_RULE
# 设置第1行为固定高度（即使内容不足也保持此高度）
table.rows[0].height_rule = WD_ROW_HEIGHT_RULE.EXACTLY
table.rows[0].height = Cm(1)
# 清除固定高度设置
table.rows[0].height = None 
# 设置第1行自动适应内容高度
table.rows[0].height_rule = WD_ROW_HEIGHT_RULE.AUTO
```

## 单元格内容

```python
# 直接设置（会覆盖原有内容及格式）
cell.text = '新内容'  

# 获取单元格内第一个段落并修改文本内容
cell.paragraphs[0].text = "保留格式的文本" 

# 使用run定位到更加精准的位置
cell.paragraphs[0].runs[0].text = "第一个段落的第一个run"
```

# 5. 图片相关操作

> 通过段落加Run的方式添加图片，图片大小需要在添加时指定

```python
# 单元格通过段落对象获得run
run = cell.paragraphs[0].add_run()
# 利用run添加图片并固定宽度（高度自适应，也可以填固定值）
run.add_picture("./test/1.jpg", width=Cm(6.4))
# 设置图片居中
cell.paragraphs[0].alignment = WD_ALIGN_PARAGRAPH.CENTER
```

# 6 节Section

> 每个Word文档至少有一个节。如果文档没有手动分节，整个文档就是一个节，通过插入分节符（如 “下一页”、“连续”、“偶数页” 等），可以将文档分成多个节，各节可以有独立的页面设置。

## 页面设置相关属性

```python
from docx.enum.section import WD_ORIENT
from docx.shared import Cm
section = doc.sections[0]
# 修改页面方向为横向
section.orientation = WD_ORIENT.LANDSCAPE
# A4纸大小
section.page_width = Cm(29.7)
section.page_height = Cm(21.0)
# 页面边距设置
section.left_margin = Cm(2.5)    
section.right_margin = Cm(2.5)   
section.top_margin = Cm(3)     
section.bottom_margin = Cm(3)  
```

## 分节符

```py
from docx.enum.section import WD_SECTION
# 插入下一页分节符（新节从下一页开始）
doc.add_section(WD_SECTION.NEW_PAGE)
# 连续分节符会在当前位置开始新节，不强制分页
doc.add_section(WD_SECTION.CONTINUOUS)
```

