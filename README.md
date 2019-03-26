# Maven-Build-Parents

Using the `eclipse-parent-updatesite` base POM, you can define Maven Tycho based builds with little effort. The parent POM covers
* compilation of java and xtend code
* building of plugins and features
* building of source plugins and source features
* test execution and coverage reports
* API documentation using javadoc
* updating, filtering, and merging of target platform definitions

## Requirements
* your project has to have a root POM that is an aggregating POM as well
* you have to activate the following extensions in the `extensions.xml` in the `.mvn` folder
  * `org.eclipse.tycho.extras:tycho-pomless:1.3.0`
  * `org.palladiosimulator:tycho-tp-refresh-maven-plugin:0.2.3`
* you have to refer to the parent POM in your root POM

## Target Platform
You can define your target platform explicitly or by just giving repository URLs to resolve bundles. If you want to use the latter, you have to use the `eclipse-parent-product`. You should, however, only use this method for product builds.

To define your target platform explicitly, you have to create target platform definitions. The parent POM will take care of merging multiple target definitions. You can refer to target files by defining a property `org.palladiosimulator.maven.tychotprefresh.tplocation.n` with the path to the file. You can use project coordinates (`groupdId:artifactId:version:classifier:fileExtension`) as well if your target definition resides inside a maven artifact. Please note that you have to use different numbers for `n` starting with 1.

You can use additional attributes in the target definition file to filter entries or update version numbers. Please note that you cannot use the Eclipse editor anymore to edit files containing additional attributes.

To select only a subset of your target definition during a build, you have to add the attribute `filter` to a `location` element and set the value to any name you like. Afterwards, you have to add a property `org.palladiosimulator.maven.tychotprefresh.filter.n` with the name you chose to your root POM. Use different numbers `n` for different filters starting with 1. The parent POM already defines the filters `nightly` and `release` that are activated by activating the `nightly` or `release` profile. The former is activated if the latter is not activated.

To update versions of units mentioned in the target platform, you have to add the attribute `refresh` and set it to `true`. Before the build takes place, the latest versions available in the given location will be used.

You can always have a look at the generated target platform definition in the `target` folder of your root project.

## Update site / JavaDoc
To define an update site, you have to create a POM with packaging type `eclipse-repository` and place a `category.xml` besides it. You can edit this file using the default Eclipse editors. The resulting update site will be placed in `target/repository`. The JavaDoc files will be placed in a `javadoc` folder inside the repository.

## Automated MWE2 Execution
As part of our continuous effort to improve the separation between generated code and custom implementations we provide support to automatically execute MWE2-based workflows during project builds. Per project, you can specify a workflow to be executed during Maven's generate-sources phase and thereby before code compilation. The workflow needs to be saved as `workflow/generate.mwe2` in order to trigger the automated execution. Similarly, by providing a `workflow/clean.mwe2` workflow you can extend Maven's clean phase with custom logic to remove generated source code.

The workflows are expected to accept a parameter `workspaceRoot` which will be provided with the outermost project root. The parameter should be passed as `platformUri` to `StandaloneSetup` bean of your workflow. Make sure to set `scanClassPath=true` to enable the workflow to create proper `platform:/resource/{project name}` URI mappings for your eclipse projects.
