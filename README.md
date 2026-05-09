<h3 align="center">
  <img src="https://teedy.io/img/github-title.png" alt="Teedy" width=500 />
</h3>

[![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
[![Maven CI/CD](https://github.com/sismics/docs/actions/workflows/build-deploy.yml/badge.svg)](https://github.com/sismics/docs/actions/workflows/build-deploy.yml)

Teedy is an open source, lightweight document management system for individuals and businesses.

<hr />
<h2 align="center">
  ✨ <a href="https://github.com/users/jendib/sponsorship">Sponsor this project if you use and appreciate it!</a> ✨
</h2>
<hr />

![New!](https://teedy.io/img/laptop-demo.png?20180301)

# Demo

A demo is available at [demo.teedy.io](https://demo.teedy.io)

- Guest login is enabled with read access on all documents
- "admin" login with "admin" password
- "demo" login with "password" password 

# Features

- Responsive user interface
- Optical character recognition
- LDAP authentication ![New!](https://www.sismics.com/public/img/new.png)
- Support image, PDF, ODT, DOCX, PPTX files
- Video file support
- Flexible search engine with suggestions and highlighting
- Full text search in all supported files
- All [Dublin Core](http://dublincore.org/) metadata
- Custom user-defined metadata ![New!](https://www.sismics.com/public/img/new.png)
- Workflow system ![New!](https://www.sismics.com/public/img/new.png)
- 256-bit AES encryption of stored files
- File versioning ![New!](https://www.sismics.com/public/img/new.png)
- Tag system with nesting
- Import document from email (EML format)
- Automatic inbox scanning and importing
- User/group permission system
- 2-factor authentication
- Hierarchical groups
- Audit log
- Comments
- Storage quota per user
- Document sharing by URL
- RESTful Web API
- Webhooks to trigger external service
- Fully featured Android client
- [Bulk files importer](https://github.com/sismics/docs/tree/master/docs-importer) (single or scan mode)
- Tested to one million documents


# Native Installation

## Requirements

Before building Teedy from source, you will need to install several prerequisites, including Java 11+, Maven 3+, NPM, Grunt, Tesseract 4, ffmpeg, and mediainfo.
We give instructions for installing these prerequisites on several platforms below.

### Linux (Ubuntu 22.04)

```console
sudo apt install \
  default-jdk \
  ffmpeg \
  grunt \
  maven \
  npm \
  tesseract-ocr-all
```

### Mac

```console
brew install \
  ffmpeg \
  grunt-cli \
  maven \
  mediainfo \
  npm \
  openjdk \
  tesseract \
  tesseract-lang
```

### Windows

It is highly recommended that you proceed to install Windows Subsystem Linux (WSL), following the link: [Install Linux on Windows with WSL
](https://docs.microsoft.com/en-us/windows/wsl/install). This will allow you to run a Linux distro (Ubuntu's the default) within the Windows environment, and you can then proceed to follow the Linux (Ubuntu 22.04) instructions to install the dependencies.

**Note**: This would mean that you should proceed to execute the following instructions within the Linux environment as well.


## How to build Teedy from the sources

Prerequisites: JDK 11, Maven 3, NPM, Grunt, Tesseract 4

Teedy is organized in several Maven modules:

- docs-core
- docs-web
- docs-web-common

First off, clone the repository: `git clone https://github.com/sustech-cs304/Teedy`
or download the sources from GitHub.

### Launch the build

From the root directory:

```console
mvn clean -DskipTests install
```

### Run a stand-alone version

From the `docs-web` directory:

```console
mvn jetty:run
```

### Build a .war to deploy to your servlet container

From the `docs-web` directory:

```console
mvn -Pprod -DskipTests clean install
```

You will get your deployable WAR in the `docs-web/target` directory.

# Contributing

All contributions are more than welcomed. Contributions may close an issue, fix a bug (reported or not reported), improve the existing code, add new feature, and so on.

The `master` branch is the default and base branch for the project. It is used for development and all Pull Requests should go there.

# License

Teedy is released under the terms of the GPL license. See `COPYING` for more
information or see <http://opensource.org/licenses/GPL-2.0>.

---

# Practice 8 — JaCoCo 测试覆盖率报告

## 前提条件

- JDK 11+
- Maven 3.6+

---

## 生成含三项报告的 Maven Site

执行以下**完整命令**（必须包含 `site:stage`，否则模块间链接全部失效）：

```bash
mvn clean test site site:stage -Dmaven.test.failure.ignore=true
```

然后用浏览器打开：

```
target/staging/index.html
```

在 **Project Reports** 中可看到：

| 报告 | 位置 | 说明 |
|------|------|------|
| Surefire Report | 父模块页面 | 所有模块测试结果汇总 |
| JaCoCo Aggregate | 父模块页面 | 跨模块覆盖率汇总 |
| JaCoCo | 点击左侧 docs-core / docs-web 子模块后 | 该模块自身覆盖率 |

> **注意**：父模块（docs-parent）无 Java 源码，所以只有子模块才有 "JaCoCo" 单模块报告。
> staging 会将所有模块的 site 聚合，子模块页面的导航栏中也会出现父模块的报告链接。

---

## 步骤一：保存原始覆盖率截图

运行上面的命令后，打开 `target/staging/index.html`，截图保存 Surefire Report、JaCoCo、JaCoCo Aggregate 的数值作为**基线**。

---

## 步骤二：只运行你新增的测试类

```bash
mvn -Dtest=YourNewTestClass test
```

将 `YourNewTestClass` 替换为你的实际类名（不含包名），例如：

```bash
mvn -Dtest=TestUserDao test
```

---

## 步骤三：重新生成完整报告对比覆盖率

```bash
mvn clean test site site:stage -Dmaven.test.failure.ignore=true
```

再次打开 `target/staging/index.html`，JaCoCo 的 **Instructions** 和 **Branches** 数值应高于步骤一的截图。

---

## 评分演示流程（对应 Evaluation 要求）

| 顺序 | 执行命令 | 展示内容 |
|------|----------|----------|
| 1 | `mvn clean test site site:stage -Dmaven.test.failure.ignore=true` | 原始 Surefire / JaCoCo / JaCoCo Aggregate 报告 |
| 2 | — | 新增测试类的代码 |
| 3 | `mvn -Dtest=YourNewTestClass test` | 新测试单独运行且通过 |
| 4 | `mvn clean test site site:stage -Dmaven.test.failure.ignore=true` | 新报告，覆盖率提升 |
