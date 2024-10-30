# Marktext 版本 fix

当前社区版本的 marktext 在 macos 上有 2 个问题：

- 设置->image里面，如果选择 picgo，会发现找不到 picgo，如下

![image](https://user-images.githubusercontent.com/41283993/158584280-79e9d419-e8de-40ff-b3b5-c2e5b8714820.png)

但是实际上已经安装了 picgo-core 客户端，picgo 命令也是可以正常使用的，只是 marktext找不到相应的路径。

- 无法直接上传剪贴板中的图片，只能保存文件后再上传

本文通过修改源代码方式，修复这些配置，并重新编译生成 marktext 包，解决上述 2 个问题。

## 源码编译

如果需要编译源码，请按照 marktext 示例的文档进行基础操作：

https://github.com/marktext/marktext/blob/develop/docs/dev/BUILD.md

其中注意事项：

1. 使用 nvm 来更换 nodejs 版本号到 16

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash
```

重新登录 shell 以后，执行：

```shell
nvm install 16
```

检查 nvm 和 npm 版本

<img src="https://raw.githubusercontent.com/dodiadodia/picture/main/1730253556646.png" title="" alt="" width="315">

通过 npm 安装 yarn

```shell
npm install --global yarn
```

查看 yarn 的版本：

<img src="https://raw.githubusercontent.com/dodiadodia/picture/main/1730253682850.png" title="" alt="" width="400"> 

2. 确认下系统里面 python3 版本，如果版本大于 10，需要安装 setuptools：

[python - No module named &#39;distutils.util&#39; ...but distutils installed? - Stack Overflow](https://stackoverflow.com/questions/69919970/no-module-named-distutils-util-but-distutils-installed)

```shell
brew install python-setuptools
```

3. checkout 源码，进行编译

```shell
git clone https://github.com/marktext/marktext.git
cd marktext

yarn install #主要过程，如果出现依赖问题需要解决

yarn build #build 完成后，dmg 包就会在 build 目录中
```

这个是在 x86 平台上的示例

![](https://raw.githubusercontent.com/dodiadodia/picture/main/1730253939857.png)

## 无法找到 picgo

提示 picgo 没有 installed 的具体原因是 marktext 检查 path 路径的时候，没有配置 picgo 所在路径。例如如果用 brew 安装的 picgo-core，picgo 命令行所在位置如下

<img src="https://raw.githubusercontent.com/dodiadodia/picture/main/1730257267764.png" title="" alt="" width="304">

需要在 marktext 的代码配置，位置`src/main/app/env.js` 10行，添加`/opt/homebrew/bin`的路径：

```shell
const patchEnvPath = () => {
  if (process.platform === 'darwin') {
    process.env.PATH += (process.env.PATH.endsWith(path.delimiter) ? '' : path.delimiter) + '/opt/homebrew/bin:/Library/TeX/texbin'
  }
}
```

## picgo无法上传剪贴板图片

按照以下方式修改`src/renderer/util/fileSystem.js`

```javascript
# 152 行
const uploadByCommand = async (uploader, filepath, suffix = '') => {
# 157 行
filepath = path.join(tmpdir(), +new Date() + suffix)
# 227 行
uploadByCommand(currentUploader, reader.result, path.extname(image.name))
```

修改完上面两处后，按照编译方式直接生成 dmg 包即可。

