# Git Workflow

这个仓库用于管理你的 AI 应用开发 / Agent / RAG 面试知识库。

## 日常流程

```bash
git status
git add .
git commit -m "Update interview notes"
```

## 建议提交节奏

- 每补完一个项目卡，提交一次。
- 每整理 3-5 道 Top30 高频题，提交一次。
- 每次模拟面试后，把改好的答案沉淀进文件，再提交一次。

## 常用命令

查看修改：

```bash
git diff
```

查看提交历史：

```bash
git log --oneline
```

撤销某个文件的未提交修改：

```bash
git restore path/to/file.md
```

如果之后要推到 GitHub：

```bash
git remote add origin <your-repo-url>
git branch -M main
git push -u origin main
```

