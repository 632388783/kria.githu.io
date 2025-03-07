---
title: Windows 11 WSL 安装 Ubuntu 22.04 教程
index_img: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
banner_img: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
cover: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
date: 2025-03-04 18:37:31
tags: [wsl, ubuntu]
categories: [wsl, ubuntu]
toc: true
permalink: /wsl-install-ubuntu22.04/
description: Windows 11 WSL 安装 Ubuntu 22.04 教程
comment: 'valine'
---

# Windows 11 WSL 安装 Ubuntu 22.04 教程

## 步骤 1：启用 WSL

1. 打开 PowerShell（以管理员身份运行）。
2. 输入以下命令以启用 WSL 和虚拟机平台：
   ```powershell
   wsl --install
   ```
3. 系统会自动安装所需的组件并重启计算机。

## 步骤 2：安装 Ubuntu 22.04

1. 重启后，再次打开 PowerShell（以管理员身份运行）。
2. 输入以下命令以查看可用的 Linux 发行版：
   ```powershell
   wsl --list --online
   ```
3. 输入以下命令以安装 Ubuntu 22.04：
   ```powershell
   wsl --install -d Ubuntu-22.04
   ```
4. 安装完成后，系统会提示你设置 Ubuntu 用户名和密码。

## 步骤 3：更新 Ubuntu 软件包

1. 打开 Ubuntu 终端（可以通过开始菜单找到 Ubuntu 应用）。
2. 输入以下命令以更新软件包列表并升级已安装的软件包：
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

## 步骤 4：验证安装

1. 在 Ubuntu 终端中输入以下命令以验证安装：
   ```bash
   lsb_release -a
   ```
2. 你应该会看到类似以下的输出，显示 Ubuntu 22.04 已成功安装：
   ```
   No LSB modules are available.
   Distributor ID: Ubuntu
   Description:    Ubuntu 22.04 LTS
   Release:        22.04
   Codename:       jammy
   ```

## 结论

现在你已经在 Windows 11 上成功安装并配置了 WSL 和 Ubuntu 22.04。你可以开始在 WSL 中使用 Ubuntu 进行开发和其他任务了。
