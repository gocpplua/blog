# 【pinus】框架编译方法

**一、安装　yarn \(我的是unbutu:16.04\)**

按照以下步骤在 Ubuntu 16.04/18.04 系统上安装 Yarn：

**步骤1.添加GPG密钥**

```text
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
```

**步骤2.添加Yarn存储库**

```text
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
```

**步骤3.更新包列表并安装Yarn**

```text
sudo apt update
sudo apt install yarn
```

如果您的系统上尚未安装 Node.js，则上面的命令将安装它。 那些使用 nvm 的人可以跳过 Node.js 安装：　

```text
sudo apt install --no-install-recommends yarn
```

**步骤4.检查Yarn的版本**

要验证 Yarn 是否已成功安装，请运行以下命令以打印 Yarn 版本号：

```text
yarn --version
```

输出：

```text
1.22.5
```

二、框架编译

```text
git clone https://github.com/node-pinus/pinus.git
cd pinus
yarn
yarn run build
```

备注: 编译中需要lerna，如果提示没安装的，请自行安装。

