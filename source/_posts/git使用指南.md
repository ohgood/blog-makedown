---
title: git使用指南
date: 2016-11-01 01:50:47
tags: ["git", "工具"]
categories: git
---
![img](http://7xpm82.com1.z0.glb.clouddn.com/img/%E6%91%98%E8%A6%81%E5%9B%BE%E7%89%87/Git.png)
<!-- more -->
Here is an instruction how to commit/push your changes to git repo:  
1.Before changing anything locally create a new branch: “git branch dev/youralias/yourfeature” (i.e. dev/alshtin/AddUnitTests). Checkout new branch using “git checkout dev/youralias/yourfeature”.

2.Make all local changes.

3.Run “git status”. You will see all changed files in red. Add them to the stage: “git add *”.

4.Run “git status” again. All new/updated/deleted files should be green now.

5.Run “git commit -m “Your commit message””.

6.Run “git status” to be sure that there is nothing left.

7.Run “git push --set-upstream origin dev/youralias/yourfeature”.

8.Now your changes are on the server and we are coming to most exciting part. Web portal that I’ve shown you before is not finished and we can’t use it to submit a pull request. Instead we must use CodeFlow. First, configure it with command: “git config --unset gitcf.squashmergepullrequest” and then run it with “git cf”. It will install CodeFlow and run it (you may need to run it second time to actually run it). In CodeFlow you will have exactly the same experience as you have now. BuddyBuild wil run automatically, at least one human reviewer is required, and you need to approve checkin when everything is done. This will complete pull request and merge you changes with master branch.

9.New build will be queued for your commit at: http://b/queue/MSASG_EntityPlatform.

10.Then you can go to list of completed pull requests: https://msasg.visualstudio.com/DefaultCollection/Bing_and_IPG/_git/EntityPlatform/pullrequests?_a=completed, open your last one, and click “Delete source branch”. This will delete your dev branch from server and changes will stay in master branch only.

11.Get you changes back in master branch: “git checkout master”, “git pull”.

} while(hired) // “do {“ was before step 1. J 
 
git reset --hard HEAD      回滚操作命令

步骤详解：
打开init.cmd - Shortcut工具，敲命令：
1. git status
2. git branch   看在哪个分支下：master(一开始进入的都是主分支master)
3. git pull       必须在master下pull   
4. cls          清屏
5. git branch dev/v-conzh/peoplemappingFiles     create a new branch
   新建一个： git branch dev/v-conzh/peoplethreemappings
                           git branch dev/v-conzh/facebook
6. git branch
7. git checkout dev/v-conzh/peoplemappingFiles       Checkout new branch
8. git branch             看当前在哪个分支

9. git merge master    在自己的分支下merge 把当前分支merge为最新的。---->至关重要的一步
10. git status     to be sure that there is nothing left.
11. 将要上传的mapping文件放到该固定路径下：（三个文件全部发的时候，只需要分别在对应文件夹下传上去即可。）
D:\gitrepos\EntityPlatform\private\src\SatoriExtraction\ERMappingRepository\MappingUnitTest\Resources\Mapping
D:\gitrepos\EntityPlatform\private\src\SatoriExtraction\ERMappingRepository\MappingUnitTest\Resources\Input
D:\gitrepos\EntityPlatform\private\src\SatoriExtraction\ERMappingRepository\MappingUnitTest\Resources\Output
注意：做改动的地方（关于文件的改动）只能在这10、11的两步中做！！！

12. 敲git status   会看到一串红色代码--->see all changed files in red     表示：   标记的红色文件做了改动
13. git add  *   
14. git status     会看到一串绿色代码--->All new/updated/deleted files should be green now        to be sure that there is nothing left.

15. git commit -m "people_famoushookups"           提交到本地仓库   其中，引号里的内容是说明注释,应该写project link,model link
16. git push --set-upstream origin dev/v-conzh/peoplethreemappings      把本地仓库推送到远程仓库中
17. git cf            打开code flow
18. 如果是第一次新建branch，则输入0，如果不是，如在一样的分支下重新上传新的mapping文件，则输入1
19. 点击publish，选择Required Reviews和Optional Reviews，点击publish即可
 
 
 
Code Flow Failed之后，回滚到之前的操作，敲击命令如下：
1. git branch
2.git fetch origin
3.git reset --hard origin/master
4.git branch(如果分支不为master，则继续步骤5.6.)
5.git checkout master(转换为主分支：master)
6.git pull(拉取最新的code)

重新上传，Code Flow:
7.（可选） git branch dev/v-conzh/peoplemappingFiles    create a new branch
8. git branch
9. git checkout dev/v-conzh/peoplemappingFiles      Checkout new branch-------->转换分支的命令
10. git branch             看当前在哪个分支
11. git merge master    在自己的分支下merge 把当前分支merge为最新的。---->至关重要的一步
12. git status     to be sure that there is nothing left.
13. 将要上传的mapping文件放到该固定路径下：
D:\gitrepos\EntityPlatform\private\src\SatoriExtraction\ERMappingRepository\MappingUnitTest\Resources\Mapping
注意：做改动的地方（关于文件的改动）只能在这12、13的两步中做！！！

14. 敲git status   会看到一串红色代码--->see all changed files in red     表示：   标记的红色文件做了改动
15. git add  *   
16. git status     会看到一串绿色代码--->All new/updated/deleted files should be green now        to be sure that there is nothing left.

17. git commit -m "people_famoushookups"           提交到本地仓库   其中，引号里的内容是说明注释,应该写project link,model link
18. git push --set-upstream origin dev/v-conzh/peoplemappingFiles      把本地仓库推送到远程仓库中
19. git cf            打开code flow
20. 如果是第一次新建branch，则输入0，如果不是，如在一样的分支下重新上传新的mapping文件，则输入1
21. 点击publish，选择Required Reviews和Optional Reviews，点击publish即可
