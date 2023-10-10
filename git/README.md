# git版本控制
版本控制最主要的功能就是追踪文件的变更。
- 工作区(Working Directory)
  - 就是我们能访问的文件系统
- 暂存区(Stage)
  - add之后的位置
- 版本库(Repository)
  - commit之后的位置
- 远程版本库（remote）
  - 集中管理的代码仓库

## 本地版本
创建仓库的过程其实就是将一个普通的文件夹升级成一个具备版本控制功能的文件夹。``git init``实际上git命令在文件夹下创建了一个.git文件夹，版本控制功能的实现，就在这个文件夹内。默认这个文件夹是隐藏不显示的。

### 将文件添加到暂存区
创建一个**bendiwenjia1.js**的文件，随便输入点什么，然后执行``git add bendiwenjia1.js``就可以将其添加到暂存区。一次性添加多个``git add .``

### 查看工作区的状态
``git status``就可以看到变动信息了。

### 从暂存区中恢复文件
一旦提交到版本库，git就会形成快照缓存，如果我们吧文件删掉了，也可以恢复。
``git checkout bendiwenjia1.js``

### 取消跟踪
``git rm --cached bendiwenjia1.js``状态改为未add

### 忽略文件.gitignore
如果你希望在使用 add .的时候忽略某一个文件可以在目录下创建一个 .gitignore文件。  
例如我们要忽略``dist.js``  
创建一个.gitignore的文件输入dist.js即可。

### 提交代码commit
一般是完成某一阶段开发，或者bug修复等情况下，可以创建一次代码提交
``git commit -m 'initapp'``  
将工作区的代码提交到本地仓库中
### 查看工作空间状态
``git status``
### 保存临时工作成果Stash
处于各种原因，可能需要保存现在的工作区的工作情况，去做另一件事，但是又不想提交，就可以用这个功能。
``git stash``报错工作现场到栈中。需要继续之前的工作时使用``git stash pop``，从栈中弹出报错的工作空间现场。但是这种方式可能存在冲突的可能。

### 查看提交记录
- ``git log``
  - 查看历史提交记录
- ``git log --oneline``
  - 查看历史记录的简洁的版本
- ``git blame <file>``
  - 以列表形式查看指定文件的历史修改记录
### 放弃修改
- ``git restore --staged  <file>...``
  - 将暂存区的文件从暂存区撤出，但不会更改工作区的内容。

### 回退到上一个提交
git reset 命令用于回退版本，可以指定退回某一次提交的版本。
``git reset [--soft | --mixed | --hard] [HEAD]``  
--mixed 为默认  
``git reset HEAD^``重置暂存区的文件与上一次的提交(commit)保持一致（撤销commit和add操作，但是保留了修改的文件）不更新工作区  
``git reset --soft HEAD^``回退到指定的上一次版本库（回退commit，会保留add操作）  
``git reset --hard HEAD^``不但版本回退，也会更新工作区的文件到上一次版本（工作区、暂存区、版本库都会改变）  
``HEAD^``也可以指定版本hash值，也可以使用多个``^``符号表示第前几个版本。也可以使用数字表示：``HEAD~1 ``  
``git reset --hard origin/master`` 将本地的状态回退到和远程的一样
#### git reset --hard
用于将当前分支的HEAD指针重置到指定的提交，并清空暂存区和工作目录的改动。这个命令会丢弃所有未提交的修改
#### git reset --soft
用于将当前分支的 HEAD 指针重置到指定的提交，但保留未提交的修改。这个命令可以使你撤销之前的提交，并将修改保留在暂存区和工作目录中。

## 分支管理
一般在多人协作开发的项目中使用的多，便于管理。
### 创建分支
默认情况下我们处在主分支master。  
- ``git branch (branchname)``创建分支
- ``git checkout (branchname)``切换分支
- ``git checkout -b (branchname)``创建切换分支
- ``git branch -D (branchname)``删除分支
  - 如果需要同步代码 后面可以跟上commit hash
- ``git push origin (branchName)``创建远程分支(本地分支push到远程)
- ``git branch``列出分支``git branch -a``列出分支包括远程的分支
- ``git merge ``分支合并

### 清洗提交历史 -- squash方式合并
可能在做某个功能的时候commit了很多次，不希望这些中间commit被看到，这个时候我们可以用``git status``查看一下发现合并代码后并没有提交。只是将所有的提交都整合到了一起放在暂存区。然后commit。

### 标签管理
给自己的代码打上版本标记。
- ``git tag v1.0``将最新提交打标签
- ``git tag v0.9 4ab034``将指定commit打标签
- ``git tag``查看打标签
- ``git tag -d v1.1``删除标签
- ``git show v0.9``查看与某标签之间的差距（修改的内容）

## 远程仓库
- ``git remote -v``查看远程仓库
- ``git remote add [shortname] [url]``shortname 为本地的版本库
- ``git remote rm name``  删除远程仓库
- ``git remote rename old_name new_name``  修改仓库名
- ``git push``推送到远程
- ``git fetch <源>``拉去，之后需要merge合并
- ``git pull``相当与先fetch + merge
  - 例如：把本地的仓库源设置成GitHub的让后推送
  - ``git remote add origin git@github.com:***********``
  - ``git pull origin dev``
  - ``git push -u origin master``

## 常见问题
- 默认不区分文件名大小写，如果需要修改已提交的代码文件名大小写
  - ``git mv -f oldfile newfile``
  - 不推荐使用``git config core.ignorecase false``
- 修改已经push的commit msg
  - ``git rebase -i HEAD~5``
    - 读取出最近的五次
      - 不输入``HEAD~5``默认最新的一次
    - 找到需要修改的一个或多个提交
    - 将对应pick改成r。 :wq
      - pick：保留该commit（缩写:p）
      - reword：保留该commit，但我需要修改该commit的注释（缩写:r）
      - edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）
      - squash：将该commit和前一个commit合并（缩写:s）
      - fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）
      - exec：执行shell命令（缩写:x）
      - drop：我要丢弃该commit（缩写:d）
        - 会修改文件！！！
    - 后面会按顺序(最先的为最早的)自动进入需要编辑的提交信息框，编辑后:wq保存即可
  - ``git push --force``
- 解决冲突``git rebase``之``abort``、``continue``、``skip``
  - ``git pull --rebase``与远程代码同步，同步过程会检查冲突
    - ``git pull --rebase``执行过程中会将本地当前分支里的每个提交(commit)取消掉，然后把将本地当前分支更新为最新的"origin"分支
      - ``git pull``的默认行为是``git fetch`` + ``git merge``
      - ``git pull --rebase``则是``git fetch + git rebase``
    - 执行完``git pull --rebase``之后如果有合并冲突
      - ``git rebase --abort``
        - 会放弃合并，回到rebase操作之前的状态，之前的提交的不会丢弃。简单来说就是撤销``rebase``。
      - ``git rebase --skip``
        - 会将引起冲突的commits丢弃掉
      - ``git rebase --continue``
        - 合并冲突
        - 结合"git add 文件"命令一起用与修复冲突，提示开发者，一步一步地有没有解决冲突。
        - 修改后检查没问题，使用``rebase continue``来合并冲突。
  - ``git fetch``从远程获取最新版本到本地，不会自动合并分支
  - ``git rebase``重新定义分支的版本库状态
- github出现22端口报错``git-ssh: connect to host github.com port 22``
  - 可以尝试使用443端口
  - ``ssh -T -p 443 git@ssh.github.com``
  - 编辑 ~/.ssh/config 文件
  - ```
    Host github.com
    Hostname ssh.github.com
    Port 443
    ```
### 关于GitHub
- Watch
  - 表示关注、这个项目的所有动态包括PR ISSUE你的信息中心和邮箱都会收到。
- Star
  - 表示点赞、表示对这个项目支持。会添加到star列表方便后续查看
- Fork
  - 相当于复制了目前项目的文件，后续变化必须手动更新。

### npm/yarn
- 安装yarn
  - ``npm install -g yarn``
- 查看源
  - ``npm get registry ``
  - ``yarn config get registry``
- 设置为淘宝镜像
  - ``npm config set registry http://registry.npm.taobao.org/``
  - ``yarn config set registry http://registry.npm.taobao.org/``
- 设置回默认的官方镜像
  - ``npm config set registry https://registry.npmjs.org/``
  - ``yarn config set registry https://registry.yarnpkg.com``

### pnpm
将依赖包存放在统一的位置共享，节约磁盘空间优化安装速度。  
- 安装``npm install -g pnpm``
- 升级``pnpm add -g pnpm``
### webpack
[详见 webpack](https://github.com/zxlfly/webpack_share)
### Yarn Workspaces
[详见 article](https://github.com/zxlfly/article/blob/bca0d073295c142a00d19ca15a6c04e271b51ed2/00.%E9%9A%8F%E7%AC%94/1.yarn%20workspace.md)
### nvm
- https://github.com/coreybutler/nvm-windows/releases
- nvm install <version>      安装制定版本的node 
- nvm list available 查看网络可以安装的版本

**设置代理(nvm\settings.txt)**  
```
node_mirror:https://npm.taobao.org/mirrors/node/
npm_mirror:https://npm.taobao.org/mirrors/npm/
```
