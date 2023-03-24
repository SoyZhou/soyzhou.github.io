---
title: 动态爬虫库 puppeteer
date: 2022-03-25 18:06:20
tags: nodejs
---

### 一、安装
1.直接在项目中使用 `Puppeteer`
  ```shell
  # npm 安装
  npm i puppeteer

  # yarn 安装
  yarn add puppeteer
  ```
  当安装 `puppeteer` 时，会下载最新版的 `Chromium`（~170MB Mac, ~282MB Linux, ~280MB Win）。

2.使用 `puppeteer-core`
  ```shell
  # npm 安装
  npm i puppeteer-core

  # yarn 安装
  yarn add puppeteer-core
  ```
  轻量版本，可以直接使用您当前的浏览器。需要确定浏览器版本是否匹配。

### 二、使用
1. 加载 & 初始化
  ```javascript
  const puppeteer = require('puppeteer');

  async function test() {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    await page.goto('https://example.com');
  }

  test()
  ```

2. 等待动态加载
   ```javascript
   await page.waitForTimeout(3000);
   ```

3. 保存截图
  ```javascript
  await page.screenshot({ path: 'example.png' });
  ```

4. 使用选择器获取元素信息
  ```javascript
  const resultsSelector = '#test > div > span';
  const result = await page.evaluate((resultsSelector) => {
      const anchors = Array.from(document.querySelectorAll(resultsSelector));
      return anchors.map((anchor) => anchor.textContent.trim());
    }, resultsSelector);
  ```

### 三、部署
#### 在 AWS EC2 Amazon-Linux 上使用 Puppeteer
1. 在安装 `Chromium` 需要先使用 `amazon-linux-extras` 安装 `EPEL`.
  ```shell
  sudo amazon-linux-extras install epel -y
  ```

2. 安装 `Chromium` 
  ```shell
  sudo yum install -y chromium
  ```

[**其他环境部署参考 ->**](https://github.com/puppeteer/puppeteer/blob/main/docs/troubleshooting.md#running-puppeteer-on-aws-ec2-instance-running-amazon-linux)