# Git-S3 Implementation Checklist

## Project Setup
- [ ] Create Maven/Gradle project structure
- [ ] Add AWS SDK dependency (`software.amazon.awssdk:s3`)
- [ ] Add JSON library dependency (Jackson or Gson)
- [ ] Set up AWS credentials (AWS CLI or environment variables)

## Core Data Structures
- [ ] Create `Commit` class (hash, message, timestamp, author, parent, files)
- [ ] Create `FileEntry` class (filename, hash, content)
- [ ] Create `Repository` class (repo ID, current branch)

## Command: `git init`
- [ ] Create `.mygit/` directory
- [ ] Create subdirectories: `objects/`, `refs/`, `staging/`
- [ ] Generate unique repository ID (UUID)
- [ ] Create `config` file with repo ID
- [ ] Create `HEAD` file pointing to main branch
- [ ] Create empty `index` file for staging area

## Command: `git add <file>`
- [ ] Check if repository is initialized
- [ ] Read file content from filesystem
- [ ] Calculate SHA-1 hash of file content
- [ ] Copy file to `.mygit/staging/` with hash as filename
- [ ] Update `.mygit/index` with: `hash|filename|timestamp`

## Command: `git commit <message>`
- [ ] Check if repository is initialized
- [ ] Read all staged files from `.mygit/index`
- [ ] Get parent commit hash from current HEAD (if exists)
- [ ] Create commit object with:
  - [ ] Unique commit hash (SHA-1 of metadata + content)
  - [ ] Commit message
  - [ ] Timestamp
  - [ ] Author info
  - [ ] Parent commit hash
  - [ ] List of file hashes and names
- [ ] Move files from staging to `.mygit/objects/`
- [ ] Save commit metadata as JSON in `.mygit/objects/<commit-hash>`
- [ ] Update HEAD to point to new commit
- [ ] Clear staging area and index

## Command: `git push <s3-bucket>`
- [ ] Check if repository is initialized
- [ ] Read repository ID from config
- [ ] Get current commit hash from HEAD
- [ ] Build commit tree (current commit + all parents)
- [ ] For each commit in tree:
  - [ ] Upload commit JSON to S3: `s3://<bucket>/<repo-id>/commits/<hash>.json`
  - [ ] For each file in commit:
    - [ ] Upload file to S3: `s3://<bucket>/<repo-id>/objects/<hash>`
- [ ] Upload HEAD reference: `s3://<bucket>/<repo-id>/refs/main`
- [ ] Display S3 URL for sharing: `s3://<bucket>/<repo-id>`

## Command: `git pull <s3-bucket> <repo-id>`
- [ ] Check if repository is initialized (init if not)
- [ ] Download HEAD reference from: `s3://<bucket>/<repo-id>/refs/main`
- [ ] Get latest commit hash from HEAD
- [ ] Download commit metadata from: `s3://<bucket>/<repo-id>/commits/<hash>.json`
- [ ] Recursively download parent commits until reaching root
- [ ] For each commit:
  - [ ] Download all file objects referenced in commit
  - [ ] Save to `.mygit/objects/`
- [ ] Checkout latest commit (copy files to working directory)
- [ ] Update local HEAD

## Utility Classes
- [ ] `HashUtils` - SHA-1 hashing for files and commits
- [ ] `FileUtils` - Read/write files, copy between directories
- [ ] `S3Utils` - Upload/download from S3 with proper error handling
- [ ] `CommitUtils` - Create, serialize, deserialize commits

## AWS S3 Integration
- [ ] Create S3 client with credentials
- [ ] Implement `uploadFile(bucket, key, content)`
- [ ] Implement `downloadFile(bucket, key)`
- [ ] Implement `listObjects(bucket, prefix)`
- [ ] Handle S3 exceptions (bucket not found, access denied)

## Advanced Features (Optional)
- [ ] Support for `.gitignore` functionality
- [ ] Branch management (create, switch, merge)
- [ ] `git status` - show modified/staged files
- [ ] `git log` - display commit history
- [ ] `git diff` - show differences between commits
- [ ] Conflict detection on pull
- [ ] Clone command (alternative to pull for new repos)
- [ ] Authentication for private repositories

## Testing
- [ ] Test init in empty directory
- [ ] Test add with single file
- [ ] Test add with multiple files
- [ ] Test commit with staged files
- [ ] Test push to S3 bucket
- [ ] Test pull from S3 to new directory
- [ ] Test full workflow: init → add → commit → push → pull
- [ ] Test error cases (missing files, invalid commands, S3 errors)

## Documentation
- [ ] README with setup instructions
- [ ] AWS credentials configuration guide
- [ ] Example usage commands
- [ ] Architecture diagram
- [ ] Differences from real Git

## Learning Objectives Covered
- ✓ File I/O in Java
- ✓ Hashing and content addressing
- ✓ AWS S3 SDK usage
- ✓ JSON serialization
- ✓ Command-line argument parsing
- ✓ Object-oriented design
- ✓ Version control concepts
