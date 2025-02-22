---
uid: 20231216155914
title: Quickadd 脚本：一键在系统应用中打开图片编辑
tags: ['quickadd', 'quickadd脚本', '图片', '附件']
description: 可以一键在系统软件中打开笔记图片的quickadd脚本
author: ProudBenzene
type: advanced
draft: false
editable: false
modified: 20231220113036
---

# Quickadd 脚本：一键在系统应用中打开图片编辑

![一键在系统应用中打开图片编辑|800](https://cdn.pkmer.cn/images/202312161705682.png!pkmer)

## 功能需求

我笔记中的很多图片都是通过截图后直接粘贴插入的，因此难免有的图片上有些瑕疵，需要用图片编辑器进行细微的修正。

通常要完成这些操作，我们需要：

1. 在 Obsidian 的附近文件夹中找到该图片文件
	- 直接在库中搜索文件名找到图片文件
	- 在出链面板中根据文件名找到图片文件
	- 在系统文件管理器中打开附件文件夹，找到图片文件
	- …
2. 双击图片（或右键图片选择打开方式才）打开图片

这些操作不仅极为繁复，而且很不优雅，比较浪费时间。

## 脚本说明

使用下面的脚本，可以直接在系统软件中打开选中的图片：

![一键在系统应用中打开图片编辑](https://cdn.pkmer.cn/images/202312161647394.gif!pkmer)

1. 在编辑模式下点击图片，Obsidian 会自动为你选中完整的图片嵌入语句
2. 使用带有该脚本的 Quickadd Macro，一键打开图片编辑器

> [!warning] 注意
> 图片在笔记中的嵌入形式必须满足以下条件本脚本才能生效：
> 1. 链接为标准 markdown 链接，不能使 Wiki 链接
> 2. 内部链接类型为以下两种之一：
> 	- 基于当前笔记的相对路径
> 	- 基于仓库根目录的绝对路径

## 脚本

```js
const { exec } = require('child_process');
const path = require("path");

// 获取笔记的基本路径
const filePath = app.workspace.getActiveFile().path;
const fileFullPath = app.vault.adapter.getFullPath(filePath)
const basePath = (app.vault.adapter).getBasePath()
// 根据相对路径得到图片的绝对路径
const selection = getSelection();
const regex = /\((.*?)\)/;
const matches = regex.exec(selection);
const selectionPath = matches[1]; //去掉嵌入语法后的图片路径
console.log(selectionPath);
const decodedPath = decodeURIComponent(selectionPath);
console.log(decodedPath);
const imagePath = path.resolve(path.dirname(fileFullPath), decodedPath); // 根据相对路径得到绝对路径
// 根据基于仓库的绝对路径得到图片的绝对路径
const Abselection = getSelection();
const Abregex = /\((.*?)\)/;
const Abmatches = Abregex.exec(Abselection);
const selectionAbPath = Abmatches[1]; //去掉嵌入语法后的图片路径
console.log(selectionAbPath);
const decodedAbPath = decodeURIComponent(selectionAbPath);
console.log(decodedAbPath);
const imageAbPath = basePath + "/" + decodedAbPath; // 绝对路径

let QuickAdd;
module.exports = async function openSelectedImage(params) {
	QuickAdd = params;
	new Notice(`打开图片编辑器成功`, 5000);
};

// 使用默认应用打开文件
//Windows
/*
exec(`start "" "${imagePath}"`, (error, stdout, stderr) => {
    if (error || stderr) {
        exec(`start "" "${imageAbPath}"`, (error, stdout, stderr) => {
            if (error) {
                console.error(`打开文件时出错: ${error.message}`);
                return;
            }
            if (stderr) {
                console.error(`打开文件时出错: ${stderr}`);
                return;
            }
            console.log(`文件已成功打开`);
        });
    } else {
        console.log(`文件已成功打开`);
    }
});
*/
//macOS
exec(`open  -a "Adobe Photoshop 2022" "${imagePath}"`, (error, stdout, stderr) => { // 尝试如果将选中图片路径按相对路径处理能否打开图片
    if (error || stderr) {
        exec(`open  -a "Adobe Photoshop 2022" "${imageAbPath}"`, (error, stdout, stderr) => { //如果不能，尝试将选中图片路径按绝对路径处理
            if (error) {
                console.error(`打开文件时出错: ${error.message}`);
                return;
            }
            if (stderr) {
                console.error(`打开文件时出错: ${stderr}`);
                return;
            }
            console.log(`文件已成功打开`);
        });
    } else {
        console.log(`文件已成功打开`);
    }
});
```

> [!warning] 注意
> 1. 根据自己的操作系统，选择注释掉另一个系统的「打开」代码
> 2. 脚本中 Windows 版是打开 Windows 默认图片编辑器，macOS 中的预览不够强，所以我设置的是打开 PS，可以自行修改具体使用哪个第三方应用打开图片