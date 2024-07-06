### 创建GithubPages项目
- 基础操作见[此处](https://docs.github.com/en/pages/quickstart)
- **注意**：Repo类型请选择Public,普通用户无法使用Private建立GhPages
    
### 个性化
现在你获得了一片空白的网页,为它添加一些内容吧。
- 可以使用Jekyll,Hexo之类来生成个性化的静态网页，方便简洁
- 在你现在正看到的这个Blog上，笔者没有采用如上方法，只是简单的采用Github的方案进行配置：
    - 采用/docs文件夹生成网页而只将main作为草稿箱
    故后文默认在docs中建立文件
        1. 在库中建立/docs
        2. 在 Settings-Pages-Branch 中选择 main /docs
    - 配置页面
        1. 手动建立 _config.yml 文件并写入配置,根据自己需求调整:
        ```
        title: 标题
        theme: 页面主题,笔者选用了jekyll-theme-cayman
        ```
        更多主题样式在[这里](https://pages.github.com/themes/)
- 一些统计卡片
    - [GitHub Readme Stats](https://github.com/anuraghazra/github-readme-stats#github-readme-stats)
