# Make Contribution

We hope DeepTutor could become a gift for the community. 🎁

## Join Our Community

<div class="community-links">
  <a href="https://github.com/HKUDS/DeepTutor/issues/78" class="community-badge wechat">
    💬 WeChat Community
  </a>
  <a href="https://github.com/HKUDS/DeepTutor/issues/167" class="community-badge wechat-collab">
    🤝 WeChat Co-creators
  </a>
  <a href="https://discord.gg/eRsjPgMU4t" class="community-badge discord">
    🌐 Discord
  </a>
</div>

## We Welcome Contributions!

Whether you're fixing bugs, improving documentation, or adding new features, your contributions are valuable to us.

### How to Contribute

1. **Report Bugs** — Found a bug? Open an issue on GitHub with reproduction steps
2. **Suggest Features** — Share ideas in GitHub Discussions or our community channels
3. **Improve Docs** — Help us improve documentation, tutorials, and examples
4. **Submit Code** — Fix bugs or implement new features through pull requests

### Contribution Guidelines

For detailed guidelines, see [CONTRIBUTING.md](https://github.com/HKUDS/DeepTutor/blob/dev/CONTRIBUTING.md).

**Key Points:**

- All contributions must be based on the `dev` branch
- Run `pre-commit run --all-files` before submitting
- Use conventional commit format: `feat:`, `fix:`, `docs:`, etc.

### Quick Start

```bash
# Fork and clone
git clone https://github.com/YOUR_USERNAME/DeepTutor.git
cd DeepTutor

# Create feature branch from dev
git checkout dev && git pull origin dev
git checkout -b feature/your-feature-name

# Install pre-commit hooks
pip install pre-commit && pre-commit install

# Make changes, then submit PR to dev branch
```

## Our Contributors

<a href="https://github.com/HKUDS/DeepTutor/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=HKUDS/DeepTutor&max=999" alt="Contributors" />
</a>

---

Thank you for your interest in contributing to DeepTutor! 🚀

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
