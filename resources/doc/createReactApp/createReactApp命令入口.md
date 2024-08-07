## 入口文件

在概览中我们看到了，create-react-app整个项目的核心目录，那么我们到底该从哪里看起呢？因为，create-react-app本质上在node环境中运行的一些命令，所以，我想入口应该先从package.json中的scripts中的命令开始看起吧：
```json
{
    "scripts": {
        "build": "cd packages/react-scripts && node bin/react-scripts.js build",
        "changelog": "lerna-changelog",
        "create-react-app": "node tasks/cra.js",
        "e2e": "tasks/e2e-simple.sh",
        "e2e:docker": "tasks/local-test.sh",
        "postinstall": "npm run build:prod -w react-error-overlay",
        "publish": "tasks/publish.sh",
        "start": "cd packages/react-scripts && node bin/react-scripts.js start",
        "screencast": "node ./tasks/screencast.js",
        "screencast:error": "svg-term --cast jyu19xGl88FQ3poMY8Hbmfw8y --out screencast-error.svg --window --at 12000 --no-cursor",
        "alex": "alex .",
        "test:integration": "jest test/integration",
        "test": "cd packages/react-scripts && node bin/react-scripts.js test",
        "eslint": "eslint .",
        "prettier": "prettier .",
        "format": "npm run prettier -- --write"
   },
}
```

这里，我们不对每一个命令做过多的解释，我们先从重要的命令开始看起，先看第一个命令：```create-react-app```.我们可以看到，他是一个node命令，运行了tasks目录下的cra.js.

### tasks/cra.js文件

打开文件，依照文件内容从上向下阅读。

首先，文件开头`use strict`表示，要使用严格模式，所以，在开发过程中要特别注意严格模式。

然后引入了三个node中的包：
```ts
const fs = require('fs');
const path = require('path');
const cp = require('child_process');
```
- fs包不用多说，是node中操作文件的一个包，了解更多可以查看：https://www.nodeapp.cn/fs.html
- path包不用多说，是node中获取文件路径的包，了解更多可以查看：https://www.nodeapp.cn/path.html
- child_process，是node中操开启子进程，做一些相关操作的包，了解更多可以查看：https://www.nodeapp.cn/child_process.html

#### cleanup

```js
const cleanup = () => {
  console.log('Cleaning up.');
  // Reset changes made to package.json files.
  cp.execSync(`git checkout -- packages/*/package.json`);
  // Uncomment when snapshot testing is enabled by default:
  // rm ./template/src/__snapshots__/App.test.js.snap
};
```
看函数名就能知道，通过开启子进程，放弃对packages下每个包的package.json的更改！

#### handleExit

```js
const handleExit = () => {
  cleanup();
  console.log('Exiting without error.');
  process.exit();
};
```

该方法主要是调用process.exit()来结束进程。

#### handleError

```js
const handleError = e => {
  console.error('ERROR! An error was encountered while executing');
  console.error(e);
  cleanup();
  console.log('Exiting with error.');
  process.exit(1);
};

process.on('SIGINT', handleExit);
process.on('uncaughtException', handleError);
```

错误处理，并且结束进程。并且通过process.on方法来监听进程。

#### car.js正文

接下来就是正文了：

```js
const gitStatus = cp.execSync(`git status --porcelain`).toString();

if (gitStatus.trim() !== '') {
  console.log('Please commit your changes before running this script!');
  console.log('Exiting because `git status` is not empty:');
  console.log();
  console.log(gitStatus);
  console.log();
  process.exit(1);
}

const rootDir = path.join(__dirname, '..');
const packagesDir = path.join(rootDir, 'packages');
const packagePathsByName = {};
fs.readdirSync(packagesDir).forEach(name => {
  const packageDir = path.join(packagesDir, name);
  const packageJson = path.join(packageDir, 'package.json');
  if (fs.existsSync(packageJson)) {
    packagePathsByName[name] = packageDir;
  }
});
Object.keys(packagePathsByName).forEach(name => {
  const packageJson = path.join(packagePathsByName[name], 'package.json');
  const json = JSON.parse(fs.readFileSync(packageJson, 'utf8'));
  Object.keys(packagePathsByName).forEach(otherName => {
    if (json.dependencies && json.dependencies[otherName]) {
      json.dependencies[otherName] = 'file:' + packagePathsByName[otherName];
    }
    if (json.devDependencies && json.devDependencies[otherName]) {
      json.devDependencies[otherName] = 'file:' + packagePathsByName[otherName];
    }
    if (json.peerDependencies && json.peerDependencies[otherName]) {
      json.peerDependencies[otherName] =
        'file:' + packagePathsByName[otherName];
    }
    if (json.optionalDependencies && json.optionalDependencies[otherName]) {
      json.optionalDependencies[otherName] =
        'file:' + packagePathsByName[otherName];
    }
  });

  fs.writeFileSync(packageJson, JSON.stringify(json, null, 2), 'utf8');
  console.log(
    'Replaced local dependencies in packages/' + name + '/package.json'
  );
});
console.log('Replaced all local dependencies for testing.');
console.log('Do not edit any package.json while this task is running.');

// Finally, pack react-scripts.
// Don't redirect stdio as we want to capture the output that will be returned
// from execSync(). In this case it will be the .tgz filename.
const scriptsFileName = cp
  .execSync(`npm pack`, { cwd: path.join(packagesDir, 'react-scripts') })
  .toString()
  .trim();
const scriptsPath = path.join(packagesDir, 'react-scripts', scriptsFileName);
const args = process.argv.slice(2);
const craScriptPath = path.join(packagesDir, 'create-react-app', 'index.js');
cp.execSync(
  `node ${craScriptPath} ${args.join(' ')} --scripts-version="${scriptsPath}"`,
  {
    cwd: rootDir,
    stdio: 'inherit',
  }
);
handleExit();
```

- 首先开启进程，通过git status命令来检查一下，当前工作区，有没有更改过的记录，如果有，那么就输出提示信息，并且退出进程。
- 如果没有修改记录，那么说明该工作区域可以操作，接下来就是获取packages路径，并且遍历该路径下的文件，然后将每个包中的package.json的路径存入到数组packagePathsByName对象中。
- 遍历packagePathsByName,使得package.json能够得到重新组织，并且添加file新的字段
- 将重新组织好的package.json再重新回写到每个包中。这样做的目的，就是为了替代在本地packages下所有包中package.json的本地以来。
- 然后通过开启子进程，运行命令npm pack打包项目中的包，并且拼接node执行create-react-app命令的路径，执行node create-react-app命令。正式进入到创建环节。
- 执行完成后，调用handleExist()退出进程。

所以到目前为止，只是创建项目的前奏，也是准备工作，接下开就是开启create-react-app命令真正的开始。

## 所用node api

- 包：
    + fs
    + path
    + child_process
- 方法：
    + fs.readdirSync()
    + fs.writeFileSync()
    + cp.execSync()
    + path.join()
    + process.exist()
