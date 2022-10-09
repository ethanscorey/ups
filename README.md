# ups
A simple package manager for my LFS distro. We use a SQLite DB to hold package information with the following schema:
```
Package:
    package_id: int, primary key
    package_name: str
    package_version: str
    source_url: str
    md5_sum: str
    install_script_url: str
    keep_source_tree: bool
    installed: bool

Dependency:
    package_id: int, foreign key
    dependency_name: str
    dependency_version: str, optional
```
### Installing Packages
When we want to install a package, we perform the following:
  - Check whether there's an entry for the package in our database. If a version number is provided, we only match packages with the correct version number; otherwise, we return the package with the most recent version
    number.
  - If the package is not in the database, warn user, then exit. The user must then add the package to the database.
  - If the package is found, we check whether the package is already installed. If not, we download the source code from the source url into the `/sources` directory.
  - We check each dependency for the package. If it's not installed, we install it. If an incorrect version is installed or the dependency is missing from the package database, we exit with an error code.
  - We extract the downloaded package from the `/sources` directory.
  - We download and run the install script.
  - If `keep_source_tree` is false, we then delete the source tree. Otherwise, we leave it in place. (The archive does not get deleted).

This is accomplished by means of a simple CLI: `ups install <package_name>([=|>|<|!=]<version>)?`

Each field in the database (aside from the package name) can be overrided by setting the appropriate option in the CLI: `ups install --source_url <source_url> <package_name>`

We can list all installed packages with `ups list-packages`.

### Removing Packages

### Upgrading Packages

### Adding Packages to the Database
To add a package to the database, we run: 
```bash
ups add-package \
  --name <package_name> \
  --version <package_version> \
  --source_url <source_url> \
  --md5sum <md5sum> \
  --install_script <install_script_url> \
```
Adding `--keep_source` or `installed` will set the appropriate Boolean values to true (default is false).

### Package Dependencies
To add a package dependency to the database we run:
```bash
ups add-dependency --package_name <package_name> <dependency>([=|>|<|!=]>version>)?
```
To list all dependencies for a package, we can run:
```bash
ups list-dependencies <package_name>
```
