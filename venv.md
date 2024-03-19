# venv

### create virtualenv

```
python3 -m venv .venv && source .venv/bin/activate
```

### Install packages

```
pip install -r requirements.txt
```

# Conda

### List conda-installed packages for `pip3 install -r requirements.txt`

```
pip list --format=freeze > requirements.txt
```
