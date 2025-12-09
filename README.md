# 知识管理与产品文件自动生成系统

集成 **FastAPI + Neo4j + 大模型** 的知识库后端与 **Vue3 + Vite + Element Plus** 的前端，可实现：

1. DeepSeek OCR 驱动的产品/配件文档自动解析与结构化存储  
2. 基于知识图谱与多模态模型的规格页、海报、说明书生成  
3. BOM 编码自动识别、知识库可视化与交互式检索  
4. ACE 智能体持续学习闭环（人工反馈→提示词/Playbook 优化）

---

## 目录结构

```
agent/
├── backend/     # FastAPI 服务与数据处理流水线
└── frontend/    # Vue3 前端（规格页/海报/说明书编辑与展示）
```

---

## 后端（`backend/`）

| 模块 | 说明 |
| --- | --- |
| `api_server.py` | FastAPI 入口，提供 OCR、知识库、规格页、BOM、ACE 等接口 |
| `src/manual_ocr.py` | 手工上传 OCR 流程：文件落盘→PDF/Office 转图片→DeepSeek OCR→结果归档 |
| `src/rag_specsheet.py` | RAG + 多模态规格页生成，支持从 Neo4j 文档或 OCR 结果生成 JSON |
| `src/specsheet_storage.py` / `src/bom_storage.py` | Session 级规格页/BOM 暂存，供前端编辑与回写 |
| `src/ace_integration.py` | ACE 持续学习：pending sample、playbook 存取、适配调用 |
| `src/api_queries.py` | Neo4j 查询与写入封装：产品/配件、文档节点、chunk/embedding、插入/挂接 |
| `src/manual_progress.py` | OCR 进度状态管理，供前端轮询 |

### 核心接口速览

- `POST /api/manual/ocr`：上传产品/配件文件，触发 DeepSeek OCR。  
- `GET /api/manual/ocr/history|{id}|progress`：查看 OCR 历史与实时进度；`DELETE /api/manual/ocr/{id}` 清理会话。  
- `POST /api/manual/insert-neo4j`：将 OCR 结果或选定文件一键写入 Neo4j 产品/配件节点。  
- `POST /api/specsheet/from_ocr_docs`：基于 OCR 文档生成规格页 JSON，并写入 ACE 待学习样本。  
- `GET/POST /api/manual/specsheet/{session_id}`：读取/保存手工会话规格页草稿。  
- `POST /api/manual/specsheet/{session_id}/ace`：提交人工修订稿，触发 ACE 训练。  
- `GET /api/products/{product}/boms/{bom}/specsheet`：知识库内按产品+BOM 拉取规格页（RAG 支持）。  
- `GET /api/bom/session/{session_id}`：查询该会话自动生成的 BOM 编码。

### 运行

```bash
cd backend
cp .env.example .env   # 若存在示例文件
# 编辑 .env，填写 Neo4j / LLM / DeepSeek OCR 等配置
uv venv .venv && source .venv/bin/activate
uv pip install -r requirements.txt  # 或 pip / uv pip sync
uvicorn api_server:app --host 0.0.0.0 --port 8000
```

---

## 前端（`frontend/`）

Vue3 + Vite + Element Plus，包含：

- 首页产品列表、BOM 选择页、产品详情（规格页 / 海报 / 说明书三套模板）  
- 规格页：容量/喷嘴/水泵、主图、色板、特性块；支持导出图片/PDF  
- 海报：多卡片布局、冷热区数据、底部规格条（计划通过工作流自动生成）  
- 说明书：目录、章节、段落/列表/图片/表格/步骤/警示块等多种模块  
- OCR、知识库、规格页/海报/说明书接口对接与可视化编辑

### 开发启动

```bash
cd frontend
npm install
npm run dev
```

默认使用 Vite，若需自定义 API 地址可在 `.env` 系列文件中配置。

---

## 工作流概览

1. **上传 & OCR**：前端上传产品/配件原始文档 → `POST /api/manual/ocr` → DeepSeek OCR 输出 Markdown / 图片。  
2. **知识库入库**：勾选合适文档 → `POST /api/manual/insert-neo4j` → Neo4j 中建立 Product/Accessory ↔ Document 关联。  
3. **规格页/BOM 生成**：  
   - OCR 内联生成：`POST /api/specsheet/from_ocr_docs` + `GET/POST /api/manual/specsheet/{session}`  
   - 知识库 RAG：`GET /api/products/{product}/boms/{bom}/specsheet`  
   - BOM 自动生成：`src/rag_bom.py` + `GET /api/bom/session/{session_id}`  
4. **ACE 闭环**：前端人工调优规格页 → `POST /api/manual/specsheet/{session}/ace`，同步优化 Playbook。  
5. **前端展示/导出**：规格页/海报/说明书模板渲染，支持导出 PDF/图片、预览知识库节点关系。

---

## 下一步规划

1. **说明书 & 海报自动生成**：复用 OCR / RAG 管线，补完文档模板，海报采用工作流串联（素材筛选→布局→渲染）。  
2. **ACE 深度优化**：扩大样本回放、引入指标监控，提升自动生成一致性。  
3. **知识库可视化增强**：更丰富的过滤、对齐产品/配件/BOM 的交互式分析能力。

---

## 许可与贡献

- 欢迎提交 Issue / PR 讨论需求与改进。  
- 上传至 GitHub 前请确保 `.env`、日志等敏感文件已排除。  
- 默认遵循项目原有许可（若未指定，可在推送前补充 LICENSE）。
