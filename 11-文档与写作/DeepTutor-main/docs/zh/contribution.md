# 参与贡献

我们希望 DeepTutor 能成为送给社区的礼物。🎁

## 加入我们的社区

<div class="community-links">
  <a href="https://github.com/HKUDS/DeepTutor/issues/78" class="community-badge wechat">
    💬 微信社区群
  </a>
  <a href="https://github.com/HKUDS/DeepTutor/issues/167" class="community-badge wechat-collab">
    🤝 微信共创群
  </a>
  <a href="https://discord.gg/eRsjPgMU4t" class="community-badge discord">
    🌐 Discord
  </a>
</div>

## 我们欢迎贡献！

无论您是修复 bug、改进文档还是添加新功能，您的贡献对我们都很有价值。

### 如何贡献

1. **报告 Bug** — 发现了 bug？在 GitHub 上开一个 issue，附上复现步骤
2. **建议功能** — 在 GitHub Discussions 或我们的社区渠道分享想法
3. **改进文档** — 帮助我们改进文档、教程和示例
4. **提交代码** — 通过 Pull Request 修复 bug 或实现新功能

### 贡献指南

详细指南请参阅 [CONTRIBUTING.md](https://github.com/HKUDS/DeepTutor/blob/dev/CONTRIBUTING.md)。

**要点：**

- 所有贡献必须基于 `dev` 分支
- 提交前运行 `pre-commit run --all-files`
- 使用约定式提交格式：`feat:`、`fix:`、`docs:` 等

### 快速入门

```bash
# Fork 并克隆
git clone https://github.com/YOUR_USERNAME/DeepTutor.git
cd DeepTutor

# 从 dev 创建功能分支
git checkout dev && git pull origin dev
git checkout -b feature/your-feature-name

# 安装预提交钩子
pip install pre-commit && pre-commit install

# 进行更改，然后向 dev 分支提交 PR
```

## 我们的贡献者

<a href="https://github.com/HKUDS/DeepTutor/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=HKUDS/DeepTutor&max=999" alt="贡献者" />
</a>

---

感谢您对 DeepTutor 贡献的兴趣！🚀

<style>
.community-links {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
  margin: 20px 0;
}

.community-badge {
  display: inline-flex;
  align-items: center;
  padding: 10px 18px;
  border-radius: 8px;
  font-weight: 500;
  font-size: 0.95rem;
  text-decoration: none;
  transition: all 0.2s ease;
}

.community-badge:hover {
  transform: translateY(-2px);
}

.community-badge.wechat {
  background: #07C160;
  color: white;
}

.community-badge.wechat-collab {
  background: #1AAD19;
  color: white;
}

.community-badge.discord {
  background: #5865F2;
  color: white;
}
</style>
