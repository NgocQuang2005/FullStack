# Git - Hướng Dẫn Từ Cơ Bản Đến Nâng Cao

## Mục Lục
1. [Git Là Gì và Tại Sao Cần Dùng](#1-git-là-gì-và-tại-sao-cần-dùng)
2. [Cài Đặt và Cấu Hình Ban Đầu](#2-cài-đặt-và-cấu-hình-ban-đầu)
3. [Các Khái Niệm Cơ Bản](#3-các-khái-niệm-cơ-bản)
4. [Lệnh Git Cơ Bản](#4-lệnh-git-cơ-bản)
5. [Làm Việc Với Remote Repository](#5-làm-việc-với-remote-repository)
6. [Branch và Merge](#6-branch-và-merge)
7. [Git Workflow Thực Tế](#7-git-workflow-thực-tế)
8. [Lệnh Git Nâng Cao](#8-lệnh-git-nâng-cao)
9. [Xử Lý Conflict](#9-xử-lý-conflict)
10. [Git Hooks](#10-git-hooks)
11. [Git Submodules](#11-git-submodules)
12. [Best Practices](#12-best-practices)
13. [Troubleshooting](#13-troubleshooting)

---

## 1. Git Là Gì và Tại Sao Cần Dùng

### Git là gì?
Git là một hệ thống quản lý phiên bản phân tán (Distributed Version Control System), được tạo ra bởi Linus Torvalds vào năm 2005. Git giúp theo dõi thay đổi trong mã nguồn và cho phép nhiều người cộng tác trên cùng một dự án.

### Tại sao nên dùng Git?
- **Theo dõi lịch sử thay đổi**: Xem ai đã thay đổi gì, khi nào
- **Backup tự động**: Mọi clone đều là backup đầy đủ
- **Cộng tác nhóm**: Nhiều người làm việc cùng lúc không xung đột
- **Branching linh hoạt**: Tạo nhánh để thử nghiệm tính năng mới
- **Rollback dễ dàng**: Quay lại phiên bản cũ khi cần thiết

---

## 2. Cài Đặt và Cấu Hình Ban Đầu

### Cài đặt Git

**Windows:**
```bash
# Tải từ https://git-scm.com/download/win
```

**macOS:**
```bash
# Sử dụng Homebrew
brew install git

# Hoặc tải từ https://git-scm.com/download/mac
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install git
```

### Cấu hình ban đầu
```bash
# Cấu hình tên và email (bắt buộc)
git config --global user.name "Tên của bạn"
git config --global user.email "email@example.com"

# Cấu hình editor mặc định
git config --global core.editor "code --wait"  # VSCode
git config --global core.editor "vim"          # Vim

# Cấu hình line ending
git config --global core.autocrlf true   # Windows
git config --global core.autocrlf input  # macOS/Linux

# Kiểm tra cấu hình
git config --list
git config user.name
```

---

## 3. Các Khái Niệm Cơ Bản

### Repository (Repo)
Là thư mục chứa dự án và toàn bộ lịch sử thay đổi của nó.

### Working Directory
Thư mục làm việc hiện tại trên máy tính của bạn.

### Staging Area (Index)
Khu vực tạm thời chứa các thay đổi chuẩn bị commit.

### Commit
Một snapshot của dự án tại một thời điểm cụ thể.

### Branch
Một nhánh phát triển độc lập từ nhánh chính.

### HEAD
Con trỏ chỉ đến commit hiện tại bạn đang làm việc.

### 3 Trạng thái của file
- **Modified**: File đã thay đổi nhưng chưa được staged
- **Staged**: File đã được add vào staging area
- **Committed**: File đã được lưu vào Git database

---

## 4. Lệnh Git Cơ Bản

### Khởi tạo repository
```bash
# Tạo repo mới
git init

# Clone repo từ remote
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git my-project  # Đặt tên thư mục khác
```

### Kiểm tra trạng thái
```bash
# Xem trạng thái files
git status

# Xem trạng thái ngắn gọn
git status -s

# Xem sự khác biệt
git diff                    # So sánh working dir với staging
git diff --staged          # So sánh staging với commit cuối
git diff HEAD              # So sánh working dir với commit cuối
```

### Staging files
```bash
# Add file cụ thể
git add file.txt

# Add nhiều files
git add file1.txt file2.txt

# Add tất cả files
git add .
git add -A

# Add theo pattern
git add "*.txt"

# Remove file khỏi staging
git reset HEAD file.txt
git restore --staged file.txt  # Git 2.23+
```

### Commit
```bash
# Commit với message
git commit -m "Add new feature"

# Commit và add cùng lúc (chỉ tracked files)
git commit -am "Update existing files"

# Commit với editor
git commit

# Sửa commit cuối cùng
git commit --amend
git commit --amend -m "New message"
```

### Xem lịch sử
```bash
# Xem log cơ bản
git log

# Xem log ngắn gọn
git log --oneline

# Xem log với graph
git log --graph --oneline

# Xem log với thống kê
git log --stat

# Xem log của file cụ thể
git log -p file.txt

# Xem log theo số lượng
git log -5

# Xem log theo thời gian
git log --since="2 weeks ago"
git log --until="2023-01-01"
```

---

## 5. Làm Việc Với Remote Repository

### Quản lý remote
```bash
# Xem remote hiện tại
git remote -v

# Thêm remote
git remote add origin https://github.com/user/repo.git

# Đổi tên remote
git remote rename origin upstream

# Xóa remote
git remote remove origin

# Thay đổi URL remote
git remote set-url origin https://github.com/user/new-repo.git
```

### Push và Pull
```bash
# Push lên remote
git push origin main
git push origin feature-branch

# Push lần đầu và set upstream
git push -u origin main

# Push tất cả branches
git push --all origin

# Pull từ remote
git pull origin main

# Pull với rebase
git pull --rebase origin main

# Fetch (chỉ tải về, không merge)
git fetch origin
git fetch --all
```

---

## 6. Branch và Merge

### Quản lý branch
```bash
# Xem tất cả branches
git branch
git branch -a        # Bao gồm remote branches
git branch -r        # Chỉ remote branches

# Tạo branch mới
git branch feature-login

# Chuyển branch
git checkout feature-login
git switch feature-login    # Git 2.23+

# Tạo và chuyển branch cùng lúc
git checkout -b feature-login
git switch -c feature-login

# Đổi tên branch
git branch -m old-name new-name
git branch -m new-name  # Đổi tên branch hiện tại

# Xóa branch
git branch -d feature-login     # Safe delete
git branch -D feature-login     # Force delete

# Xóa remote branch
git push origin --delete feature-login
```

### Merge
```bash
# Merge branch vào branch hiện tại
git merge feature-login

# Merge với message custom
git merge feature-login -m "Merge feature login"

# Merge không tạo commit mới (fast-forward)
git merge --ff-only feature-login

# Merge luôn tạo commit mới
git merge --no-ff feature-login

# Hủy merge khi có conflict
git merge --abort
```

### Rebase
```bash
# Rebase branch hiện tại lên main
git rebase main

# Rebase interactive (sửa lịch sử commits)
git rebase -i HEAD~3

# Tiếp tục rebase sau khi fix conflict
git rebase --continue

# Hủy rebase
git rebase --abort
```

---

## 7. Git Workflow Thực Tế

### Gitflow Workflow

**Các branch chính:**
- `main/master`: Branch sản phẩm
- `develop`: Branch phát triển chính
- `feature/*`: Branches tính năng mới
- `release/*`: Branches chuẩn bị phát hành
- `hotfix/*`: Branches sửa lỗi khẩn cấp

**Quy trình:**
```bash
# 1. Tạo feature branch
git checkout develop
git pull origin develop
git checkout -b feature/user-authentication

# 2. Phát triển tính năng
git add .
git commit -m "Add login functionality"
git push origin feature/user-authentication

# 3. Tạo Pull Request để merge vào develop

# 4. Sau khi merge, xóa feature branch
git checkout develop
git pull origin develop
git branch -d feature/user-authentication
```

### GitHub Flow (Đơn giản hơn)

**Quy trình:**
```bash
# 1. Tạo branch từ main
git checkout main
git pull origin main
git checkout -b add-user-profile

# 2. Commit thay đổi
git add .
git commit -m "Add user profile page"
git push origin add-user-profile

# 3. Tạo Pull Request
# 4. Review và merge
# 5. Xóa branch sau khi merge
```

---

## 8. Lệnh Git Nâng Cao

### Stash (Lưu tạm thời)
```bash
# Lưu thay đổi tạm thời
git stash
git stash save "Work in progress on feature X"

# Xem danh sách stash
git stash list

# Apply stash (giữ stash)
git stash apply
git stash apply stash@{1}

# Pop stash (xóa stash sau khi apply)
git stash pop

# Xóa stash
git stash drop stash@{1}
git stash clear  # Xóa tất cả
```

### Cherry-pick
```bash
# Áp dụng commit cụ thể vào branch hiện tại
git cherry-pick abc1234

# Cherry-pick nhiều commits
git cherry-pick abc1234 def5678

# Cherry-pick range
git cherry-pick abc1234..def5678
```

### Reset và Revert
```bash
# Reset về commit trước (giữ thay đổi trong working dir)
git reset --soft HEAD~1

# Reset về commit trước (xóa khỏi staging)
git reset --mixed HEAD~1  # Mặc định
git reset HEAD~1

# Reset về commit trước (xóa hoàn toàn)
git reset --hard HEAD~1

# Revert commit (tạo commit mới để undo)
git revert abc1234
git revert HEAD    # Revert commit cuối
```

### Reflog
```bash
# Xem lịch sử HEAD
git reflog

# Recovery commit đã mất
git reflog
git checkout abc1234  # Commit ID từ reflog
git checkout -b recovery-branch
```

### Bisect (Tìm bug)
```bash
# Bắt đầu bisect
git bisect start

# Đánh dấu commit hiện tại là bad
git bisect bad

# Đánh dấu commit tốt
git bisect good abc1234

# Git sẽ checkout commit ở giữa
# Test và đánh dấu good/bad
git bisect good   # hoặc git bisect bad

# Kết thúc bisect
git bisect reset
```

---

## 9. Xử Lý Conflict

### Khi nào xảy ra conflict?
- Merge branches có thay đổi cùng một dòng
- Rebase với thay đổi trùng lặp
- Pull khi có thay đổi local chưa commit

### Xử lý merge conflict
```bash
# 1. Khi merge bị conflict
git merge feature-branch
# Auto-merging file.txt
# CONFLICT (content): Merge conflict in file.txt

# 2. Kiểm tra files có conflict
git status

# 3. Mở file và sửa conflict
# File sẽ có dạng:
# <<<<<<< HEAD
# Nội dung từ branch hiện tại
# =======
# Nội dung từ branch được merge
# >>>>>>> feature-branch

# 4. Sửa file, xóa markers, giữ lại nội dung mong muốn

# 5. Add và commit
git add file.txt
git commit  # Hoặc git commit -m "Resolve merge conflict"
```

### Tools hỗ trợ resolve conflict
```bash
# Sử dụng merge tool
git mergetool

# Cấu hình merge tool
git config --global merge.tool vimdiff
git config --global merge.tool vscode
```

---

## 10. Git Hooks

### Hooks là gì?
Hooks là scripts tự động chạy khi có các sự kiện Git cụ thể xảy ra.

### Các hooks phổ biến
- `pre-commit`: Chạy trước khi commit
- `commit-msg`: Kiểm tra commit message
- `pre-push`: Chạy trước khi push
- `post-receive`: Chạy sau khi receive push

### Ví dụ pre-commit hook
```bash
# File: .git/hooks/pre-commit
#!/bin/sh

# Chạy linter
npm run lint
if [ $? -ne 0 ]; then
  echo "Lint failed. Please fix the issues before committing."
  exit 1
fi

# Chạy tests
npm test
if [ $? -ne 0 ]; then
  echo "Tests failed. Please fix the issues before committing."
  exit 1
fi
```

### Sử dụng Husky (Node.js projects)
```bash
# Cài đặt Husky
npm install --save-dev husky

# Initialize
npx husky-init

# Thêm hook
npx husky add .husky/pre-commit "npm test"
```

---

## 11. Git Submodules

### Submodules là gì?
Submodules cho phép bạn bao gồm một Git repository khác như một subdirectory trong project của bạn.

### Thêm submodule
```bash
# Thêm submodule
git submodule add https://github.com/user/library.git lib/library

# Commit submodule
git commit -m "Add library submodule"
```

### Clone project có submodules
```bash
# Clone và initialize submodules
git clone --recursive https://github.com/user/project.git

# Hoặc clone rồi init submodules
git clone https://github.com/user/project.git
cd project
git submodule init
git submodule update
```

### Cập nhật submodules
```bash
# Update submodule to latest commit
cd lib/library
git pull origin main
cd ../..
git add lib/library
git commit -m "Update library submodule"

# Update tất cả submodules
git submodule update --remote
```

---

## 12. Best Practices

### Commit Messages
```bash
# Format tốt:
# <type>(<scope>): <subject>
# 
# <body>
# 
# <footer>

# Ví dụ:
feat(auth): add login functionality

Add user authentication with JWT tokens.
Includes login, logout, and token refresh.

Closes #123
```

**Types phổ biến:**
- `feat`: Tính năng mới
- `fix`: Sửa bug
- `docs`: Cập nhật tài liệu
- `style`: Thay đổi format, không ảnh hưởng code
- `refactor`: Refactor code
- `test`: Thêm/sửa tests
- `chore`: Cập nhật build tools, dependencies

### Branch Naming
```bash
# Format tốt:
feature/feature-name
bugfix/issue-description
hotfix/critical-bug
release/version-number

# Ví dụ:
feature/user-authentication
bugfix/login-error
hotfix/payment-security
release/1.2.0
```

### .gitignore
```gitignore
# Dependencies
node_modules/
bower_components/

# Build outputs
dist/
build/
*.exe
*.dll

# IDE files
.vscode/
.idea/
*.swp
*.swo

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Environment variables
.env
.env.local

# Database
*.db
*.sqlite
```

### Các nguyên tắc làm việc
1. **Commit thường xuyên**: Commit nhỏ, có ý nghĩa
2. **Pull trước khi push**: Đảm bảo sync với remote
3. **Review code**: Sử dụng Pull Request
4. **Test trước khi merge**: Đảm bảo code hoạt động
5. **Protect main branch**: Không push trực tiếp lên main

---

## 13. Troubleshooting

### Các lỗi phổ biến và cách khắc phục

#### 1. "fatal: not a git repository"
```bash
# Kiểm tra có phải trong Git repo không
pwd
ls -la  # Tìm thư mục .git

# Nếu chưa có, khởi tạo repo
git init
```

#### 2. "Your branch is ahead/behind"
```bash
# Branch ahead (có commit chưa push)
git push origin main

# Branch behind (cần pull)
git pull origin main

# Hoặc fetch và merge riêng
git fetch origin
git merge origin/main
```

#### 3. "Please commit your changes or stash them"
```bash
# Option 1: Commit changes
git add .
git commit -m "WIP: save current work"

# Option 2: Stash changes
git stash
# Làm việc khác...
git stash pop
```

#### 4. Accidentally committed to wrong branch
```bash
# Nếu chưa push
git reset --soft HEAD~1  # Undo commit, keep changes
git stash                # Stash changes
git checkout correct-branch
git stash pop
git commit -m "Correct commit message"
```

#### 5. Need to change commit message
```bash
# Commit cuối cùng chưa push
git commit --amend -m "New message"

# Commit đã push (cẩn thận với shared branches)
git commit --amend -m "New message"
git push --force-with-lease origin branch-name
```

#### 6. Merge conflicts
```bash
# Xem files có conflict
git status

# Cancel merge
git merge --abort

# Hoặc resolve manually rồi commit
# (xem phần 9 - Xử lý Conflict)
```

#### 7. Deleted file recovery
```bash
# Nếu file bị xóa nhưng chưa commit
git restore file.txt

# Nếu đã commit
git log --oneline  # Tìm commit trước khi xóa
git checkout abc1234 -- file.txt
```

#### 8. Remove file from Git but keep locally
```bash
git rm --cached file.txt
echo "file.txt" >> .gitignore
git commit -m "Remove file.txt from tracking"
```

### Lệnh hữu ích cho debugging
```bash
# Xem config hiện tại
git config --list

# Xem remote URLs
git remote -v

# Xem branch tracking info
git branch -vv

# Xem size của repo
git count-objects -vH

# Clean working directory
git clean -fd  # Remove untracked files and directories

# Check repository integrity
git fsck
```

---

## Kết Luận

Git là công cụ mạnh mẽ và cần thiết cho mọi developer. Việc nắm vững Git sẽ giúp bạn:

- Quản lý code hiệu quả
- Cộng tác nhóm tốt hơn
- Theo dõi và rollback thay đổi dễ dàng
- Tự tin hơn khi làm việc với các dự án lớn

Hãy thực hành thường xuyên và không ngại thử nghiệm với các lệnh Git khác nhau. Nhớ rằng Git rất mạnh mẽ nhưng cũng có thể phục hồi hầu hết các lỗi nếu bạn biết cách sử dụng.

**Lời khuyên cuối:** Bắt đầu với các lệnh cơ bản, sau đó dần dần học các tính năng nâng cao khi bạn cần thiết trong công việc thực tế.