# GitHub Actions to Free Disk Space on Ubuntu runners

A customizable GitHub Actions to free disk space on Ubuntu GitHub Actions runners.

## Example

```yaml
name: Free Disk Space (Ubuntu)
on: push

jobs:
  free-disk-space:
    runs-on: ubuntu-latest
    steps:

    - name: Free Disk Space (Ubuntu)
      uses: jlumbroso/free-disk-space@main
      with:
        # all of these default to true, but feel free to set to
        # false if necessary for your workflow
        android: true
        dotnet: true
        haskell: true
        large-packages: true
        swap-storage: true
```

