---
layout: post
title: "Dash Tips: Deployment on Rstudio Connect"
---

## Deploy to Rstudio Connect from Git

[Rstudio Connect](https://www.rstudio.com/products/connect/) is a handy option to deploy light weight data science / bioinfo applications. To deploy an app, you can either push the project by command line `rsconnect deploy ...` or import from a git repo. The latter is a better choice in my opinion. Importing from git repo seperates development and production. Manual push is also spared since Rsconnect can automatically pull the latest commit from the configured branch. You can also take advantage of the CI workflow (gitlab) / actions (github).

## Prepare manifest.json

A *manifest.json* file is required for importing from git. For a python project, first `pip install rsconnect-python`. In the package a command `rsconnect write-manifest` can be used to create the file.

The *manifest.json* file looks like:
```
{
  "version": 1,
  "metadata": {
    "appmode": "python-dash",
    "entrypoint": "app:app"
  },
  "locale": "en_US.UTF-8",
  "python": {
    "version": "3.9.7",
    "package_manager": {
      "name": "pip",
      "version": "22.1.2",
      "package_file": "requirements.txt"
    }
  },
  "files": {
    "requirements.txt": {
      "checksum": "3a3589259a7a29913bfdbe0c9e3db90a"
    },
    ".gitignore": {
      "checksum": "79be07b809f111a1d719e90a010f32b5"
    },
    "Makefile": {
      "checksum": "7fc801d88fa888d915b8becf5dfb466e"
    },
    "README.md": {
      "checksum": "40112eced702e7a6347d3e66117a7bc7"
    },
    "app.py": {
      "checksum": "447e27fa483ae16b8e9d310993d1cb37"
    },
    "check_manifest_files.py": {
      "checksum": "c6164389e846bf8e9add01863c527e04"
    },
    "hooks/pre-commit": {
      "checksum": "cf29b2920af18ed96f3ff2a92985934c"
    },
    "setup.py": {
      "checksum": "46d30059744ab86162e21e716e8a29db"
    }
  }
}
```

Under the project folder there can be several files not necessary to be included in the *manifest.json*, e.g. the venv folder, the vscode setting, ipynb checkpoints, test, etc. A tip here is to write the command in `Makefile`. Everytime you need to update *manifest.json*, just run `make manifest`. Here is an example Makefile (see how many files I excluded):

```bash
.PHONY: manifest
VENV := venv
manifest:
	./$(VENV)/bin/rsconnect write-manifest dash -e app:app --overwrite \
	-x "data/*" \
	-x "notebook/*" \
	-x "*/.ipynb_checkpoints/*" \
	-x ".ipynb_checkpoints/*" \
	-x "*/__pycache__/*" \
	-x "*.egg-info/*" \
	-x "dist/" \
	-x "build/" \
	-x "test/" \
	-x ".pytest_cache" \
	-x ".gitlab-ci.yml" \
	-x "LICENSE" \
	-x ".vscode" \
	-x ".coverage" \
	-x "orig_manifest.json" \
	.
```

Other steps like *build*, *test* can also be written in the Makefile. Not only reproducability can be better but also the CI workflow can be easier to setup.

Note the last file to be excluded *orig_manifest.json*, let's discuss it in next section.

## Check *manifest.json* by pre-commit hook

Sometimes we might forget to update *manifest.json* after making some code change. If we push such a commit, it's either your workflow / action fails (if you configured CI to validate *manifest.json*) or the rsconnect deployment (even worse).

Git hooks are executable files (usually shell scripts). We can use a pre-commit hook as a gatekeeper. It could give us an error message and prevent a commit if the *manifest.json* is not updated. To achieve this goal, we can create a new *manifest.json* and compare it the staged one. An example pre-commit hook (file path: *.git/hooks/pre-commit*):

```bash
#!/bin/sh
# Check if manifest.json represents the latest code change
mv manifest.json orig_manifest.json
make manifest >/dev/null 2>&1
python check_manifest_files.py orig_manifest.json manifest.json >/dev/null 2>&1
if [ $? -ne 0 ]
then
    echo "manifest.json not updated!"
    mv orig_manifest.json manifest.json
    exit 1
else
    mv orig_manifest.json manifest.json
    exit 0
fi
```

Explanation:
* Line 3: Since `rsconnect write-manifest` will always create the file named as *manifest.json*, we need to give the staged one a new name to avoid an overwrite. 
* Line 4: Use `make manifest` to simplify re-generating process.
* Line 5: Write a comparison script which focuses on checksum of files. If inconsistency is found, raise an error (code in below).
* Line 6-end: 
    * Since a git hook should not modify files, change file name back.
    * By suppressing other outputs (`>/dev/null 2>&1`), the error message (line 8) can be properly shown in alert.
    * Depending on exit code of command above, exit 0 and proceed to commit or exit 1 thus halt the commit. 

An example *check_manifest_files.py*:
```python
"""
Compare if the "files" field in the 2nd manifest.json are consistent with the 1st.
"""
import sys
import json

def load_json(fname, field="files"):
    with open(fname, 'r') as f:
        return json.load(f)[field]

if __name__ == "__main__":
    files1 = load_json(sys.argv[1])
    files2 = load_json(sys.argv[2])
    for fname in files1:
        assert files2[fname]["checksum"] == files1[fname]["checksum"]
    print("all files are consistent, ready to deploy")

```

Also note that *.git/* folder is not version controlled. We can put the *pre-commit* to *hooks/pre-commit* and make sure copy it to *.git/hooks* everytime we clone the project somewhere else.

## Example Dash Project Folder
Below is the example dash app project folder for RSConnect deployment.

```
.
├── app.py      # app entrypoint defined here
├── package     # the package for this app
│   └── ...
├── build
│   └── ...
├── check_manifest_files.py
├── hooks
│   └── pre-commit      # remember to copy it to .git/hooks/
├── LICENSE
├── Makefile
├── manifest.json
├── README.md
├── requirements.txt
├── setup.py
├── test
│   └── ...
└── venv
    └── ...
```