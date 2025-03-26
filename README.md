# Filesystem MCP Server 文件系统MCP服务器

Node.js server implementing Model Context Protocol (MCP) for filesystem operations.
Node.js服务器实现了用于文件系统操作的模型上下文协议（MCP）。
注意：你必须指定至少一个允许服务器访问的目录路径作为参数。例如，要允许访问桌面目录：node src/index.js C:\Users\Qrh12\Desktop
## Features 特性
- Read/write files 读/写文件
- Create/list/delete directories 创建/列表/删除目录
- Move files/directories 移动文件/目录
- Search files 搜索文件
- Get file metadata 获取文件元数据

Note: The server will only allow operations within directories specified via args.
注意：服务器只允许在 args 指定的目录内进行操作。

## API

### Resources 资源
- `file://system`: File system operations interface
  `file://system` ：文件系统操作界面

### Tools 工具

#### read_file
- Read complete contents of a file 读取文件的完整内容
- Input: `path` (string)  `path`  （string）
- Reads complete file contents with UTF-8 encoding 使用UTF-8编码读取完整的文件内容

#### read_multiple_files
- Read multiple files simultaneously 同时读取多个文件
- Input: `paths` (string[]) 输入： `paths`  （string[]）
- Failed reads won't stop the entire operation 读取失败不会停止整个操作

#### write_file
- Create new file or overwrite existing (exercise caution with this) 创建新文件或覆盖现有文件（请谨慎使用）
- Inputs: 输入:
  - `path` (string): File location `path`  (string)：文件位置
  - `content` (string): File content `content`  (string)：文件内容

#### edit_file
- Make selective edits using advanced pattern matching and formatting 使用高级模式匹配和格式化进行选择性编辑
- Features: 特点:
  - Line-based and multi-line content matching 基于行和多行内容匹配
  - Whitespace normalization with indentation preservation 保留缩进的空白规范化
  - Fuzzy matching with confidence scoring 基于置信度评分的模糊匹配
  - Multiple simultaneous edits with correct positioning 多个同时编辑与正确的定位
  - Indentation style detection and preservation 检测和保存缩进风格
  - Git-style diff output with context git风格的带上下文的diff输出
  - Preview changes with dry run mode 演练模式下的预览更改
  - Failed match debugging with confidence scores 带有置信度分数的匹配调试失败
- Inputs: 输入:
  - `path` (string): File to edit `path`  (string)：要编辑的文件
  - `edits` (array): List of edit operations `edits`  (array)：编辑操作列表
    - `oldText` (string): Text to search for (can be substring) `oldText`  (string)：要搜索的文本（可以是子字符串）
    - `newText` (string): Text to replace with `newText` （字符串）：要替换的文本
  - `dryRun` (boolean): Preview changes without applying (default: false) `dryRun`  (boolean)：不应用更改预览（默认：false）
  - `options` (object): Optional formatting settings `options` （对象）：可选的格式化设置
    - `preserveIndentation` (boolean): Keep existing indentation (default: true) `preserveIndentation`  (boolean)：保留现有的缩进（默认值：true）
    - `normalizeWhitespace` (boolean): Normalize spaces while preserving structure (default: true) `normalizeWhitespace`  (boolean)：在保持结构不变的情况下规范化空格（默认值：true）
    - `partialMatch` (boolean): Enable fuzzy matching (default: true) `partialMatch`  (boolean)：启用模糊匹配（默认：true）
- Returns detailed diff and match information for dry runs, otherwise applies changes 返回演练的详细差异和匹配信息，否则应用更改
- Best Practice: Always use dryRun first to preview changes before applying them 最佳实践：在应用更改之前，一定要先使用dryRun进行预览

#### create_directory
- Create new directory or ensure it exists 创建新目录或确保它存在
- Input: `path` (string)  `path`  （string）
- Creates parent directories if needed 如果需要，创建父目录
- Succeeds silently if directory exists 如果目录存在，则静默成功

#### list_directory
- List directory contents with [FILE] or [DIR] prefixes 列出以[FILE]或[DIR]前缀的目录内容
- Input: `path` (string)  `path`  （string）

#### move_file
- Move or rename files and directories 移动或重命名文件和目录
- Inputs: 输入:
  - `source` (string) `source` (字符串)
  - `destination` (string) `destination` (字符串)
- Fails if destination exists 如果目标存在则失败

#### search_files
- Recursively search for files/directories 递归搜索文件/目录
- Inputs: 输入:
  - `path` (string): Starting directory `path`  (string)：起始目录
  - `pattern` (string): Search pattern `pattern` （字符串）：搜索模式
  - `excludePatterns` (string[]): Exclude any patterns. Glob formats are supported. `excludePatterns`  (string[])：排除任何模式。支持Glob格式。
- Case-insensitive matching 不区分大小写的匹配
- Returns full paths to matches 返回匹配项的完整路径

#### get_file_info
- Get detailed file/directory metadata 获取详细的文件/目录元数据
- Input: `path` (string)  `path`  （string）
- Returns: 返回:
  - Size 大小
  - Creation time 创建时间
  - Modified time 修改时间
  - Access time 访问时间
  - Type (file/directory) 类型(文件/目录)
  - Permissions 权限

#### list_allowed_directories
- List all directories the server is allowed to access 列出服务器允许访问的所有目录
- No input required 无需输入
- Returns: 返回:
  - Directories that this server can read/write from 此服务器可以从中读/写的目录

## Usage with Claude Desktop
使用Claude Desktop

Add this to your `claude_desktop_config.json`:
将其添加到 `claude_desktop_config.json` 中：

Note: you can provide sandboxed directories to the server by mounting them to /projects. Adding the ro flag will make the directory readonly by the server.
注意：您可以通过将它们挂载到 /projects 来向服务器提供沙盒目录。添加 ro 标志将使该目录被服务器只读。

### Docker 码头工人
Note: all directories must be mounted to /projects by default.
注意：所有目录默认必须挂载到 /projects 。

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--mount", "type=bind,src=/Users/username/Desktop,dst=/projects/Desktop",
        "--mount", "type=bind,src=/path/to/other/allowed/dir,dst=/projects/other/allowed/dir,ro",
        "--mount", "type=bind,src=/path/to/file.txt,dst=/projects/path/to/file.txt",
        "mcp/filesystem",
        "/projects"
      ]
    }
  }
}
```

### NPX
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/username/Desktop",
        "/path/to/other/allowed/dir"
      ]
    }
  }
}
```

## Build 构建

### Docker build: 码头工人构建:
```
docker build -t mcp/filesystem -f src/filesystem/Dockerfile .
```

## License 许可证
This MCP server is licensed under the MIT License. This means you are free to use, modify, and distribute the software, subject to the terms and conditions of the MIT License. For more details, please see the LICENSE file in the project repository.这个MCP服务器是根据MIT许可证授权的。这意味着您可以自由地使用、修改和发布软件，遵守MIT许可证的条款和条件。有关更多详细信息，请参阅项目存储库中的许可证文件。
这个MCP服务器是根据MIT许可证授权的。这意味着您可以自由地使用、修改和发布软件，遵守MIT许可证的条款和条件。有关更多详细信息，请参阅项目存储库中的许可证文件。

npm install -g ./filesystem
mcp-filesystem /path/to/allowed/directory

npx @modelcontextprotocol/server-filesystem /path/to/allowed/directory
