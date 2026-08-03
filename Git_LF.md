## Git LF concept  
Git LFS (Large File Storage) works by keeping big files out of your actual Git history. \
When you track a file with LFS, Git stores only a small text pointer in the repo — a few lines containing a hash and file size — while the real file content gets uploaded to a separate LFS storage server (GitHub, GitLab, and Bitbucket all provide one alongside the repo).\
When someone clones or checks out the repo, the LFS client sees the pointers and downloads the actual files from that storage.\
The point of this is that Git itself is terrible with large binaries: every version of every file lives in the repo history forever, so a 500 MB model committed ten times bloats the repo by 5 GB for everyone who clones it, forever.\
LFS avoids that because you only download the versions you check out.

## How to set it up
```
git lfs install
git lfs track "*.pt" "*.onnx" "*.h5"   # whatever formats you use
git add .gitattributes
git add model.pt
git commit -m "Add model"
```
