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

Now, should you put your ML models in the company repo? Some things worth weighing before you do:

### Where LFS fits well: 
smallish models (tens to a few hundred MB) that change infrequently, where it's genuinely useful to have the exact model version tied to the exact code commit that produced or uses it. If your team already uses the repo as the source of truth and models are more like assets than experiment outputs, LFS is fine and simple.

### Where it hurts: 
LFS storage and bandwidth usually cost money on hosted platforms (GitHub's free tier is only 1 GB storage / 1 GB bandwidth per month, for example — worth checking what your company's plan includes). Models that change often will accumulate versions in LFS storage, and deleting old ones is awkward. CI pipelines that clone the repo will pull the models every time unless configured not to. And once large files are in the repo's history — even via LFS — untangling them later is a chore. It also affects everyone: colleagues cloning the repo for unrelated work may end up downloading your models.

For ML work specifically, many teams keep models out of Git entirely and instead use something built for the purpose: DVC (which versions models in Git-like fashion but stores them in S3/GCS/Azure), a model registry like MLflow or Weights & Biases, or plain object storage with a naming convention, with just a config file or hash reference committed to Git. That keeps the repo lean and gives you things LFS doesn't, like experiment metadata and lineage.
