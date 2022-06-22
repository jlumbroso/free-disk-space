# free-disk-space
A customizable GitHub Actions to free disk space on Ubuntu GitHub Actions runners.

## Example

```yaml
name: Free Disk Space
on: push

jobs:
  free-disk-space:
    runs-on: ubuntu-latest
    steps:

    - name: Free Disk Space
      uses: jlumbroso/free-disk-space@main
```

