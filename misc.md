# Misc

1. publish to pypi

   ```bash
   python3 setup.py sdist bdist_wheel
   twine check dist/*
   twine upload dist/*
   ```

2. get some files from another branch, [stackoverflow](https://stackoverflow.com/questions/2364147/how-to-get-just-one-file-from-another-branch)

   ```bash
   git switch main
   git restore --source dev -- some.file 
   ```