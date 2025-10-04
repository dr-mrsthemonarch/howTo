You have the remote repo URL (e.g. git@github.com:you/repo.git or https://github.com/you/repo.git).

Git is installed: git --version.

You’ve configured your name/email (if not):
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

A — If your local folder is not a git repo yet

Open a terminal and go to your project folder:

cd /path/to/your/project


Initialize a repo (create the branch main as the initial branch):

git init -b main


If your Git is older and -b isn't supported:
git init then git branch -M main

Create a .gitignore (important to avoid committing secrets, build artifacts, node_modules, etc.). Minimal example:

# .gitignore
node_modules/
.env
dist/
build/
.DS_Store


Create it with:

cat > .gitignore <<'EOF'
node_modules/
.env
dist/
build/
.DS_Store
EOF


Stage and commit your files:

git add .
git commit -m "Initial commit: add project files"


Add the remote repository and push:

git remote add origin <REMOTE_URL>
git remote -v   # confirm remote is set correctly
git push -u origin main


Replace <REMOTE_URL> with your remote (SSH or HTTPS).

If git push is rejected (remote already has commits), choose one:

Merge remote into local (safe, preserves both histories):

git pull origin main --allow-unrelated-histories
# resolve conflicts if any, then:
git add .
git commit   # finish the merge commit if necessary
git push -u origin main


Rebase local commits on top of remote (neater linear history):

git fetch origin
git rebase origin/main
# resolve conflicts, then:
git rebase --continue
git push -u origin main


Force overwrite remote (destructive — do NOT do this unless you know remote can be replaced):

git push -u origin main --force-with-lease


--force-with-lease is safer than --force.

B — If your local folder is already a git repo

Go to your repo and check status:

cd /path/to/repo
git status
git remote -v


If no remote named origin exists, add it:

git remote add origin <REMOTE_URL>
git remote -v


If origin exists but points to the wrong URL:

git remote set-url origin <REMOTE_URL>


Ensure you’re on the branch you want to push (example: main):

git branch         # see current branch
git switch -c main # create+switch if you need a main branch
# or rename current branch to main:
git branch -M main


Stage & commit any uncommitted local files:

git add .
git commit -m "Add local source files"


Integrate remote changes (recommended) and push:

git fetch origin
git pull --rebase origin main   # keeps a linear history
# resolve any conflicts, then:
git push -u origin main


If histories are unrelated and pull --rebase fails, use:

git pull origin main --allow-unrelated-histories
# resolve conflicts, commit, then:
git push -u origin main

C — If you prefer to push to a new branch (safer)
git checkout -b add-local-files
git add .
git commit -m "Add local source files"
git push -u origin add-local-files


Then open a Pull Request on your Git host to merge into main.

Example — full common flow (local not git, remote empty)
cd ~/projects/my-app
git init -b main
cat > .gitignore <<'EOF'
node_modules/
.env
dist/
EOF
git add .
git commit -m "Initial commit: app files"
git remote add origin git@github.com:youruser/yourrepo.git
git push -u origin main
