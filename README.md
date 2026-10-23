# .NET Bootcamp Docs | جزوه‌های بوت‌کمپ دات‌نت

Persian-language lesson notes, C# examples, and exercises for a .NET bootcamp. The documentation follows the classroom sessions and helps students review concepts, run examples, and practice between classes.

**[Read the documentation →](https://mohammadhoseindehghani.github.io/dotnet-bootcamp-docs/)**

## Learning content

Start with the [C# learning path](docs/csharp/index.md), then work through the sessions in order:

| Session | Topics | Notes |
| --- | --- | --- |
| 01 | Console output, variables, data types, conversions, and user input | [Session 01](docs/csharp/session-01.md) |
| 02 | Operators, strings, escape characters, conditions, and loops | [Session 02](docs/csharp/session-02.md) |
| 03 | Loop control, arrays, `foreach`, searching, and splitting text | [Session 03](docs/csharp/session-03.md) |

SQL Server and ASP.NET Core are planned learning paths. More lessons will be added as the bootcamp progresses.

The site includes right-to-left Persian text, light and dark themes, search, and copyable code examples. It is built with Zensical and published through GitHub Pages.

## Run the documentation locally

You need Git and Python with `pip` and `venv` available. A .NET SDK is only needed to run the C# lesson examples; building the documentation does not require it.

Clone the repository:

```sh
git clone https://github.com/mohammadhoseindehghani/dotnet-bootcamp-docs.git
cd dotnet-bootcamp-docs
```

Create a virtual environment and install the documentation generator:

**Windows PowerShell**

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install zensical
.\.venv\Scripts\zensical.exe serve
```

**macOS / Linux**

```sh
python3 -m venv .venv
.venv/bin/python -m pip install zensical
.venv/bin/zensical serve
```

Open the local address printed in the terminal. Changes to the documentation are reflected in the preview while the server is running. Press `Ctrl+C` to stop it.

To generate the static site:

```powershell
# Windows PowerShell
.\.venv\Scripts\zensical.exe build --clean
```

```sh
# macOS / Linux
.venv/bin/zensical build --clean
```

The generated files are written to `site/`, which is excluded from Git.

## Run the C# examples

Create a console project with the .NET SDK:

```sh
dotnet new console -n BootcampPractice
cd BootcampPractice
```

Replace the contents of `Program.cs` with one lesson example at a time, then run:

```sh
dotnet run
```

Code blocks marked `csharp` contain program code; blocks marked `text` show expected output. Keep practice projects outside this documentation repository.

## Repository structure

```text
docs/
  index.md                   # Documentation homepage
  csharp/
    index.md                 # C# learning path
    session-01.md            # Lesson notes and exercises
    session-02.md
    session-03.md
  stylesheets/extra.css       # Custom styles and RTL presentation
.github/workflows/
  publish-docs.yml            # GitHub Pages build and deployment
zensical.toml                # Site configuration, theme, and navigation
LICENSE                     # MIT license
```

## Contributing

Corrections, clearer explanations, and improvements to the exercises are welcome. Open an issue to report a problem or submit a pull request with a focused change.

1. Edit the Markdown files under `docs/`. Keep lesson explanations in Persian and follow the surrounding formatting.
2. Keep examples appropriate to the concepts covered in that session and earlier sessions.
3. When adding a lesson, update both the navigation in `zensical.toml` and the session list in `docs/csharp/index.md`.
4. Preview the affected pages, check Persian and code rendering, and run `zensical build --clean` using the environment commands above before submitting.

## Publishing

The [Publish Documentation workflow](.github/workflows/publish-docs.yml) builds and deploys the site when changes are pushed to `main` or `master`. It can also be started manually from the repository's **Actions** tab.

For a new repository or fork, select **GitHub Actions** under **Settings → Pages → Build and deployment → Source**. Update `site_url` in `zensical.toml` to match the published address before deploying your own copy.

## License

This repository's lesson notes and code examples are available under the [MIT License](LICENSE).

Copyright © 2026 Mohammadhosein Dehghani.
