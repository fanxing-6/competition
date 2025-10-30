## API文档格式要求

- 所有API文档应该保证md格式
- md文档第一标题为应该为当前文件下所包含API的前缀，以mindspore.nn.Cell.md为例：
```markdown
# mindspore.nn.Cell

mindspore.nn.Cell(auto_prefix=True, flags=None)

API说明、参数、样例等

mindspore.nn.Cell.add_flags(\**flags)

API说明、参数、样例等

mindspore.nn.Cell.add_flags_recursive(\**flags)

API说明、参数、样例等

......
```
mindspore.nn.Cell、mindspore.nn.Cell.add_flags、mindspore.nn.Cell.add_flags_recursive三个API拥有共同前缀mindspore.nn.Cell，与第一行大标题`# mindspore.nn.Cell`一致
- API文档放在一个文件夹下（可以有子文件夹递归）

## 数据处理

运行下面代码进行分段标识

```python
import pathlib

import sys

  

def process_md_file(file_path: pathlib.Path):

"""

处理单个 Markdown 文件，通过对第一个非空行

执行 split('# ', 1) 来获取 API 前缀。

"""

try:

# 打印相对路径，使其更清晰

relative_path = file_path.relative_to(pathlib.Path.cwd())

except ValueError:

relative_path = file_path

print(f"--- 正在处理: {relative_path} ---")

try:

# 使用 splitlines() 读取所有行，同时去除换行符

lines = file_path.read_text(encoding='utf-8').splitlines()

except Exception as e:

print(f"读取文件失败 {file_path.name}: {e}", file=sys.stderr)

return

  

if not lines:

print(f"警告: 文件 {file_path.name} 为空。跳过。")

return

  

# --- 您的新思路: 对第一个非空行直接使用 split ---

main_prefix = None

h1_line_index = -1

for i, line in enumerate(lines):

stripped_line = line.strip()

if stripped_line: # 找到第一个非空行

# 按照您的要求：直接使用 split 函数

parts = stripped_line.split('# ', 1)

if len(parts) > 1:

# 成功分割，'# ' 后的所有内容为 prefix

main_prefix = parts[1].strip()

h1_line_index = i

print(f"找到 Main API Prefix: {main_prefix}")

else:

# 分割失败，意味着第一个非空行不包含 '# '

# main_prefix 将保持为 None

pass

# 无论成功与否，都已处理完 "第一个非空行"，必须

break

# 检查是否成功找到了 prefix

if main_prefix is None:

print(f"警告: 在 {file_path.name} 中，第一个非空行不包含 '# ' 分隔符。跳过此文件。")

return

  

# --- 后续逻辑与之前相同 ---

processed_lines = []

first_api_block_found = False

  

# 1. 将 H1 标题（及之前的所有内容）添加到 processed_lines

processed_lines.extend(lines[:h1_line_index + 1])

# 2. 继续处理 H1 标题之后的行

start_index = h1_line_index + 1

  

for line in lines[start_index:]:

stripped_line = line.strip()

# 跳过空行

if not stripped_line:

processed_lines.append(line)

continue

  

# 检查是否为 API 定义的开始

# (以 prefix 开头, 且不是注释)

is_api_start = False

if stripped_line.startswith(main_prefix) and not stripped_line.startswith('#'):

# 检查是 'prefix.xxx' 'prefix(...)' 'prefix:'

# 或者是 'prefix' (例如属性)

if len(stripped_line) == len(main_prefix):

is_api_start = True # e.g., mindspore.nn.Cell.bprop_debug

else:

next_char = stripped_line[len(main_prefix)]

if next_char in ('.', '(', ':'):

is_api_start = True # e.g., mindspore.nn.Cell.add_flags or mindspore.nn.Cell(...)

  

if is_api_start:

if not first_api_block_found:

# 这是第一个 API 块 (例如 mindspore.nn.Cell(...))

first_api_block_found = True

processed_lines.append(line)

else:

# 这是后续的 API 块

processed_lines.append("") # 添加一个空行

processed_lines.append("---END---")

processed_lines.append("") # 添加一个空行

processed_lines.append(line)

else:

# 这不是 API 的开头，只是普通内容行

processed_lines.append(line)

  

# 3. 在文件末尾添加最后一个分隔符（如果至少处理过一个 API 块）

if first_api_block_found:

processed_lines.append("")

processed_lines.append("---END---")

processed_lines.append("") # 确保文件以新行结束

  

# 4. 将处理后的内容写回文件

try:

output_content = "\n".join(processed_lines)

file_path.write_text(output_content, encoding='utf-8')

print(f"--- 完成: {file_path.name} ---")

except Exception as e:

print(f"写入文件失败 {file_path.name}: {e}", file=sys.stderr)

  
  

def main():

# 目标文件夹路径

target_dir = pathlib.Path("填入目标文件夹路径")

  

if not target_dir.is_dir():

print(f"错误: 目录不存在: {target_dir}", file=sys.stderr)

return

  

# 使用 rglob("*.md") 来递归搜索所有子文件夹中的 .md 文件

print(f"正在 {target_dir} 及其子目录中递归搜索 .md 文件...")

md_files = list(target_dir.rglob("*.md"))

  

if not md_files:

print(f"在 {target_dir} 及其子目录中未找到 .md 文件。")

return

  

print(f"找到了 {len(md_files)} 个 .md 文件。开始处理...")

  

for file_path in md_files:

process_md_file(file_path)

  

print("\n所有文件处理完毕。")

  

if __name__ == "__main__":

main()
```



