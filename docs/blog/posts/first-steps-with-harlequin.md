---
date:
  created: 2025-06-16
tags:
  - Harlequin
  - Duckdb
  - Tooling
categories:
  - Tools
links:
- External links:
  - Docs: https://harlequin.sh/docs/getting-started/usage
---

# First steps with Harlequin

Today I used Harlequin for the first time. It is a SQL IDE for the terminal and the setup or installation was surprisingly easy.

<!-- more -->

I tried out the tool as part of my "elt-on-github-actions" repository (more information on that will follow). I wanted to test the conditional logic in my marts models: If it is the production environment, output to blob storage and if it is not, then create a view in the duckdb database.

Since I was already using `uv` to manage the dependencies of the dbt project, getting Harlequin to run was as simple as typing `uvx harlequin database/dwh_local.duckdb`. This command uses the shorthand notation for `uv tool` which runs the tool in another ephemeral environment. I then call Harlequin and point it to my local duckdb file.

That worked like a charm.

## Handy shortcuts

There were also some useful keyboard shortcuts that I quickly picked up from the documentation's "Get oriented" guide.

- `ctrl + j` to execute a query
- `ctrl + w` to clear the query editor
- `ctrl + n` to open a new buffer (or tab) in the query editor
- `F10` for fullscreen mode, this is especially useful when browsing the results viewer
- `F2` to focus (again) on the query editor
- `F6` to focus on the data catalogue
- `.` on any database object to select options such as "view DDL" or "describe"
- `ctrl + b` to hide or show the data catalogue pane
- `crtl + q` to quit the program

## Conclusion

Overall, I must say that I really like Harlequin. I was positively surprised by how quickly I was able to get it running in my project.

I am certain I will use it more often in the future considering its tight integration with `uv` and the ease-of-use once I picked up a few keyboard shortcuts.
