# Open Pubmed Deep Research

**Open Pubmed Deep Research（OPDR）** 是一个面向生物医学文献综述场景的桌面应用。  
它把 **PubMed 检索、全文获取、PDF 转 Markdown、向量化、RAG 写作、导出成品** 串成了一条完整流水线。

适合的使用场景：

- 医学与生命科学综述写作
- 研究课题的快速文献摸底与框架搭建
- 需要引用可追溯、便于人工复核的长文生成任务

这个应用的定位不是“替你一键发论文”，而是把综述写作里最耗时、最机械的部分自动化，并尽可能保留 **PMID 级别的出处追溯能力**。

## 核心特点

- **一站式流程**：从 PubMed 检索到 Word / HTML 导出，无需在多个工具之间来回切换。
- **本地运行，本地保存**：应用本体运行在本地，工作区数据保存在本地目录。
- **可接本地或云端模型**：LLM 使用 OpenAI 兼容接口，Embedding 使用兼容当前请求格式的接口。
- **可追溯写作**：生成内容保留 PMID 引用，HTML 导出可点击查看对应素材上下文。
- **下载失败可降级继续**：拿不到 PDF 时，仍可使用摘要 `.md` 进入后续向量化与写作流程。
- **适合长篇综述**：支持分层框架、分章节写作、失败片段重试和最终整合导出。

## 软件界面

<p align="center">
  <img src="https://github.com/XinYu-pumch/OPDR/blob/main/gui.png" alt="软件界面" width="100%">
</p>

<p align="center"><em>工作区、设置面板与实时日志集中在一个桌面应用中。</em></p>

## 实现原理

应用内部采用的是一条面向综述写作的自动化管线：

1. 从 PubMed 按关键词、相关性或时间顺序检索文献。
2. 尝试通过 Europe PMC 或 Sci-Hub 获取全文 PDF；若无法获取，则退化为摘要文本。
3. 将 PDF 批量转为 Markdown，并做基础清理。
4. 以段落为单位切块，写入本地 ChromaDB 向量数据库。
5. 基于全部摘要生成分层综述框架。
6. 按章节做 RAG 检索与填充，生成可导出的完整综述。

<p align="center">
  <img src="https://github.com/XinYu-pumch/OPDR/blob/main/flow_chart.png" alt="实现原理" width="100%">
</p>

## 生成效果

导出的 HTML 综述支持目录导航，并可点击正文中的 PMID 查看该引用在当前章节中对应的素材上下文，便于人工核查。

<p align="center">
  <img src="https://github.com/XinYu-pumch/OPDR/blob/main/review.png" alt="综述展示" width="100%">
</p>

完整示例可下载后查看（https://github.com/XinYu-pumch/OPDR/blob/main/Igg4RD.html）

## 支持平台

- 当前发行版：`macOS` + `Apple Silicon`
- 暂不支持：`Intel Mac`
- Windows 版本：计划中

## 下载与安装

面向普通用户时，建议通过仓库的 `Releases` 页面获取压缩包（解压得到app文件）。

典型安装流程：

1. 下载最新的 `Open Pubmed Deep Research.zip`
2. 解压得到Open Pubmed Deep Research.app（可移动到“应用程序中”）
3. 启动应用
4. 首次启动时选择**工作区目录**
备注：若打开失败，打开终端，输入"sudo xattr -rd com.apple.quarantine"，然后将应用拖入该代码后，得到类似“sudo xattr -rd com.apple.quarantine /Applications/应用名称.app”，输入开机密码以清除隔离属性

说明：

- 工作区会存放你的集合数据、XML、PDF、Markdown、向量库和最终导出文件。
- 应用不会把这些数据写入 app bundle。

## 首次配置

首次进入应用后，建议先完成设置面板中的两项配置：

### 1. 工作区目录

- 选择一个本地文件夹作为工作区
- 应用会在该目录下创建 `content/` 用于保存研究集合

### 2. 模型配置

#### LLM

- 目前按代码实现，LLM 接口需要兼容 OpenAI 风格
- `Base URL` 建议填写到 `/v1`
- 推荐使用长上下文模型，以提升框架生成和章节写作效果
  - 推荐模型：gemini2.5 pro、gemini3.0pro、gemini3.1pro
  - 推荐中转服务商：https://poloai.top（需gmail注册）

#### Embedding

- 用于向量化和 RAG 检索
- 仓库默认示例为 `SiliconFlow` 的 embedding 接口
- 默认示例模型是 `Pro/BAAI/bge-m3`
  - 推荐“硅基流动（SiliconFlow）”的嵌入模型（https://docs.siliconflow.cn/cn/userguide/introduction）

使用建议：

- 保存配置后，再开始正式工作流
- 长任务运行过程中，尽量不要切回设置页，以免打断当前执行中的流程

## 使用流程

### 1. 创建集合

输入集合名称，例如 `IL6` 或 `lung_cancer_2024`。  
应用会在本地创建目录：

```text
{workspace}/content/{collection_name}/
```

后续该集合的 XML、PDF、Markdown、向量库和导出文件都会存放在这里。

### 2. 检索 PubMed 文献

- 输入 PubMed 检索式
- 支持按相关性或时间排序
- 支持高级日期筛选
- 如果你有多个检索式，可以多次执行检索，再统一点击“合并结果”

执行后会在集合目录中生成若干 XML 片段；合并后会得到：

- `pubmed_results.xml`

建议：

- 合并后的总文献量尽量控制在约 `250` 篇以内
- 文献太多时，综述框架质量和后续写作稳定性通常会下降

### 3. 生成综述框架

- 输入综述主题
- 可额外填写自定义撰写要求
- 点击生成后，应用会基于摘要合集调用 LLM 生成多级标题框架

输出文件：

- `prompt.txt`
- `reveiw_framework.csv`

说明：

- 当前代码里文件名拼写就是 `reveiw_framework.csv`（错别词，暂不修正）
- 如果结果不满意，可以重新生成，或手动编辑该 CSV
- 手动编辑时请保持列结构及列名不变

### 4. 下载全文

限速步骤。

下载策略：

- 有 `PMCID` 的文献优先尝试从PMC 获取 PDF
- 其他文献尝试通过 Sci-Hub 镜像下载
- 如果获取不到 PDF，则自动生成摘要 `.md` 作为兜底材料

下载所得：

- `*.pdf` （以pmid命名的文献pdf原文）
- `*.md` （以pmid命名的文献md摘要）
- `literature_without_pdf.txt` （未获取到原文pdf的文献pmid list）
- `downloaded_literature.zip` （所有下载的文献原文或摘要打包）

### 5. Markdown 转化

点击“Markdown 转化”后，应用会批量处理 PDF 并转换为 Markdown。  
`Workers` 建议参考你的 CPU 核心数设置，默认值为 `8`。

输出文件：

- PDF 对应的 `*.md`

### 6. 向量化入库

应用会对集合目录中的 Markdown 做清洗、切块和向量化，并写入本地 ChromaDB。

向量库目录位于：

```text
{workspace}/content/{collection_name}/{collection_name}/
```

### 7. 内容撰写

点击“开始/继续撰写”后，应用会：

1. 根据框架拆分章节任务
2. 对每个章节执行向量检索
3. 把检索到的文本块作为素材传给 LLM
4. 生成章节 JSON 结果

两个常用参数：

- `RAG Top-K`
  每个论点检索多少条素材。默认 `7`。
- `Concurrency`
  同时向 LLM 发起多少个章节请求。默认 `3`。

说明：

- `Top-K` 越大，理论上信息覆盖更充分，但也会增加模型处理负担
- 如果有章节失败，可以点击“重试失败项”重新生成

输出文件：

- `review_parts/section_*.json`
- `review_parts/prompt_section_*.txt`

### 8. 导出成品

支持两种导出：

- `HTML`：交互式综述，正文中的 PMID 可点击查看来源素材
- `Word`：便于修改和继续编辑的 `.docx` 文件

输出文件：

- `{collection_name}_Review.html`
- `{collection_name}_Review.docx`

## 常见输出文件一览

每个集合目录中通常会看到以下文件：

- `pubmed_results.xml`
  PubMed 合并后的 XML 元数据
- `abstract_combined.txt`
  摘要合集
- `prompt.txt`
  框架生成时实际发送给 LLM 的提示词
- `reveiw_framework.csv`
  综述框架 CSV
- `*.pdf`
  下载到的全文 PDF
- `*.md`
  原文 Markdown，或下载失败时保留的摘要 Markdown
- `downloaded_literature.zip`
  当前集合的 PDF / MD 打包文件
- `literature_without_pdf.txt`
  未成功获取 PDF 的 PMID 列表
- `review_parts/section_*.json`
  分章节生成结果
- `review_parts/prompt_section_*.txt`
  章节写作提示词
- `{collection_name}_Review.docx`
  Word 导出文件
- `{collection_name}_Review.html`
  交互式 HTML 导出文件

## 使用建议

- PubMed 检索式质量会直接影响后续框架和正文质量。
- 框架生成前，先尽量把检索结果清理到一个合理规模。
- 如果某些文献没有 PDF，不代表这篇文献完全无法参与写作；摘要 `.md` 仍可继续进入后续流程。
- 生成结果依赖你配置的 LLM、Embedding、检索规模和框架质量，正式使用前仍建议人工复核。



## 已知限制

- 全文可获取性取决于目标文献是否在 Europe PMC / PMC 中可用，以及目标下载源的可访问性。
- 大规模 PDF 转换、Embedding 和章节写作耗时会比较长。
- 当前发行重点是 macOS Apple Silicon。

## 合规说明

本项目涉及对 PubMed、Europe PMC、第三方模型接口以及部分全文获取渠道的调用逻辑。  
在公开分发和实际使用前，请根据你的目标地区、机构规范和服务条款，自行确认版权、网络访问与合规要求。

## 项目目标

Open Pubmed Deep Research 的目标不是替代研究者判断，而是：

- 更快地完成文献检索、整理和初稿搭建
- 让每个章节都尽量有出处、有上下文、可复核
- 把综述写作从“重复劳动”转向“高质量人工审阅与修订”
