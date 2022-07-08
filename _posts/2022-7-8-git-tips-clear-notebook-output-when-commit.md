---
layout: post
title: "Git tips: Clear jupyter notebook output using config and attributes"
---

To make the version control more efficient, we don't want git to track the output of jupyter notebooks every time a notebook is committed. If the output is indeed necessary we can always convert an HTML as a report and keep it static.

To clear output automatically after staging, first modify the `.git/config` file by adding the following lines:  
```
[filter "strip-notebook-output"]
    clean = "jupyter nbconvert --ClearOutputPreprocessor.enabled=True --to=notebook --stdin --stdout --log-level=ERROR %f"
```

Then create the `.gitattributes` file under project folder with content:
```
*.ipynb filter=strip-notebook-output
```
Note that the `.gitattributes` file is version controlled while the `.git/config` file is not, thus collaborators need to manually edit it.

Referenced from [this blog post](https://zhauniarovich.com/post/2020/2020-10-clearing-jupyter-output-p3/) and an update in [this stackoverflow post](https://stackoverflow.com/questions/28908319/how-to-clear-jupyter-notebooks-output-in-all-cells-from-the-linux-terminal). 
