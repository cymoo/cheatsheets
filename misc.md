# Misc

1. publish to pypi

   ```bash
   python3 setup.py sdist bdist_wheel
   twine check dist/*
   twine upload dist/*
   ```
