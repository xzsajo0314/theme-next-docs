---
title: 教你手搓一個屬於自己的博客 NexT開源篇
date: 2025-09-28 00:37:59
---

> 更新时间：2025年9月28日 00点37分

## Fork 本項目到自己的 GitHub 倉庫（博客原石）
<https://github.com/next-theme/theme-next-docs>

![GitHub全自動部署-14.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758990540605_GitHub全自動部署-14.png)

![GitHub全自动部署-2.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758920815866_GitHub全自动部署-2.png)

![GitHub全自动部署-3.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758920816732_GitHub全自动部署-3.png)

![GitHub全自动部署-4.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758920819579_GitHub全自动部署-4.png)

![GitHub全自动部署-9.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758979030584_GitHub全自动部署-9.png)

## 在本地創建 SSH 密鑰對（鑰匙、鎖）
   - 打開終端或命令提示符
   - 執行以下命令創建密鑰對
     ```bash
     ssh-keygen -t rsa -b 4096
     ```
   - 將生成的公鑰（鎖） `id_rsa_next.pub` 內容複製到 GitHub 倉庫的部署密鑰設置中
     - 執行以下命令查看公鑰內容
     ```bash
     cat ~/.ssh/id_rsa_next.pub
     ```
     或者
     ```bash
     type $env:USERPROFILE\.ssh\id_rsa_next.pub
     ```
     - 複製終端上的公鑰內容
     - 進入您的倉庫設置頁面
     - 選擇`SSH and GPG keys`
     - 點擊`New SSH key`
     - 將公鑰內容粘貼到`Key`字段中
     - 為密鑰添加`Title`，例如`id_rsa_next.pub`
     - 點擊`Add SSH key`保存密鑰  

## 管理多個賬號GitHub倉庫的SSH密鑰
   - 若您有其他 GitHub 賬號，需要為每個賬號創建一個獨立的密鑰對
   - 每個密鑰對的文件名稱應包含賬號名稱，例如`id_rsa_vitepress`、`id_rsa_next`等
   - 編輯`~/.ssh/config`文件（若不存在，則創建一個），添加以下內容
```bash
Host github-vitepress
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_vitepress

Host github-next
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_next
```

![GitHub登录凭据.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758920815738_GitHub登录凭据.png)

## 刪除 HTTPS 登錄憑證，改用 SSH 登錄（博客鑰匙）
   - 刪除 HTTPS 登錄憑證后，git clone `博客原石` 到本地
   - 打開終端或命令提示符
   - 執行以下命令克隆`博客原石`到本地
     ```bash
     git clone git@github-next:<您的用戶名>/theme-next-docs.git
     ```
   - 確認登錄憑證已替換
     ```bash
     git remote -v
     ```
     應該會看到類似以下的輸出：
     ```bash
     origin  git@github-next:<您的用戶名>/theme-next-docs.git (fetch)
     origin  git@github-next:<您的用戶名>/theme-next-docs.git (push)
     ```
   - 若不是以上的輸出：
     - 打開終端或命令提示符
     - 執行以下命令將遠程倉庫地址替換為 SSH 地址
     ```bash
     git remote set-url origin git@github-next:<您的用戶名>/theme-next-docs.git 
     ```
## 將原工作流全數刪除，寫入GitHub Actions全自動部署工作流
   - 刪除倉庫`.github/workflows/`文件夾裏的所有文件
   - 在倉庫`.github/workflows/`文件夾中創建一個名為`pages.yml`的文件
   - 填入以下內容
```yaml
name: Pages

on:
  push:
    branches:
      - master # default branch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 20
        uses: actions/setup-node@v4
        with:
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
          # Ref: https://github.com/actions/setup-node#supported-version-syntax
          node-version: "20"
      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

![GitHub全自動部署-16.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758991502863_GitHub全自動部署-16.png)

## 修改package.json文件
  - 打開`package.json`文件
  - 找到`"lint": "markdownlint -c .markdownlint.json source/docs"`這一行
  - 在它後面添加`,`並另起一行填入一下内容
  ```bash
  "build": "hexo clean && hexo generate",
  "clean": "hexo clean",
  "deploy": "hexo deploy",
  "server": "hexo server"
  ```

![GitHub全自動部署-15.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758991030216_GitHub全自動部署-15.png)

## 修改博客url和root
  - 打開 `_config.yml`文件
  - 尋找 `url: https://theme-next.js.org`
  - 修改爲 `url: https://<您的用戶名>.github.io/theme-next-docs/`

## 設置Build and deployment from GitHub Actions
  - In your GitHub repo’s setting, navigate to `Settings` > `Pages` > `Source`. Change the source to `GitHub Actions` and save.

![GitHub全自動部署-13.png](https://cloudflare-imgbed-7oz.pages.dev/file/1758989293234_GitHub全自動部署-13.png)

## Commit and Push
  ```bash
  git add .
  git commit -m "github action update"
  git push origin master
  ```