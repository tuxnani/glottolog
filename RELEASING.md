
# Releasing glottolog/glottolog


## Preparations

- Make sure you have the latest ISO 639-3 code tables from 
  https://iso639-3.sil.org/code_tables/download_tables
  put into `build/`,
- Check out `master`, pull the latest changes:
  ```shell
  git checkout master
  git pull origin master
   ```
- Check the tree and references running
  ```
  glottolog-admin check
  ```
  making sure there are no `ÈRROR`s


## Creating the release PR

- Check out a new branch:
  ```shell
  git checkout -b release-<version>
  ```
- Update 
  - `CHANGES.md`
  - `CONTRIBUTORS.md`
  - `config/publication.ini` adding a proper version number
- Run
  ```shell
  glottolog-admin update
  ```
  and make sure the data can be loaded into treedb:
  ```shell
  cd ../treedb
  python -c "import treedb; treedb.load(); treedb.write_csv()"
  cd ../glottolog
  ```
- Update automatically created BibTeX files:
  - `iso6393.bib`: Run `glottolog-admin isobib`
  - `elpub.bib`: Run `glottolog-admin elpubbib`
  - `evobib.bib`:
    - download the latest version from https://doi.org/10.5281/zenodo.4071598
    - Run `glottolog-admin evobib evobib-converted.bib`
  - `dplace.bib`: Run `glottolog-admin dplacebib`
  - `benjamins.bib`:
    - Switch to the clone of `glottolog/benjamins`
    - Pull the latest changes via FTP 
    - Recreate `benjamins.bib`, running `python to_bib.py`
    - Switch back to `glottolog/glottolog`
    - Run `glottolog-admin benjaminsbib PATH/TO/benjamins/benjamins.bib`
- Run `glottolog-admin bib` to create `build/monster-utf8.bib` - about 20mins
- Run
  ```shell
  glottolog-admin release
  git commit -a -m"release <version>"
  git push origin release-<version>
  ```
- Create a corresponding PR on GitHub


## Releasing

Upon approval of the PR:

- Merge the release PR into master
  ```shell
  gh pr checkout <#>
  gh pr review
  gh pr merge
  git pull origin
  git tag -a v<version> -m "release <version>"
  ```
- Push the tag to origin:
  ```shell
  git push --tags origin
  ```
- Create corresponding release of glottolog-cldf: 
  ```shell
  cd ../glottolog-cldf
  cldfbench makecldf --with-cldfreadme --with-zenodo --glottolog-version v<version> cldfbench_glottolog.py
  cldfbench readme cldfbench_glottolog.py
  pytest
  git commit -a -m"release <release>"
  git tag -a v<version> -m "release <version>"
  git push origin
  git push --tags origin
  ```
- Re-set the version number to dev mode, by incrementing and adding `.dev0`:
  - `config/publication.ini`
- Create "proper" releases on GitHub and have it picked up by ZENODO.
- Add DOI badges from ZENODO as soon as they become available.
- Close fixed issues.


# Checking a pull request

1. Update the `master` branch and store languoid stats:
   ```
   git checkout master
   git pull origin
   glottolog-admin writelanguoidstats
   ```
2. Check out the PR branch and run the checks:
   ```
   git checkout <BRANCH>
   glottolog-admin check --old-languoids
   ```
3. If necessary, run `glottolog-admin updatesources` and continue with 2.


## Troubleshooting

If `glottolog-admin bib` fails at
```python
  File "/home/forkel/venvs/glottolog/pyglottolog/src/pyglottolog/references/bibfiles_db.py", line 559, in assign_ids
    assert Entry.allhash(conn)
AssertionError
```
check the bad entries in `build/bibfiles.sqlite3`:
```sql
sqlite> select * from entry where hash is null;
292775|21|hw:Nigam:Andhra-Pradesh||||
```

This can be due to malformed BibTeX entries - and correspondingly must
be fixed in the BibTeX before re-running `glottolog-admin bib --rebuild`.

