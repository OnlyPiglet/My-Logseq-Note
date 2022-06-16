- 【】
- ## git 不同commit版本之间文件内容差异比较
- ```sh
  git diff 297b648acecbf37ff3dccb5dc8a060cc3a7a2b70  92bb275f262dee64978fe7f5fa1bb4978113f790 --name-only | grep -v '^dashboard/' | xargs git diff 297b648acecbf37ff3dccb5dc8a060cc3a7a2b70  92bb275f262dee64978fe7f5fa1bb4978113f790
  ```
-
- ```shell
  ##一个 feature 开发完成后 发现需要一些小的修改 同时希望保持上一次的commit id 需要修改可以直接使用
  git add .
  ## 直接将修改 合并到上次commit，并使用上次的提交信息
  git commit --amend --no-edit
  ```
-
- ```shell
  ##github 开源项目，fork - clone - commit - pull/request
  ## 新建issue
  
  ## 1. fork opensource project first
  
  ## 2. 下载fork后的项目
  git clone ....
  
  ## 关联原有项目 
  git remote add upstream https://github.com/xxx.git
  
  ## 3. 基于issue新建分支
  
  ## 4. 切换至issue分支
  
  ## 5. 修改代码 add,commit代码
  
  ## 6. 
  
  ```