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

看一下init方法主要做的事儿：

+ 通过commander命令获取创建项目的参数，比如我们在终端运行create-react-app myApp的命令，那么我们就可以通过commander命令获取到参数myApp,而且这里是必传的参数，否则就会提示错误。剩余其他的log，就是运行命令的提示信息，最终通过 .parse(process.argv)方法解析参数。就是这么简单，不要被庞大的代码吓到。

+ 紧接着就是通过checkForLatestVersion方法获取了npm上当前最新的create-react-app的版本并且和当前开发的版本号作对比，如果是npm上的版本高，那就提示你要更新当前版本，其实这是一个废话昂。但是他就这么干了。咱也没办法。

+ 如果一切都检查没有问题了，那么就调用createApp函数，开始进行项目的创建。

#### checkForLatestVersion函数

```js
const checkForLatestVersion = () => {
	return new Promise((resolve, reject) => {
		https
			.get(
				'https://registry.npmjs.org/-/package/create-react-app/dist-tags',
				(res) => {
					console.log(res);
					if (res.statusCode === 200) {
						let body = '';
						res.on('data', (data) => (body += data));
						res.on('end', () => {
							resolve(JSON.parse(body).latest);
						});
					} else {
						reject(new Error('something bad happened'));
					}
				}
			)
			.on('error', () => reject(new Error('something bad happened')));
	});
};
```

因为node中的https包，还是一个callback的方式，所以这里发送了一个get请求后，并且封装了一个pormise并返回获取到的当前已经发布的npm最新的create-react-app包的版本号。

#### isUsingYarn函数
```js
function isUsingYarn() {
  return (process.env.npm_config_user_agent || '').indexOf('yarn') === 0;
}
```

#### createApp函数

```js
function createApp(name, verbose, version, template, useYarn, usePnp) {
  const unsupportedNodeVersion = !semver.satisfies(
    // Coerce strings with metadata (i.e. `15.0.0-nightly`).
    semver.coerce(process.version),
    '>=14'
  );

  if (unsupportedNodeVersion) {
    console.log(
      chalk.yellow(
        `You are using Node ${process.version} so the project will be bootstrapped with an old unsupported version of tools.\n\n` +
          `Please update to Node 14 or higher for a better, fully supported experience.\n`
      )
    );
    // Fall back to latest supported react-scripts on Node 4
    version = 'react-scripts@0.9.x';
  }

  const root = path.resolve(name);
  const appName = path.basename(root);

  checkAppName(appName);
  fs.ensureDirSync(name);
  if (!isSafeToCreateProjectIn(root, name)) {
    process.exit(1);
  }
  console.log();

  console.log(`Creating a new React app in ${chalk.green(root)}.`);
  console.log();

  const packageJson = {
    name: appName,
    version: '0.1.0',
    private: true,
  };
  fs.writeFileSync(
    path.join(root, 'package.json'),
    JSON.stringify(packageJson, null, 2) + os.EOL
  );

  const originalDirectory = process.cwd();
  process.chdir(root);
  if (!useYarn && !checkThatNpmCanReadCwd()) {
    process.exit(1);
  }

  if (!useYarn) {
    const npmInfo = checkNpmVersion();
    if (!npmInfo.hasMinNpm) {
      if (npmInfo.npmVersion) {
        console.log(
          chalk.yellow(
            `You are using npm ${npmInfo.npmVersion} so the project will be bootstrapped with an old unsupported version of tools.\n\n` +
              `Please update to npm 6 or higher for a better, fully supported experience.\n`
          )
        );
      }
      // Fall back to latest supported react-scripts for npm 3
      version = 'react-scripts@0.9.x';
    }
  } else if (usePnp) {
    const yarnInfo = checkYarnVersion();
    if (yarnInfo.yarnVersion) {
      if (!yarnInfo.hasMinYarnPnp) {
        console.log(
          chalk.yellow(
            `You are using Yarn ${yarnInfo.yarnVersion} together with the --use-pnp flag, but Plug'n'Play is only supported starting from the 1.12 release.\n\n` +
              `Please update to Yarn 1.12 or higher for a better, fully supported experience.\n`
          )
        );
        // 1.11 had an issue with webpack-dev-middleware, so better not use PnP with it (never reached stable, but still)
        usePnp = false;
      }
      if (!yarnInfo.hasMaxYarnPnp) {
        console.log(
          chalk.yellow(
            'The --use-pnp flag is no longer necessary with yarn 2 and will be deprecated and removed in a future release.\n'
          )
        );
        // 2 supports PnP by default and breaks when trying to use the flag
        usePnp = false;
      }
    }
  }

  run(
    root,
    appName,
    version,
    verbose,
    originalDirectory,
    template,
    useYarn,
    usePnp
  );
}
```

一看参数，我勒个去，这么多，是不是有点吓人，别慌，挺住，这都是纸老虎，我们先不要看参数都是干什么的，我们直接阅读代码，遇到所用的参数再说。仔细一读，你就会发现，函数体的内容和他的函数名八竿子打不着，这就是一个校验函数，是在创建之前的一些列校验而已。

+ 第一步，还是检查你得node版本，如果不能大于等于14，那么就输出提示信息
+ 接下来就是对于创建项目的文件名校验，这里是通过checkAppName方法
+ 当项目名检查通过后，那就说明项目可以创建了，所以，此时就创建一个以参数为name为名的文件夹，并且向其根目录下写入package.json文件。其内容也很简单，只有三个字段，name,version,private.
+ 接下来就是检查，能不能用yarn 或者是npm，或者pnp的信息，这里用的俩个函数是usePnp，useYarn()和checkThatNpmCanReadCwd(),否则就退出进程。
+ 当一切包管理工具都检查通过后，说明我们的包管理工具也正常，那么就调用run方法，开始创建

#### checkAppName函数

```js
function checkAppName(appName) {
  const validationResult = validateProjectName(appName);
  if (!validationResult.validForNewPackages) {
    console.error(
      chalk.red(
        `Cannot create a project named ${chalk.green(
          `"${appName}"`
        )} because of npm naming restrictions:\n`
      )
    );
    [
      ...(validationResult.errors || []),
      ...(validationResult.warnings || []),
    ].forEach(error => {
      console.error(chalk.red(`  * ${error}`));
    });
    console.error(chalk.red('\nPlease choose a different project name.'));
    process.exit(1);
  }

  // TODO: there should be a single place that holds the dependencies
  const dependencies = ['react', 'react-dom', 'react-scripts'].sort();
  if (dependencies.includes(appName)) {
    console.error(
      chalk.red(
        `Cannot create a project named ${chalk.green(
          `"${appName}"`
        )} because a dependency with the same name exists.\n` +
          `Due to the way npm works, the following names are not allowed:\n\n`
      ) +
        chalk.cyan(dependencies.map(depName => `  ${depName}`).join('\n')) +
        chalk.red('\n\nPlease choose a different project name.')
    );
    process.exit(1);
  }
}
```
首先是检查了，文件夹名称是不是禁用的，或者是是不是重复，如果是，那么就提示，并退出，其次检查是不是和react依赖相同的名字，比如不能用react作为您的项目文件名，否则也提示信息，并且退出。是不是很简单。

#### checkThatNpmCanReadCwd函数
```js
function checkThatNpmCanReadCwd() {
  const cwd = process.cwd();
  let childOutput = null;
  try {
    // Note: intentionally using spawn over exec since
    // the problem doesn't reproduce otherwise.
    // `npm config list` is the only reliable way I could find
    // to reproduce the wrong path. Just printing process.cwd()
    // in a Node process was not enough.
    childOutput = spawn.sync('npm', ['config', 'list']).output.join('');
  } catch (err) {
    // Something went wrong spawning node.
    // Not great, but it means we can't do this check.
    // We might fail later on, but let's continue.
    return true;
  }
  if (typeof childOutput !== 'string') {
    return true;
  }
  const lines = childOutput.split('\n');
  // `npm config list` output includes the following line:
  // "; cwd = C:\path\to\current\dir" (unquoted)
  // I couldn't find an easier way to get it.
  const prefix = '; cwd = ';
  const line = lines.find(line => line.startsWith(prefix));
  if (typeof line !== 'string') {
    // Fail gracefully. They could remove it.
    return true;
  }
  const npmCWD = line.substring(prefix.length);
  if (npmCWD === cwd) {
    return true;
  }
  console.error(
    chalk.red(
      `Could not start an npm process in the right directory.\n\n` +
        `The current directory is: ${chalk.bold(cwd)}\n` +
        `However, a newly started npm process runs in: ${chalk.bold(
          npmCWD
        )}\n\n` +
        `This is probably caused by a misconfigured system terminal shell.`
    )
  );
  if (process.platform === 'win32') {
    console.error(
      chalk.red(`On Windows, this can usually be fixed by running:\n\n`) +
        `  ${chalk.cyan(
          'reg'
        )} delete "HKCU\\Software\\Microsoft\\Command Processor" /v AutoRun /f\n` +
        `  ${chalk.cyan(
          'reg'
        )} delete "HKLM\\Software\\Microsoft\\Command Processor" /v AutoRun /f\n\n` +
        chalk.red(`Try to run the above two lines in the terminal.\n`) +
        chalk.red(
          `To learn more about this problem, read: https://blogs.msdn.microsoft.com/oldnewthing/20071121-00/?p=24433/`
        )
    );
  }
  return false;
}
```
#### checkNpmVersion函数

```js
function checkNpmVersion() {
  let hasMinNpm = false;
  let npmVersion = null;
  try {
    npmVersion = execSync('npm --version').toString().trim();
    hasMinNpm = semver.gte(npmVersion, '6.0.0');
  } catch (err) {
    // ignore
  }
  return {
    hasMinNpm: hasMinNpm,
    npmVersion: npmVersion,
  };
}
```
#### checkYarnVersion函数
```js
function checkYarnVersion() {
  const minYarnPnp = '1.12.0';
  const maxYarnPnp = '2.0.0';
  let hasMinYarnPnp = false;
  let hasMaxYarnPnp = false;
  let yarnVersion = null;
  try {
    yarnVersion = execSync('yarnpkg --version').toString().trim();
    if (semver.valid(yarnVersion)) {
      hasMinYarnPnp = semver.gte(yarnVersion, minYarnPnp);
      hasMaxYarnPnp = semver.lt(yarnVersion, maxYarnPnp);
    } else {
      // Handle non-semver compliant yarn version strings, which yarn currently
      // uses for nightly builds. The regex truncates anything after the first
      // dash. See #5362.
      const trimmedYarnVersionMatch = /^(.+?)[-+].+$/.exec(yarnVersion);
      if (trimmedYarnVersionMatch) {
        const trimmedYarnVersion = trimmedYarnVersionMatch.pop();
        hasMinYarnPnp = semver.gte(trimmedYarnVersion, minYarnPnp);
        hasMaxYarnPnp = semver.lt(trimmedYarnVersion, maxYarnPnp);
      }
    }
  } catch (err) {
    // ignore
  }
  return {
    hasMinYarnPnp: hasMinYarnPnp,
    hasMaxYarnPnp: hasMaxYarnPnp,
    yarnVersion: yarnVersion,
  };
```

#### run函数

```js
function run(
  root,
  appName,
  version,
  verbose,
  originalDirectory,
  template,
  useYarn,
  usePnp
) {
  Promise.all([
    getInstallPackage(version, originalDirectory),
    getTemplateInstallPackage(template, originalDirectory),
  ]).then(([packageToInstall, templateToInstall]) => {
    const allDependencies = ['react', 'react-dom', packageToInstall];

    console.log('Installing packages. This might take a couple of minutes.');

    Promise.all([
      getPackageInfo(packageToInstall),
      getPackageInfo(templateToInstall),
    ])
      .then(([packageInfo, templateInfo]) =>
        checkIfOnline(useYarn).then(isOnline => ({
          isOnline,
          packageInfo,
          templateInfo,
        }))
      )
      .then(({ isOnline, packageInfo, templateInfo }) => {
        let packageVersion = semver.coerce(packageInfo.version);

        const templatesVersionMinimum = '3.3.0';

        // Assume compatibility if we can't test the version.
        if (!semver.valid(packageVersion)) {
          packageVersion = templatesVersionMinimum;
        }

        // Only support templates when used alongside new react-scripts versions.
        const supportsTemplates = semver.gte(
          packageVersion,
          templatesVersionMinimum
        );
        if (supportsTemplates) {
          allDependencies.push(templateToInstall);
        } else if (template) {
          console.log('');
          console.log(
            `The ${chalk.cyan(packageInfo.name)} version you're using ${
              packageInfo.name === 'react-scripts' ? 'is not' : 'may not be'
            } compatible with the ${chalk.cyan('--template')} option.`
          );
          console.log('');
        }

        console.log(
          `Installing ${chalk.cyan('react')}, ${chalk.cyan(
            'react-dom'
          )}, and ${chalk.cyan(packageInfo.name)}${
            supportsTemplates ? ` with ${chalk.cyan(templateInfo.name)}` : ''
          }...`
        );
        console.log();

        return install(
          root,
          useYarn,
          usePnp,
          allDependencies,
          verbose,
          isOnline
        ).then(() => ({
          packageInfo,
          supportsTemplates,
          templateInfo,
        }));
      })
      .then(async ({ packageInfo, supportsTemplates, templateInfo }) => {
        const packageName = packageInfo.name;
        const templateName = supportsTemplates ? templateInfo.name : undefined;
        checkNodeVersion(packageName);
        setCaretRangeForRuntimeDeps(packageName);

        const pnpPath = path.resolve(process.cwd(), '.pnp.js');

        const nodeArgs = fs.existsSync(pnpPath) ? ['--require', pnpPath] : [];

        await executeNodeScript(
          {
            cwd: process.cwd(),
            args: nodeArgs,
          },
          [root, appName, verbose, originalDirectory, templateName],
          `
        const init = require('${packageName}/scripts/init.js');
        init.apply(null, JSON.parse(process.argv[1]));
      `
        );

        if (version === 'react-scripts@0.9.x') {
          console.log(
            chalk.yellow(
              `\nNote: the project was bootstrapped with an old unsupported version of tools.\n` +
                `Please update to Node >=14 and npm >=6 to get supported tools in new projects.\n`
            )
          );
        }
      })
      .catch(reason => {
        console.log();
        console.log('Aborting installation.');
        if (reason.command) {
          console.log(`  ${chalk.cyan(reason.command)} has failed.`);
        } else {
          console.log(
            chalk.red('Unexpected error. Please report it as a bug:')
          );
          console.log(reason);
        }
        console.log();

        // On 'exit' we will delete these files from target directory.
        const knownGeneratedFiles = ['package.json', 'node_modules'];
        const currentFiles = fs.readdirSync(path.join(root));
        currentFiles.forEach(file => {
          knownGeneratedFiles.forEach(fileToMatch => {
            // This removes all knownGeneratedFiles.
            if (file === fileToMatch) {
              console.log(`Deleting generated file... ${chalk.cyan(file)}`);
              fs.removeSync(path.join(root, file));
            }
          });
        });
        const remainingFiles = fs.readdirSync(path.join(root));
        if (!remainingFiles.length) {
          // Delete target folder if empty
          console.log(
            `Deleting ${chalk.cyan(`${appName}/`)} from ${chalk.cyan(
              path.resolve(root, '..')
            )}`
          );
          process.chdir(path.resolve(root, '..'));
          fs.removeSync(path.join(root));
        }
        console.log('Done.');
        process.exit(1);
      });
  });
}
```

这个函数看起来貌似有点难度哈，貌似请求了挺多了东西。我们一一来看。首先第一个promise.all(),通过getInstallPackage和getTemplateInstallPackage函数请求了点东西，看名字好像是都和包有关，