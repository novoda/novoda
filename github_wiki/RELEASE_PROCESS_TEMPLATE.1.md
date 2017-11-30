Follow these steps to release a new version of this library:

### Increase the version
Make sure the version number specified in the `merlin/build.gradle` file is correct.

[[images/gradle_version.png]]

This form, `0.9.1` is known as **Semantic Versioning** and is structured as `major.minor.patch`. To find out more about Semantic Versioning go to http://semver.org/

### Novoda CI
Now that a release tag has been created on GitHub we need to head to the Novoda CI. The release job needs
to be run manually with the `DRY_RUN` parameter unchecked.

[[images/release_build_parameters.png]]

### Head to Bintray
After the build has completed on the Novoda CI we head to [bintray](https://bintray.com/novoda/) to make sure that the
new version is present.

### Publish a Release

[[images/release_tag.png]]

**Tag version:** Follows the version added in the `merlin/build.gradle` file prefixed with a `v` 

`v0.9.1`

**Release Title:** Will match the Tag version and can include a summary 

`v0.9.1:Level 24 support`

**Description:** Explain what this release aims to address, include links to any prominent PRs as these will provide more detail.
