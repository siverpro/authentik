# Releasing authentik

### Creating a standard release

-   Ensure a branch exists for the version family (for 2022.12.2 the branch would be `version-2022.12`)
-   Merge all the commits that should be released on the version branch

    If backporting commits to a non-current version branch, cherry-pick the commits.

-   Check if any of the changes merged to the branch make changes to the API schema, and if so update the package `@goauthentik/api` in `/web`
-   Push the branch, which will run the CI pipeline to make sure all tests pass
-   Create/update the release notes

    #### For initial releases:

    -   Copy `website/docs/releases/_template.md` to `website/docs/releases/v2022.12.md` and replace `xxxx.x` with the version that is being released

    -   Fill in the section of `Breaking changes` and `New features`, or remove the headers if there's nothing applicable

    -   Run `git log --pretty=format:'- %s' version/2022.11.3...version-2022.12`, where `version/2022.11.3` is the tag of the previous stable release. This will output a list of all commits since the previous release.

    -   Paste the list of commits since the previous release under the `Minor changes/fixes` section.

    -   Sort the list of commits alphabetically and remove all commits that have little importance, like dependency updates and linting fixes

    -   Run `make gen-diff` and copy the contents of `diff.md` under `API Changes`

    -   Update `website/sidebars.js` to include the new release notes, and move the oldest release into the `Previous versions` category.

    -   Run `make website`

    #### For subsequent releases:

    -   Paste the list of commits since the previous release into `website/docs/releases/v2022.12.md`, creating a new section called `## Fixed in 2022.12.2` underneath the `Minor changes/fixes` section

    -   Sort the list of commits alphabetically and remove all commits that have little importance, like dependency updates and linting fixes

    -   Run `make gen-diff` and copy the contents of `diff.md` under `API Changes`, replacing the previous changes

    -   Run `make website`

-   Run `bumpversion` on the version branch with the new version (i.e. `bumpversion --new-version 2022.12.2 minor --verbose`)
-   Push the tag and commit
-   A GitHub actions workflow will start to run a last test in container images and create a draft release on GitHub
-   Edit the draft GitHub release

    -   Make sure the title is formatted `Release 2022.12.0`
    -   Add the following to the release notes

        ```
        See https://goauthentik.io/docs/releases/2022.12
        ```

        Or if creating a subsequent release

        ```
        See https://goauthentik.io/docs/releases/2022.12#fixed-in-2022121
        ```

    -   Auto-generate the full release notes using the GitHub _Generate Release Notes_ feature

### Preparing a security release

-   Create a draft GitHub Security advisory

<details><summary>Template</summary>
<p>

```markdown
### Summary

Short summary of the issue

### Patches

authentik x, y and z fix this issue, for other versions the workaround can be used.

### Impact

Describe the impact that this issue has

### Details

Further explain how the issue works

### Workarounds

Describe a workaround if possible

### For more information

If you have any questions or comments about this advisory:

-   Email us at [security@goauthentik.io](mailto:security@goauthentik.io)
```

</p>
</details>

-   Request a CVE via the draft advisory
-   If possible, add the original reporter in the advisory
-   Implement a fix on a local branch `security/CVE-...`

    The fix must include unit tests to ensure the issue can't happen again in the future

    Update the release notes as specified above, making sure to address the CVE being fixed

    Create a new file `/website/docs/security/CVE-....md` with the same structure as the GitHub advisory

    Include the new file in the `/website/sidebars.js`

-   Check with the original reporter that the fix works as intended
-   Announce the release of the vulnerability via Mailing list and discord

<details><summary>Mailing list template</summary>
<p>

```markdown
We'll be publishing a security Issue and accompanying Fix on _date_, 13:00 UTC with the Criticality level High. Fixed versions x, y and z will be released alongside a workaround for previous versions. For more infos, see the authentik Security policy here: https://goauthentik.io/docs/security/policy.
```

</p>
</details>

<details><summary>Discord template</summary>
<p>

```markdown
@everyone We'll be publishing a security Issue and accompanying Fix on _date_, 13:00 UTC with the Criticality level High. Fixed versions x, y and z will be released alongside a workaround for previous versions. For more infos, see the authentik Security policy here: https://goauthentik.io/docs/security/policy.
```

</p>
</details>

### Creating a security release

-   On the date specified in the announcement, push the local `security/CVE-...` branch into a PR, and squash merge it if the pipeline passes
-   If the fix made any changes to the API schema, merge the PR to update the web API client
-   Cherry-pick the merge commit onto the version branch
-   If the fix made any changes to the API schema, manually install the latest version of the API client in `/web`
-   Resume the instructions above, starting with the `bumpversion` step