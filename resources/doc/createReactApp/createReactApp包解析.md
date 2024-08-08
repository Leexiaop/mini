## create-react-app包

这里才是命令create-react-app命令的真正内容所之处，当我们在终端运行命令```npx create-react-app my-app```后，其中的奥秘就尽在其中。那我们就来解析其中的奥秘。

打开creat-react-app包，我们看到的目录是这样的：

```html
├─__test__               # 该包的测试文件
├─node_modules           # 独属于包自己的依赖
├─createReactApp.js      # createReactApp应用程序
├─index.js               # createReactApp应用程序的入口文件
├─LICENSE                # MIT
├─package.json           # 依赖文件
└─README.md              # 文档
```

看到目录结构极其简单，对于我们而言，我们只需要关注三个文件即可：

- package.json 该包中用到了那些依赖，这些依赖都是干什么的
- index.js 入口文件做了处理
- createReactApp.js正文程序，这个命令的背后有什么奥妙

### package.json 用到了些什么依赖

```JSON
{
    "dependencies": {
        "chalk": "^4.1.2",
        "commander": "^4.1.1",
        "cross-spawn": "^7.0.3",
        "envinfo": "^7.8.1",
        "fs-extra": "^10.0.0",
        "hyperquest": "^2.1.3",
        "prompts": "^2.4.2",
        "semver": "^7.3.5",
        "tar-pack": "^3.4.1",
        "tmp": "^0.2.1",
        "validate-npm-package-name": "^3.0.0"
    },
    "devDependencies": {
        "cross-env": "^7.0.3",
        "jest": "^27.4.3"
    }
}
```

讲这里的目的，主要是想看下create-react-app用到了那些依赖包，以便日后自己开发一个脚手架的时候，也能照搬照抄不费吹灰之力，所谓天下文章一大抄，就看会抄不会抄，对吧。
- chalk: 控制台信息美化工具，可以输出各种你想要的工具， https://www.npmjs.com/package/chalk
- commander: 控制台命令交互工具，https://www.npmjs.com/package/commander
- cross-spawn:夸平台进程解决工具，https://www.npmjs.com/package/cross-spawn
- envinfo:获取系统信息，https://www.npmjs.com/package/envinfo
- fs-extra:操作文件，读写文件内容，https://www.npmjs.com/package/fs-extra
- hyperquest:处理请求的包，https://www.npmjs.com/package/hyperquest
- prompts:实现node的命令问答，https://www.npmjs.com/package/prompts
- semver:处理模块的依赖关系，https://www.npmjs.com/package/semver
- tar-pack:包的自动管理工具，https://www.npmjs.com/package/tar-pack
- tmp:临时文件和目录的创建器，https://www.npmjs.com/package/tmp
- validate-npm-package-name:npm包校验，https://www.npmjs.com/package/validate-npm-package-name
- cross-env:配置环境变量，https://www.npmjs.com/package/cross-env
- jest:测试框架，https://jestjs.io/

### 入口文件做了什么处理

```js
'use strict';

const currentNodeVersion = process.versions.node;
const semver = currentNodeVersion.split('.');
const major = semver[0];

if (major < 14) {
  console.error(
    'You are running Node ' +
      currentNodeVersion +
      '.\n' +
      'Create React App requires Node 14 or higher. \n' +
      'Please update your version of Node.'
  );
  process.exit(1);
}

const { init } = require('./createReactApp');

init();
```
看到这段代码，你是不是会感到很惊讶？没错，createReactApp的入口文件就是这么简单，只是检查了一下，你用的node版本是多少，如果等于或者是低于14的node是不能使用该命令，然后就是打出错误信息，结束进程，如果node版本没有问题，那么就直接开始初始化命令，接下来就是createReactApp的事儿了。

### createReactApp中的奥妙

在整个createReactApp.js中，只有一个全局变量projectName,剩余的都是函数方法，我们按照程序执行的先后顺序，调用关系，来一一分析这其中的奥秘。

#### init函数

```js
function init() {
  const program = new commander.Command(packageJson.name)
    .version(packageJson.version)
    .arguments('<project-directory>')
    .usage(`${chalk.green('<project-directory>')} [options]`)
    .action(name => {
      projectName = name;
    })
    .option('--verbose', 'print additional logs')
    .option('--info', 'print environment debug info')
    .option(
      '--scripts-version <alternative-package>',
      'use a non-standard version of react-scripts'
    )
    .option(
      '--template <path-to-template>',
      'specify a template for the created project'
    )
    .option('--use-pnp')
    .allowUnknownOption()
    .on('--help', () => {
      console.log(
        `    Only ${chalk.green('<project-directory>')} is required.`
      );
      console.log();
      console.log(
        `    A custom ${chalk.cyan('--scripts-version')} can be one of:`
      );
      console.log(`      - a specific npm version: ${chalk.green('0.8.2')}`);
      console.log(`      - a specific npm tag: ${chalk.green('@next')}`);
      console.log(
        `      - a custom fork published on npm: ${chalk.green(
          'my-react-scripts'
        )}`
      );
      console.log(
        `      - a local path relative to the current working directory: ${chalk.green(
          'file:../my-react-scripts'
        )}`
      );
      console.log(
        `      - a .tgz archive: ${chalk.green(
          'https://mysite.com/my-react-scripts-0.8.2.tgz'
        )}`
      );
      console.log(
        `      - a .tar.gz archive: ${chalk.green(
          'https://mysite.com/my-react-scripts-0.8.2.tar.gz'
        )}`
      );
      console.log(
        `    It is not needed unless you specifically want to use a fork.`
      );
      console.log();
      console.log(`    A custom ${chalk.cyan('--template')} can be one of:`);
      console.log(
        `      - a custom template published on npm: ${chalk.green(
          'cra-template-typescript'
        )}`
      );
      console.log(
        `      - a local path relative to the current working directory: ${chalk.green(
          'file:../my-custom-template'
        )}`
      );
      console.log(
        `      - a .tgz archive: ${chalk.green(
          'https://mysite.com/my-custom-template-0.8.2.tgz'
        )}`
      );
      console.log(
        `      - a .tar.gz archive: ${chalk.green(
          'https://mysite.com/my-custom-template-0.8.2.tar.gz'
        )}`
      );
      console.log();
      console.log(
        `    If you have any problems, do not hesitate to file an issue:`
      );
      console.log(
        `      ${chalk.cyan(
          'shttps://github.com/facebook/create-react-app/isues/new'
        )}`
      );
      console.log();
    })
    .parse(process.argv);

  if (program.info) {
    console.log(chalk.bold('\nEnvironment Info:'));
    console.log(
      `\n  current version of ${packageJson.name}: ${packageJson.version}`
    );
    console.log(`  running from ${__dirname}`);
    return envinfo
      .run(
        {
          System: ['OS', 'CPU'],
          Binaries: ['Node', 'npm', 'Yarn'],
          Browsers: [
            'Chrome',
            'Edge',
            'Internet Explorer',
            'Firefox',
            'Safari',
          ],
          npmPackages: ['react', 'react-dom', 'react-scripts'],
          npmGlobalPackages: ['create-react-app'],
        },
        {
          duplicates: true,
          showNotFound: true,
        }
      )
      .then(console.log);
  }

  if (typeof projectName === 'undefined') {
    console.error('Please specify the project directory:');
    console.log(
      `  ${chalk.cyan(program.name())} ${chalk.green('<project-directory>')}`
    );
    console.log();
    console.log('For example:');
    console.log(
      `  ${chalk.cyan(program.name())} ${chalk.green('my-react-app')}`
    );
    console.log();
    console.log(
      `Run ${chalk.cyan(`${program.name()} --help`)} to see all options.`
    );
    process.exit(1);
  }

  // We first check the registry directly via the API, and if that fails, we try
  // the slower `npm view [package] version` command.
  //
  // This is important for users in environments where direct access to npm is
  // blocked by a firewall, and packages are provided exclusively via a private
  // registry.
  checkForLatestVersion()
    .catch(() => {
      try {
        return execSync('npm view create-react-app version').toString().trim();
      } catch (e) {
        return null;
      }
    })
    .then(latest => {
      if (latest && semver.lt(packageJson.version, latest)) {
        console.log();
        console.error(
          chalk.yellow(
            `You are running \`create-react-app\` ${packageJson.version}, which is behind the latest release (${latest}).\n\n` +
              'We recommend always using the latest version of create-react-app if possible.'
          )
        );
        console.log();
        console.log(
          'The latest instructions for creating a new app can be found here:\n' +
            'https://create-react-app.dev/docs/getting-started/'
        );
        console.log();
      } else {
        const useYarn = isUsingYarn();
        createApp(
          projectName,
          program.verbose,
          program.scriptsVersion,
          program.template,
          useYarn,
          program.usePnp
        );
      }
    });
}
```