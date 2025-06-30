[![img](https://github.com/layabox/LayaAir/raw/LayaAir_3.2/logo.png)](https://layaair.com/)

# LayaAir Engine Documentation

**The [LayaAir](https://layaair.com/) engine from [Layabox](https://www.layabox.com/) is a cross-platform engine** that can be deployed to multiple platforms with one click. In addition to HTML5 web, it also supports publishing native apps (Android, iOS, HarmonyOS NEXT, Windows, Mac, Linux) and mini-games (WeChat Mini Games, ByteDance Mini Games, Alipay Mini Games, OPPO Mini Games, vivo Mini Games, Xiaomi Fast Games, Taobao Mini Games, etc.).

It supports both 2D and 3D development and is applied in a wide range of fields including gaming, education, advertising, marketing, digital twins, the metaverse, AR guided tours, VR experiences, architectural design, industrial design, and many more.

The LayaAir engine provides a powerful integrated IDE environment that includes a 3D scene editor, material editor, particle editor, blueprint editor, animation editor, physics editor, and UI editor. The IDE offers extensive extensibility for developers to customize their workflows, and developers can even upload plugins to the resource store for sharing and sales.

![Screenshot of LayaAirIDE](https://official.layabox.com/laya_data/news/2025/0104/7-1.gif)

(Figure 1) 2D Particle Visual Editing Effect

![](https://official.layabox.com/laya_data/news/2023/0630/21.gif) 

(Figure 2) 3D Particle Visual Editing Effect

## Guidelines for Contributing to Documentation

We welcome developers to modify and submit documentation for the LayaAir engine as we work together to build a robust developer ecosystem. For proactive and high-quality contributors, the official LayaAir team will issue honor certificates and provide free VIP-level LayaAir engine services.

### 1. Based on Markdown Syntax

The LayaAir documentation is written in Markdown. If you are familiar with the syntax, you can write and preview the documentation using editors like VS Code.

We also recommend using Markdown editors such as Typora for a superior writing and reading experience.

### 2. File and Directory Naming Conventions

- Directory names should be based on the structure and hierarchy of the engine and IDE, with clear and logical names. Refer to the existing directory structure as a guide.
- The file name must be `readme.md`. For example, the documentation for WebXR, which belongs to the 3D engine, should be placed at `3D/WebXR/readme.md`.
- Images used in the documentation should be stored in a dedicated directory, usually at the same level as the `readme.md` file. For instance, the image directory for the WebXR documentation is located at `3D/WebXR/img`.

### 3. Documentation Quality Requirements

- The article title must be a level 1 heading, indicated by `#`.
- Subsection headings must be level 2 headings, indicated by `##`.
- Since the documentation website's navigation supports only up to level 3 headings, try to keep the document within three levels (using `###` for level 3 headings).
- The documentation should have a clear and logical navigation structure with a balance of text and images, and it should be as beginner-friendly as possible by clearly explaining detailed procedures.
- Every piece of content must be validated through testing and compiled objectively based on facts, rather than relying on subjective experience.

If you encounter any issues while writing the documentation, please contact our official customer service for assistance.

![](https://layaair.com/img/wechat.8197aa26.jpg) 

(Scan to add our customer service WeChat)

## How to Compile Markdown into a Static Website

Usually, you can simply read the compiled website documentation on the official engine website ([https://layaair.com/3.x/doc/](https://layaair.com/3.x/doc/)). However, if you need to set up the documentation locally or preview the output, you can follow the guide below:

### 1. Installing the Environment

The LayaAir3 documentation is compiled using GitBook. **Note that the GitBook environment has been tested to work only with Node 10.24.x versions.**

If your local Node environment is not this version, it is recommended to install the nvm management tool first for managing multiple Node versions, available at [https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases).

#### Common nvm Commands

Install a specific Node version, for example, 10.24.1:

```sh
nvm install 10.24.1
```

List the installed Node versions:

```sh
nvm list
```

Switch to a specific Node version, for example, 10.24.1:

```sh
// nvm use [version] [arch] : Use the specified Node version. You can specify 32/64-bit  
nvm use 10.24.1
```

#### Installing gitbook-cli

Once you have set up the above environment and are running Node 10.24.x, install gitbook-cli:

```sh
npm install -g gitbook-cli
```

### 2. Common GitBook Commands

Compile the documentation into a static website:

```sh
gitbook build
```

> After executing the `gitbook build` command, a `_book` folder will be generated in the root directory. The contents of this folder represent the static website for the documentation.

Compile the documentation into a website and start a local web server:

```sh
gitbook serve
```

> The `gitbook serve` command will first call `gitbook build` to compile the book, and then start a web server, which by default listens on port 4000. Open [http://localhost:4000](http://localhost:4000/) in your browser to view the e-book.

### 3. Explanation of Compilation-Related Files

- The `SUMMARY.md` file serves as the website's navigation menu. By adjusting this file, you can change the structure of the left-side navigation on the website. You can open `SUMMARY.md` to review the specific format.
- The `book.json` file is the configuration file for GitBook plugins, which is used to configure commonly used GitBook plugins and improve the documentation's usability.

