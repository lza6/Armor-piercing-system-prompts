# “什么是 Skill”：程序动画示例

36 秒可编辑 HTML/SVG/HyperFrames 工程，展示八个镜头、整段旁白、时间轴和确定性文字。它适合流程、关系和独立元素运动，不是逐笔故事路线。

![程序动画封面](preview.png)

[观看 14 秒 MP4 样片](what-is-skill-sample.mp4)。样片中的人物、气泡、流程节点、箭头和文字都是独立 SVG/HTML 元素，由 GSAP/HyperFrames 按真实旁白时间轴驱动；它没有假装成“同一幅画被逐笔画出来”。

```powershell
npm install
npm run check
npm run dev
npm run render
```

`npm install` 安装固定版本的 GSAP；生命周期脚本从 npm 包复制浏览器文件到 `assets/gsap.min.js`。该生成文件不提交到仓库，许可遵循 GSAP 自身条款。HyperFrames 由命令按固定版本调用。

旁白只作案例资产。新项目优先复用用户提供的合格音轨；需要新合成时默认使用火山引擎刘飞 `zh_male_liufei_uranus_bigtts`，用户明确指定时可改用其他火山音色。不要把案例文件本身或电脑系统朗读当成所有项目的正式旁白。
