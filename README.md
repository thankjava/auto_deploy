# auto_deploy 基于 node gulp及watch组件

    自动部署工程服务，上传code到目标服务器，执行远程ssh重启应用（用于实际开发中部署各个环境的服务，减少部署成本）
    Gulp介绍（中文）： http://www.gulpjs.com.cn/ 
    github（英文）： https://github.com/gulpjs/gulp   
---
### 背景
    在开发node端项目时接触到gulp，node端的代码部署方式非常简单，由于node是脚本非编译类型的代码，
    所以每次需要更新开发测试环境的代码时，只需要在本地执行一条gulp deployDev（配置的任务名），则将自动将本地代码push
    到服务器上。那么对于咱们部署代码就还差一步就完成了，那就是再重启服务。美好的事情就是node pm2 
    https://www.npmjs.com/package/pm2 （pm2 是一个带有负载均衡功能的Node应用的进程管理器.）实现了watch
    （监听文件变化的功能），它能发现code发生变化后自动重启应用。那么所有的部署最终变成了一句命令，你只需要把你
    的代码push到服务器上那么pm2就自动重启了，部署就那么那么的方便简单。对于后端java应用，由于我们是新应用我们
    要从无部署到各个环境很耗时，很烦，每次都要耗费大量时间，那么为什么我们不能利用gulp思想，node的部署方式实现
    push代码就完事，服务器什么的自动给我重启就好了？
---
### 构思
    对比node部署和java部署
    1.node不需要编译 java需要编译，也就是说java需要push到服务器上的资源是有选择性的有用的jar包
    2.node端的所有代码上传的资源其实是在自己电脑上（有一个相当集中的source管理位置）java的code也在自己电脑上
      当每次上传服务器需要ssh到不同的服务器，更有双节点的服务器。例如：不得不把相同的代码放在gray环境的两台不同的
      服务器上，然后重启，实际上做的是同样的重复动作
    3.node有pm2 管理自己的应用 java只能手动重启
    解决node 与 java 部署的差异
    1.针对于上述1 我们可以选择性的更新需要更新的jar和配置文件 node gulp每次实际上是全量把本地的文件全部push上去的
    2.针对于上述2我们可以抽出一台单独的服务器作为所有环境的code管理，这台机器上有我们所有环境的代码，分布在有规律的文件夹
    3.node 虽然有pm2重启，但是java项目都有写好的startup.sh和shoudown.sh 脚本重启应用，只需要远程执行下他们就可以
---
### 实现 auto_deploy
    核心功能
    1.watch指定的文件位置，如果发生变化让upload模块执行上传
    2.upload模块在发现watch告知有文件需要上传时按照配置将变更的文件分类并push到每台应用机器
    3.执行远程shell 重启目标服务器的应用（从某种意义上说只要能远程重启目标应用都可以用auto_deploy管理）
---
### auto_deploy能做什么
    1． 统一的code资源管理机器，ssh永远只需要上一台机器，再也不用去找不同环境的机器每个每个上传
    2． code一次上传根据配置全量服务器更新，通过配置文件可以告诉auto_deploy这次资源变动需要去更新哪些服务器
    3． 更简单的配置文件变更，gray环境要该配置？没事ssh到auto_deploy，vi需要变动的文件，完事后你可以做自己的事了，
    是的它会给你push到gray配置的两台机器中，并且重启，你甚至不用push config到auto_deploy的机器上，你需要在
    auto_deploy机器上vi一下
    4． 一次性同步各个环境code ，dev环境和gray代码不同？也和test环境代码不同？我也不确定哪个环境的代码是最新的了，
    我得挨个去部署这dev/test/gray环境的代码？你可以在auto_deploy中的正确位置去建立一个all的目录，然后通过配置文件
    告知all这个文件夹变更后给我把dev/test/gray都给我统统部署了。（*注意针对于这种一次push更新不同环境的一定只能有code，
    不然配置文件几个环境都变成一套了）
