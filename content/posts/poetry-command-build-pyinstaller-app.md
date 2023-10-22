---
title: "Create a custom poetry command to build your Pyinstaller app"
date: 2023-10-22T00:25:38+01:00
type: "post"
tags: ["python", "poetry", "pyinstaller"]
---

If you are:
- Developing a Python desktop app 
- Using [`Pyinstaller`](https://pyinstaller.org/en/stable/index.html) to budle your app into a single executable file
- Managing your environment and packages with [`poetry`](https://python-poetry.org/)

and you want to have a single one-liner (`poetry run build`) to trigger the build process...

> ✨ Then I can help you! ✨

---

## Backstory

I was myself looking for a similar solution when I found this very pertinent
[StackOverfow question.](https://stackoverflow.com/questions/76145761/use-poetry-to-create-binary-distributable-with-pyinstaller-on-package/77181745#77181745)
In there I found some insipiration that led me to do some digging, that led me to
figure out the solution! Trying to be a good citizen, I posted it as a reply, and trying
to be an even better one, I took the time to format it nicely.

So nicely in fact that I've decided to include it here, for whoever might need it.

---

## One-liner `poetry` command to trigger the build process of your app

As you may know already, Poetry will only let you run 'scripts' if they are functions inside your package.
Just like in your `pyproject.toml`, you can map the start command to `main:start`,
which is the `start()` function of your `main.py` module.

Similarly, you can create a function in a module that triggers Pyinstaller and 
map that to a command that you can run as `poetry run <command>`.

Assuming you have a module structure like this:

```
my_package
├── my_package
│   ├── __init__.py
│   ├── pyinstaller.py
│   └── main.py
└── pyproject.toml
```

1. Create a file `pyinstaller.py` to call the Pyinstaller API

The file should be inside your package structure, as shown in the diagram above.
This is adapted from the Pyinstaller docs

```python
import PyInstaller.__main__
from pathlib import Path

HERE = Path(__file__).parent.absolute()
path_to_main = str(HERE / "main.py")

def install():
    PyInstaller.__main__.run([
        path_to_main,
        '--onefile',
        '--windowed',
        # other pyinstaller options... 
    ])
```

2. Map the build command in `pyproject.toml`

In the `pyproject.toml` file, add this

```
[tool.poetry.scripts]
build = "my_package.pyinstaller:install"
```

3. From the terminal, invoke the build command

You must do so under the virtual environment that poetry creates:

```
poetry run build
```

🎉 Profit
