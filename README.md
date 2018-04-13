# 如何打造自己的脚手架[转]

# 前言

### 在实际的开发过程中，从零开始建立项目的结构是一件让人头疼的事情，所以各种各样的脚手架工具应运而生。笔者使用较多的`yoeman`，`express-generator`和`vue-cli`便是当中之一。它们功能丰富，但最核心的功能都是能够快速搭建一个完整的项目的结构，开发者只需要在生成的项目结构的基础上进行开发即可，非常简单高效。作为一个不折腾会死星人，在熟悉了使用方法以后就开始琢磨起它们的原理来了。经过仔细研究文档和源码，终于算是摸清了其核心的原理，并且依据这个原理自己搭建了一款叫做[SCION](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fjrainlau%2Fscion)的脚手架。现在让我们就以SCION为例，从零开始搭建一款属于我们自己的脚手架工具吧！

# 核心原理

### `yoeman`搭建项目需要提供`yoeman-generator`。`yoeman-generator`本质上就是一个具备完整文件结构的项目样板，用户需要手动地把这些`generator`下载到本地，然后`yoeman`就会根据这些`generator`自动生成各种不同的项目。`vue-cli`提供了相当丰富的选项和设定功能，但是其本质也是从远程仓库把不同的`模版`拉取到本地，而并非是什么“本地生成”的黑科技。这样看来，思路也就有了——首先建立不同的样板项目，然后脚手架根据用户的指令引用样板项目生成实际项目。样板项目可以内置在脚手架当中，也可以部署在远程仓库。为了更广的适用范围，`SCION`采用的是第二种方式。

# 技术选型

### [node.js](https://link.juejin.im/?target=https%3A%2F%2Fnodejs.org%2F)：整个脚手架工具的根本组成部分，推荐使用最新的版本。[es6](https://link.juejin.im/?target=http%3A%2F%2Fes6.ruanyifeng.com%2F)：新版本的`node.js`对于`es6`的支持度已经非常高，使用`es6`能够极大地提升开发效率和开发感受。[commander](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ftj%2Fcommander.js%2F)：TJ大神开发的工具，能够更好地组织和处理命令行的输入。[co](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ftj%2Fco)：TJ大神开发的异步流程控制工具，用更舒服的方式写异步代码。[co-prompt](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ftj%2Fco-prompt)：还是TJ大神的作品……传统的命令行只能单行一次性地输入所有参数和选项，使用这个工具可以自动提供提示信息，并且分步接收用户的输入，体验类似`npm init`时的一步一步输入参数的过程。

# 整体架构

### 国际惯例，着手开发之前得先弄明白整体架构，看图：![img](https://user-gold-cdn.xitu.io/2018/4/13/162bc91509578896?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)首先明白`模版`的概念。一个模版就是一个项目的样板，包含项目的完整结构和信息。模版的信息都存放在一个叫做`templates.json`的文件当中。用户可以通过命令行对`templates.json`进行添加、删除、罗列的操作。通过选择不同的模版，SCION会自动从远程仓库把相应的模板拉取到本地，完成项目的搭建。最终整个脚手架的文件结构如下：`=================  |__ bin    |__ scion  |__ command    |__ add.js    |__ delete.js    |__ init.js    |__ list.js  |__ node_modules  |__ package.json  |__ templates.json`

# 入口文件

### 首先建立项目，在`package.json`里面写入依赖并执行`npm install`：`"dependencies": {    "chalk": "^1.1.3",    "co": "^4.6.0",    "co-prompt": "^1.0.0",    "commander": "^2.9.0"  }`在根目录下建立`\bin`文件夹，在里面建立一个无后缀名的`scion`文件。这个`bin\scion`文件是整个脚手架的入口文件，所以我们首先对它进行编写。首先是一些初始化的代码：`#!/usr/bin/env node --harmony'use strict' // 定义脚手架的文件路径process.env.NODE_PATH = __dirname + '/../node_modules/'const program = require('commander') // 定义当前版本program    .version(require('../package').version )// 定义使用方法program    .usage('')`从前文的架构图中可以知道，脚手架支持用户输入4种不同的命令。现在我们来写处理这4种命令的方法：`program    .command('add')    .description('Add a new template')  .alias('a')  .action(() => {    require('../command/add')()  })program    .command('list')    .description('List all the templates')    .alias('l')    .action(() => {        require('../command/list')()    })program    .command('init')    .description('Generate a new project')  .alias('i')  .action(() => {    require('../command/init')()  })program    .command('delete')    .description('Delete a template')    .alias('d')    .action(() => {        require('../command/delete')()    })``commander`的具体使用方法在这里就不展开了，可以直接到[官网](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Ftj%2Fcommander.js%2F)去看详细的文档。最后别忘了处理参数和提供帮助信息：`program.parse(process.argv)if(!program.args.length){  program.help()}`完整的代码请看[这里](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fjrainlau%2Fscion%2Fblob%2Fmaster%2Fbin%2Fscion)。使用`node`运行这个文件，看到输出如下，证明入口文件已经编写完成了。`Usage: scion   Commands:    add|a      Add a new template    list|l     List all the templates    init|i     Generate a new project    delete|d   Delete a template  Options:    -h, --help     output usage information    -V, --version  output the version number`

# 处理用户输入

### 在项目根目录下建立`\command`文件夹，专门用来存放命令处理文件。在根目录下建立`templates.json`文件并写入如下内容，用来存放模版信息：`{"tpl":{}}`

### 添加模板

### 进入`\command`并新建`add.js`文件：`'use strict'const co = require('co')const prompt = require('co-prompt')const config = require('../templates')const chalk = require('chalk')const fs = require('fs')module.exports = () => { co(function *() {   // 分步接收用户输入的参数   let tplName = yield prompt('Template name: ')   let gitUrl = yield prompt('Git https link: ')   let branch = yield prompt('Branch: ')       // 避免重复添加   if (!config.tpl[tplName]) {     config.tpl[tplName] = {}     config.tpl[tplName]['url'] = gitUrl.replace(/[\u0000-\u0019]/g, '') // 过滤unicode字符     config.tpl[tplName]['branch'] = branch   } else {     console.log(chalk.red('Template has already existed!'))     process.exit()   }      // 把模板信息写入templates.json   fs.writeFile(__dirname + '/../templates.json', JSON.stringify(config), 'utf-8', (err) => {     if (err) console.log(err)     console.log(chalk.green('New template added!\n'))     console.log(chalk.grey('The last template list is: \n'))     console.log(config)     console.log('\n')     process.exit()    }) })}`

### 删除模板

### 同样的，在`\command`文件夹下建立`delete.js`文件：`'use strict'const co = require('co')const prompt = require('co-prompt')const config = require('../templates')const chalk = require('chalk')const fs = require('fs')module.exports = () => {    co(function *() {        // 接收用户输入的参数        let tplName = yield prompt('Template name: ')        // 删除对应的模板        if (config.tpl[tplName]) {            config.tpl[tplName] = undefined        } else {            console.log(chalk.red('Template does not exist!'))            process.exit()        }                // 写入template.json        fs.writeFile(__dirname + '/../templates.json', JSON.stringify(config),     'utf-8', (err) => {            if (err) console.log(err)            console.log(chalk.green('Template deleted!'))            console.log(chalk.grey('The last template list is: \n'))            console.log(config)            console.log('\n')            process.exit()        })    })}`

### 罗列模板

### 建立`list.js`文件：`'use strict'const config = require('../templates')module.exports = () => {     console.log(config.tpl)     process.exit()}`

# 构建项目

### 现在来到我们最重要的部分——构建项目。同样的，在`\command`目录下新建一个叫做`init.js`的文件：`'use strict'const exec = require('child_process').execconst co = require('co')const prompt = require('co-prompt')const config = require('../templates')const chalk = require('chalk')module.exports = () => { co(function *() {    // 处理用户输入      let tplName = yield prompt('Template name: ')      let projectName = yield prompt('Project name: ')      let gitUrl      let branch    if (!config.tpl[tplName]) {        console.log(chalk.red('\n × Template does not exit!'))        process.exit()    }    gitUrl = config.tpl[tplName].url    branch = config.tpl[tplName].branch    // git命令，远程拉取项目并自定义项目名    let cmdStr = `git clone ${gitUrl} ${projectName} && cd ${projectName} && git checkout ${branch}`    console.log(chalk.white('\n Start generating...'))    exec(cmdStr, (error, stdout, stderr) => {      if (error) {        console.log(error)        process.exit()      }      console.log(chalk.green('\n √ Generation completed!'))      console.log(`\n cd ${projectName} && npm install \n`)      process.exit()    })  })}`可以看到，这一部分代码也非常简单，关键的一句话是`let cmdStr = `git clone ${gitUrl} ${projectName} && cd ${projectName} && git checkout ${branch}``它的作用正是从远程仓库克隆到自定义目录，并切换到对应的分支。熟悉git命令的同学应该明白，不熟悉的同学是时候补补课啦！

# 全局使用

### 为了可以全局使用，我们需要在`package.json`里面设置一下：`"bin": {    "scion": "bin/scion"  },`本地调试的时候，在根目录下执行`npm link`即可把`scion`命令绑定到全局，以后就可以直接以`scion`作为命令开头而无需敲入长长的`node scion`之类的命令了。现在我们的脚手架工具已经搭建好了，一起来尝试一下吧！

# 使用测试

### add | a 添加模版命令![img](data:image/svg+xml;utf8,)init | i 生成项目命令![img](https://user-gold-cdn.xitu.io/2018/4/13/162bc9158867f3db?imageslim)delete | d 删除模版命令 和 list | l 罗列模版命令![img](https://user-gold-cdn.xitu.io/2018/4/13/162bc91562ecbe6b?imageslim)大功告成啦！现在我们的整个脚手架工具已经搭建完成了，以后只需要知道模板的git https地址和branch就可以不断地往SCION上面添加，团队协作的话只需要分享SCION的`templates.json`文件就可以了。