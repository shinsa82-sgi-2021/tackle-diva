# (Working draft) Development Policy, Design Document and Implementation

## Past Releases (Tags)

- `v1.0.0`: the first release. (Nov., 2021)
- `v2.0.0`: second release (and the first public release). (Jan., 2022)

## Branches

- `upstream`: copy of main branch of the upstream repo. May be useful to create PRs. 
- `main`: the main branch (and dev branch for v2). See this branch for the latest revision.
  - You cannot directly push to `main` branch. Need to create a PR.
- `dev`, `shinsa82`, etc.: development branches.
- `v1`: ver. 1 release branch.

## Overall: Development env.

Use dev-container of VS Code. 

- Start up VS Code in the repository root and select ReOpen in Container.
- In the dev container, repository directory `diva-doa` is mounted on `/workspace/diva-doa` and you are there.
- You can run `doa/migrate.sh` directly like:
    ```bash
    bash doa/migrate.sh -o /tmp/out https://github.com/saud-aslam/trading-app
    ls /tmp/out  # list generated files
    ```

## Overall: Python integration

- Use `poetry` (https://python-poetry.org/) for dependency management and installation.
    - It's fast.
    - It can resolve dependencies that pip cannot.
- Use `typer` (https://typer.tiangolo.com/) for CLI apps.
    - Type based: easy to read, code and test.
    - Automatically create shell-completion files.

----
# Architecture

Legend:
- Dotted line: unimplemented.
- Dashed line: reference.

The figure below shows an architecture overview of DiVA-DOA.
See [Main Logic](#main-logic) section below for (1) and (2) in the diagram.

![Architecture overview](arch.dot.svg)

## Overall

We write [${ROOT}](..) for the directory `tackle-diva/doa` of `tackle-diva` repository.
Files related to tool main logic is just as follows:

```
${ROOT}
├── doa/
└── run-doa.sh
```

- [${ROOT}/doa](../doa) contains main logic and mounted on `/work` in the Docker image.
  - [${ROOT}/doa/migrate.sh](../doa/migrate.sh) is the main script.
- [${ROOT}/run-doa.sh](../run-doa.sh) is a wrapper script, This starts a container of `diva-doa` image and run the main script. 

## Main Logic

Hereafter, app directory in the container `/work` is denoted by `${WORK}` and `${APP_ROOT}` as a root of the target app.
A main script [${WORK}/migrate.sh](../doa/migrate.sh) will be executed.

## (*) DBMS config. analysis

None

## (1) SQL file analysis

Implemented in a Python program `${WORK}/analyzers/analyze_sqls.py`.

- Glob `${APP_ROOT}/**/*.sql` files under the root of the target app (using Python `iglob`).
- Genrate manifest(s) of ConfigMap resource(s) that contains **basename** of each SQL file as a key and file content as a value. This is done by `kubectl create cm <name> --from-file ... -o yaml --dry-run=client`.
 
For exaample,
```
${APP_ROOT}
└── ddl/
    ├── foo.sql
    └── bar.sql
```

Then a manifest file for ConfigMap that contains `foo.sql` and `bar.sql` as its keys.

### Possible Issues:

- Since only basenames are stored as key, it does not work in the situation like
  - SQL files `${APP_ROOT}/foo.sql` and `${APP_ROOT}/ddl/foo.sql` exist.

## (2) DB init script analysis

DiVA needs to know how to execute SQL scripts that are found the step (2).

- Current implementation extracts lines that contain `psql` command. from the file that the user specified by `-i` option (e.g., `migrate.sh -i start-up.sh ...`).
- It parses `psql` arguments (using Lark library) and rewrites host name and user name to work in container environment.
  - Typical syntax: `psql -h <host> -U <user> -d <database> -f <sql-file>`

### Possible Issues:

- Current implementation contains hardcode portion and is not flexible.
- Often such command lines are included in README files...

## (*) Program code analysis and transformation

- To appear.