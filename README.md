# Tauri 2.0 + Next.js 15 App Router Template

![Tauri window screenshot](public/tauri-nextjs-template-2_screenshot.png)

This is a [Tauri](https://v2.tauri.app/) project template using [Next.js](https://nextjs.org/),
bootstrapped by combining [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app)
and [`create tauri-app`](https://v2.tauri.app/start/create-project/).

This template uses [`pnpm`](https://pnpm.io/) as the Node.js dependency
manager, and uses the [App Router](https://nextjs.org/docs/app) model for Next.js.

## Template Features

- TypeScript frontend using [Next.js 15](https://nextjs.org/) React framework
- [TailwindCSS 4](https://tailwindcss.com/) as a utility-first atomic CSS framework
  - The example page in this template app has been updated to use only TailwindCSS
  - While not included by default, consider using
    [React Aria components](https://react-spectrum.adobe.com/react-aria/index.html)
    and/or [HeadlessUI components](https://headlessui.com/) for completely unstyled and
    fully accessible UI components, which integrate nicely with TailwindCSS
- Opinionated formatting and linting already setup and enabled
  - [Biome](https://biomejs.dev/) for a combination of fast formatting, linting, and
    import sorting of TypeScript code, and [ESLint](https://eslint.org/) for any missing
    Next.js linter rules not covered by Biome
  - [clippy](https://github.com/rust-lang/rust-clippy) and
    [rustfmt](https://github.com/rust-lang/rustfmt) for Rust code
- GitHub Actions to check code formatting and linting for both TypeScript and Rust

## Getting Started

### Running development server and use Tauri window

After cloning for the first time, change your app identifier inside
`src-tauri/tauri.conf.json` to your own:

```jsonc
{
  // ...
  // The default "com.tauri.dev" will prevent you from building in release mode
  "identifier": "com.my-application-name.app",
  // ...
}
```

To develop and run the frontend in a Tauri window:

```shell
pnpm tauri dev
```

This will load the Next.js frontend directly in a Tauri webview window, in addition to
starting a development server on `localhost:3000`.
Press <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>I</kbd> in a Chromium based WebView (e.g. on
Windows) to open the web developer console from the Tauri window.

### Building for release

To export the Next.js frontend via SSG and build the Tauri application for release:

```shell
pnpm tauri build
```

### Source structure

Next.js frontend source files are located in `src/` and Tauri Rust application source
files are located in `src-tauri/`. Please consult the Next.js and Tauri documentation
respectively for questions pertaining to either technology.

## Caveats

### Static Site Generation / Pre-rendering

Next.js is a great React frontend framework which supports server-side rendering (SSR)
as well as static site generation (SSG or pre-rendering). For the purposes of creating a
Tauri frontend, only SSG can be used since SSR requires an active Node.js server.

Please read into the Next.js documentation for [Static Exports](https://nextjs.org/docs/app/building-your-application/deploying/static-exports)
for an explanation of supported / unsupported features and caveats.

### `next/image`

The [`next/image` component](https://nextjs.org/docs/basic-features/image-optimization)
is an enhancement over the regular `<img>` HTML element with server-side optimizations
to dynamically scale the image quality. This is only supported when deploying the
frontend onto Vercel directly, and must be disabled to properly export the frontend
statically. As such, the
[`unoptimized` property](https://nextjs.org/docs/api-reference/next/image#unoptimized)
is set to true for the `next/image` component in the `next.config.js` configuration.
This will allow the image to be served as-is, without changes to its quality, size,
or format.

### ReferenceError: window/navigator is not defined

If you are using Tauri's `invoke` function or any OS related Tauri function from within
JavaScript, you may encounter this error when importing the function in a global,
non-browser context. This is due to the nature of Next.js' dev server effectively
running a Node.js server for SSR and hot module replacement (HMR), and Node.js does not
have a notion of `window` or `navigator`.

The solution is to ensure that the Tauri functions are imported as late as possible
from within a client-side React component, or via [lazy loading](https://nextjs.org/docs/app/building-your-application/optimizing/lazy-loading).

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and
  API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

And to learn more about Tauri, take a look at the following resources:

- [Tauri Documentation - Guides](https://v2.tauri.app/start/) - learn about the Tauri
  toolkit.

## 在 macOS 上交叉编译 Windows 应用程序

如果你需要在 macOS 上构建 Windows 版本的应用程序，需要安装一些额外的依赖项和工具。以下是完整的步骤：

### 准备工作

1. 首先，需要安装 Windows 目标平台的 Rust 工具链：

```bash
rustup target add x86_64-pc-windows-msvc
```

2. 安装 NSIS 安装程序框架（用于创建 Windows 安装包）：

```bash
brew install nsis
```

3. 安装 LLVM，包含 Windows 资源编译器：

```bash
brew install llvm
```

4. 将 LLVM 添加到环境变量 PATH 中（需要在当前会话和永久配置中都添加）：

```bash
# 当前会话临时添加
export PATH="/opt/homebrew/opt/llvm/bin:$PATH"

# 永久添加到配置文件
echo 'export PATH="/opt/homebrew/opt/llvm/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

5. 安装 cargo-xwin 工具（关键步骤）：

```bash
cargo install --locked cargo-xwin
```

### 使用 cargo-xwin 进行跨平台构建

从 Tauri v1.3 开始，官方提供了实验性的跨平台构建功能，允许在 macOS 上构建 Windows 应用程序。这种方法使用 cargo-xwin 作为构建运行器，它会自动下载并配置 Windows SDK。

构建命令：

```bash
pnpm run build:win
```

这个命令已在 package.json 中配置为：

```json
"build:win": "tauri build --runner cargo-xwin --target x86_64-pc-windows-msvc"
```

构建成功后，Windows 安装包将位于：
```
src-tauri/target/x86_64-pc-windows-msvc/release/bundle/nsis/
```

### 跨平台构建的局限性

1. 这是一个实验性功能，可能不适用于所有项目
2. 不支持跨平台构建的应用签名
3. 某些带有复杂 C/C++ 依赖的 Rust crate 可能会构建失败

如果遇到问题，可以设置 `XWIN_CACHE_DIR` 环境变量来指定 Windows SDK 的缓存位置：

```bash
export XWIN_CACHE_DIR=~/xwin-cache
```

### 其他可能遇到的问题

1. **错误：Target x86_64-pc-windows-msvc is not installed**

   解决方案：运行 `rustup target add x86_64-pc-windows-msvc` 安装相应的目标平台。

2. **错误：called `Result::unwrap()` on an `Err` value: NotAttempted("llvm-rc")**

   解决方案：安装 LLVM 并确保 `llvm-rc` 在 PATH 中可用。
   
   ```bash
   brew install llvm
   export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
   ```

3. **错误与替代方案**

   如果使用 cargo-xwin 仍然无法成功构建，可以考虑以下替代方案：

#### 方案一：使用 Docker 容器

使用包含 Windows 构建环境的 Docker 容器进行构建。

#### 方案二：使用 GitHub Actions

通过 GitHub Actions 在 Windows 虚拟机上构建，这是官方推荐的跨平台构建方式。

#### 方案三：使用 Windows 虚拟机

在 macOS 上使用 Windows 虚拟机进行构建。

## 使用 GitHub Actions 进行跨平台构建

本项目已配置 GitHub Actions 工作流，可以自动构建跨平台应用程序。使用这种方法，你不需要在本地进行复杂的交叉编译设置。

### 自动构建流程

项目中包含两个工作流：

1. **测试构建** (`.github/workflows/test-build.yml`)
   - 在每次推送到 `main` 或 `dev` 分支时触发
   - 在 macOS、Windows 和 Linux 平台上进行构建测试
   - 不创建发布版本，仅验证代码是否可以在各平台上构建

2. **发布构建** (`.github/workflows/release.yml`)
   - 在创建新标签 (以 `v` 开头，如 `v0.1.0`) 时触发
   - 也可以通过 GitHub 界面手动触发
   - 构建所有平台的应用并创建 GitHub Release
   - 生成以下版本：
     - macOS Intel (x86_64)
     - macOS Apple Silicon (ARM64)
     - Windows 64位
     - Linux 64位

### 如何发布新版本

要使用 GitHub Actions 发布新版本，请按照以下步骤操作：

1. 更新 `package.json` 中的版本号
2. 提交更改并推送到 GitHub
3. 创建一个新标签，格式为 `v版本号`（例如 `v0.2.1`）：

```bash
git tag v0.2.1
git push origin v0.2.1
```

4. GitHub Actions 将自动触发构建过程
5. 构建完成后，可以在项目的 Releases 页面找到所有平台的安装包

### 手动触发构建

你也可以手动触发发布构建：

1. 进入项目的 GitHub 页面
2. 点击 "Actions" 选项卡
3. 从左侧列表选择 "Release" 工作流
4. 点击 "Run workflow" 按钮
5. 选择要从哪个分支构建，然后点击 "Run workflow"