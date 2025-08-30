# ⚙️ Shared GitHub Actions

This repository serves as a **central hub for reusable GitHub Actions** designed to streamline automation across multiple projects. By consolidating common workflows and actions here, you can ensure consistency, reduce duplication, and simplify maintenance.

## 📚 Overview

- General-purpose GitHub Actions
- Reusable workflows via `workflow_call`
- Automation scripts and templates
- Designed for use across various repositories

## 🚀 How to Use

You can reference actions or workflows from this repository in other projects using the following formats:

### 🔹 Using an Action

```yaml
uses: your-username/your-repo-name@v1
with:
  input1: value
  input2: value
```

### 🔹 Using a Reusable Workflow
```yaml
jobs:
  example:
    uses: your-username/your-repo-name/.github/workflows/workflow-name.yml@v1
    with:
      input1: value
    secrets:
      TOKEN: ${{ secrets.TOKEN }}
```

✅ Tip: Use version tags (@v1, @main, or specific commit SHAs) to ensure stability and avoid breaking changes.

🤝 Contributing

Feel free to open issues or pull requests to improve or expand the available actions. Contributions are welcome!
