### Github link local repo to new remote repo

- Create Repo on GitHub
- Run
  ```bash
  git init
  git add .
  git commit -m "Initial commit"
  ```
  
- Connect to Remote Repo using SSH link and verify 
  ```bash
  git remote add origin git@github.com:berislav-vidakovic/ChatAppJn.git
  git remote -v
  ```

- Push code (--set-upstream the same -u flag)
  ```bash  
  git push -u origin main
  ```