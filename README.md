<!--
 * @Descripttion: 
 * @version: 
 * @Author: Piers
 * @Date: 2021-06-23 14:17:57
 * @LastEditors: Please set LastEditors
 * @LastEditTime: 2021-06-25 13:58:37
-->
## master分支

[https://aniubilityteam.github.io](https://aniubilityteam.github.io) 站点的静态资源分支，不需要在此分支做任何修改

## source分支

默认的主分支，源代码分支，所有页面内容修改都在这个分支

## source分支修改步骤

- 在/source/下新建`.md`文件记录文章
- 文章顶部需要写`tags` `categories` `description` 等内容，保证文章会自动分类 归档 缩略显示
- 本地 `hexo clean` 清除本地生成静态文件的记录
- 本地 `hexo g` 重新生成静态文件
- 本地 `hexo s` 启动静态文件，访问localhost:4000, 看到生成的效果
- 本地 `hexo d` 将静态文件部署到远端
- 前往[https://aniubilityteam.github.io](https://aniubilityteam.github.io)查看效果