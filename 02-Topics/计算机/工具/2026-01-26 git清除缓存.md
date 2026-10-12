我做了以下操作：

1. **
    
    ```
    git rm --cached
    ```
    
    ** - 从 Git 缓存中移除 pth 文件（本地文件保留）
2. **
    
    ```
    git commit --amend
    ```
    
    ** - 修正上一个 commit，包含 .gitignore 的更新和移除 pth 文件
3. **
    
    ```
    git push --force-with-lease
    ```
    
    ** - 强制推送修正后的 commit

现在 

transformer_model.pth 不再被 Git 跟踪，以后也不会被误上传了