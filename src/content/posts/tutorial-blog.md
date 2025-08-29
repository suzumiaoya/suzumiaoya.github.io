---
title: 如何搭建一个自己的个人博客
published: 2025-08-29
description: 使用Github Action搭建个人博客
tags: [教程, git, Github, 命令行]
category: 教程
draft: false
---

### **Tutorial to Create Your Own Blog**

#### **原理**

在本地对静态网页进行修改，再使用 Git push 到 GitHub 仓库中。GitHub Actions 和 GitHub Pages 会自动将这些静态网页托管在 GitHub 的服务器上。

-----

### **准备工作**

* 一个你自己的 GitHub 账号。
* 选择一个你喜欢的静态网页模板（[Astro 提供了许多不错的选择](https://astro.build/themes/)）。
* 稳定的网络连接（指可以正常访问你的 GitHub 仓库并使用 Git）。

-----

### **第一步：在 GitHub 上为你的个人博客创建仓库**

1. 进入你选择的网页模板的 GitHub 页面，点击右上角的 **"Fork"**。
2. 在 **Repository name** 一栏，填写你的仓库名，格式必须是：`你的github用户名.github.io`。其他设置保持默认即可，然后点击 **"Create fork"**。

-----

### **第二步：创建 SSH 连接**

SSH 连接能让你更安全、更便捷地与 GitHub 仓库进行交互。

1. **生成 SSH 密钥**：

      * 在终端中进入 `C:/Users/你的用户名/.ssh` 目录。
      * 输入以下指令并按三次回车键：

        ```bash
        ssh-keygen -t rsa -C "你的github邮箱"
        ```

      * 看到 `The key fingerprint is: SHA256: ...` 的内容时，说明密钥已创建成功。

2. **将公钥添加到 GitHub**：

      * 在浏览器上进入你的 GitHub 主页，选择 **Settings**，在左侧找到 **SSH and GPG keys**，然后点击 **New SSH key**。
      * **Title** 可以随意填写。
      * **Key** 部分用你刚刚创建的公钥文件 `id_rsa.pub` 的内容填充。用记事本打开该文件，复制所有内容并粘贴进去。
      * 点击 **Add SSH key**。

3. **测试连接**：

      * 在终端中输入 `ssh -T git@github.com`。
      * 如果出现 `Hi 你的github用户名! You've successfully authenticated...` 的提示，说明 SSH 连接已成功建立。

-----

### **第三步：在本地克隆你的仓库**

1. 在你喜欢的位置创建用于存放仓库的文件夹（例如 `D:/blog/`）。
2. 在终端中进入该目录，输入 `git init` 将其初始化为一个 Git 仓库。
3. 进入你在 GitHub 上创建的仓库页面，点击 **Code**，选择 **SSH** 选项，复制给出的 URL。
4. 在终端中输入 `git clone` 加上刚刚复制的 URL。克隆完成后，你会看到一个新的文件夹，名称就是你的仓库名。

-----

### **第四步：本地配置与远程连接**

1. **清除 Git 全局设置**：
    为了避免账户混淆，建议先清除 Git 的全局用户名和邮箱。

    ```bash
    git config --global --unset user.name
    git config --global --unset user.email
    ```

2. **配置本地仓库**：
      * 进入你的仓库文件夹：`cd 你的github用户名.github.io`。
      * 配置只对当前仓库生效的用户名和邮箱：

        ```bash
        git config user.name "你的github用户名"
        git config user.email "github绑定的邮箱"
        ```

3. **验证连接**：
      * 输入 `git remote -v`，如果出现以下内容，说明连接成功：

        ```
        origin  git@github.com:你的github用户名/你的github用户名.github.io.git (fetch)
        origin  git@github.com:你的github用户名/你的github用户名.github.io.git (push)
        ```

-----

### **第五步：配置 GitHub Pages 并部署**

1. 在你的 GitHub 仓库页面，点击 **Actions**，然后选择 **enable them** 开启它。
2. 在 **Settings** 里面的 **Pages** 中，将 **Source** 更改为 **GitHub Actions**。
3. 在终端中输入 `pnpm i` 安装依赖，然后输入 `pnpm dev` 启动本地网页。你可以按住 `Ctrl` 再点击网址，在浏览器中预览你的网页。
4. 预览完成后，在终端里按下 `Ctrl + C` 退出。
5. 使用以下指令将本地网页推送到 GitHub：

    ```bash
    git add .
    git commit -m "feat: first page release"
    git push
    ```

6. 提交完成后，等待 GitHub 仓库部署。部署完成后，在浏览器中输入 `你的github用户名.github.io` 即可在线访问网页。

-----

### **如何更新你的博客**

* 阅读你所使用的模板的 `README` 部分，它会告诉你如何修改网页配置和撰写新帖子。
* 在本地修改完成后，你可以再次运行 `pnpm dev` 预览。
* 确认无误后，使用以下 Git 指令提交更改并推送到 GitHub，你的博客就会自动更新。

    ```bash
    git add .
    git commit -m "本次更新的内容说明"
    git push
    ```
