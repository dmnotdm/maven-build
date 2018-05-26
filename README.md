[![Build Status](https://travis-ci.org/home1-oss/maven-build.svg?branch=master)](https://travis-ci.org/home1-oss/maven-build)

-----
There are link issues on git service generated pages, see gitbook or maven site.
+ [gitbook](https://home1-oss.github.io/home1-oss-gitbook/release/)
+ [maven site](https://home1-oss.github.io/home1-oss/release/)
-----

# maven-build
Parent pom for maven based projects

maven-build support code environment segregation and build deploy segregation.  


## I. Usage

Set maven-build as parent in maven projects.

        <parent>
            <groupId>cn.home1</groupId>
            <artifactId>maven-build</artifactId>
            <version>${maven-build.version}</version>
        </parent>

No dependency will be imported, maven-build only responsible for software engineering,
it does not interfering dependency management.

You need to provide few properties and environment variables, see next chapter.


## II. Properties, Environment variables and their default values

### 1. fetch or deploy
- internal-nexus3.repository
> default: http://nexus3.internal:28081/nexus/repository
Set this property to a real world url.

- local-nexus3.repository
> default: http://nexus3.localal:28081/nexus/repository
Set this property to a real world url.

- build.publish.channel
> default: snapshot
Set this property to 'release' when building release artifact.

### 2. maven-surefire-plugin and maven-failsafe-plugin

- maven.test.skip
> default: false

- maven.integration-test.skip
> default: false

- maven.test.failure.ignore
> default: false

### 3. com.github.eirslett:frontend-maven-plugin
- frontend.nodeDownloadRoot
> default: https://nodejs.org/dist/

- frontend.npmDownloadRoot
> default: http://registry.npmjs.org/npm/-/

Customize these properties only when building front-end projects and default sites are not reachable.

### 4. report
#### 4.1 sonarqube
- internal-sonar.host.url
> default: http://sonarqube.internal:9000
- local-sonar.host.url
> default: http://sonarqube.local:9000

Need these properties only when profile 'sonar' is activated.

#### 4.2 maven-checkstyle-plugin and maven-pmd-plugin

- checkstyle.config.location
> default: https://raw.githubusercontent.com/home1-oss/maven-build/master/src/main/checkstyle/google_checks_8.10.xml
Need to customize when github.com is not reachable.

- pmd.ruleset.location
> default: https://raw.githubusercontent.com/home1-oss/maven-build/master/src/main/pmd/pmd-ruleset-6.0.1.xml
Need to customize when github.com is not reachable.

- project.reporting.outputEncoding
> default: UTF-8

### 5. site
- site.path
> default: ${project.artifactId}-${build.publish.channel}
Customize this properties if you want to publish relative projects's site into same parent directory, 
use same value on relative projects.

- GITHUB_SITE_AUTH_TOKEN
> default: N/A (blank)
- GITHUB_SITE_REPO_OWNER
> default: N/A (blank)
- GITHUB_SITE_REPO_NAME
> default: N/A (blank)

Need these properties only when deploy site to github.com.

### 6. GPG
- GPG_KEYNAME
> default: N/A (blank)
- GPG_PASSPHRASE
> default: N/A (blank)
 
Need these properties only when deploy artifacts to maven central repository.

### 7. maven-compiler-plugin and maven-javadoc-plugin
- project.build.sourceEncoding
> default: UTF-8


## III. Plugins

### 1. build
com.github.eirslett:frontend-maven-plugin

com.amashchenko.maven.plugin:gitflow-maven-plugin
> Gitflow branch model

maven-compiler-plugin with errorprone
> Source encoding.
Java source and target version see maven-build-java*

maven-enforcer-plugin
> Avoid dependency conflict

maven-source-plugin
> Build jar of source code

maven-surefire-plugin and maven-failsafe-plugin with includes and excludes configuration

pl.project13.maven:git-commit-id-plugin

### 2. report
org.asciidoctor:asciidoctor-maven-plugin
org.owasp:dependency-check-maven
org.codehaus.mojo:animal-sniffer-maven-plugin
org.codehaus.mojo:findbugs-maven-plugin
org.codehaus.mojo:jdepend-maven-plugin
org.codehaus.mojo:taglist-maven-plugin
org.codehaus.mojo:versions-maven-plugin
maven-checkstyle-plugin
maven-jxr-plugin
maven-pmd-plugin
maven-surefire-report-plugin

### 3. site
maven-site-plugin
com.github.github:site-maven-plugin

maven-antrun-plugin
> auto clean
'${project.basedir}/src/site/markdown/README.md'
'${project.basedir}/src/site/markdown/src/readme'
'${project.basedir}/src/site/resources'

maven-resources-plugin
> copy
'${project.basedir}/README.md' to '${project.basedir}/src/site/markdown'
'${project.basedir}/src/readme' to '${project.basedir}/src/site/markdown/src/readme'
'${project.basedir}/src/readme' to '${project.basedir}/src/site/resources/src/readme'
'${project.basedir}/src/site/markdown/images' to '${project.basedir}/src/site/resources/images'


## IV. Profiles

### 1. build
infrastructure_github
> Use maven central service.
Deploy maven site to github.

infrastructure_internal
> Use nexus service at organization internal network.
Deploy maven site into organization internal mvnsite.

infrastructure_local
> Use nexus service at user's local (see [docker-nexus3](../../docker-nexus3/README.html)).
Deploy maven site into local mvnsite (see [docker-proxy](../../docker-proxy/README.html)).

git-commit-id
> Generate src/main/resources/git.properties.
activate automatically if '${maven.multiModuleProjectDirectory}/.git/HEAD' exists

publish-deploy-segregation-with-wagon
> activate by set 'publish_deploy_segregation' to 'true'

jacoco-build
> Test coverage report.
activate by property 'jacoco' absent (Enabled by default, to disable, set `-Djacoco=false`)

cobertura
> activate by set property 'jacoco' to 'false'

### 2. report
spring-restdocs
> activate by set property 'spring-restdocs' to 'true'

reports-for-site
> activate by set property 'site' to 'true'

jacoco-report
> activate by property 'jacoco' absent

dependency-check
> Generate a detailed dependency report, This takes a long time, disabled by default.
> activate by set property 'dependency-check' to 'true' (`mvn -Ddependency-check=true`), 
need to be used together with `site` profile of build-site module.

clirr
> activate by set property 'site' to 'true'

sonar
> activate by property 'sonar' present

### 3. site
site
> Generate maven site for project,
run `mvn -Dsite=true site site:stage site:stage-deploy` to active this profile and build site, 
set `-Dsite.path=maven-build-snapshot` to specify upload directory.

infrastructure_github_site_publish
> publish project site to github  
activate on property 'github-publish' present  
needs:  
env.GITHUB_SITE_AUTH_TOKEN  
env.GITHUB_SITE_REPO_OWNER


## V. Repositories

central, spring-libs-release, spring-milestone, spring-libs-snapshot


## VI. Jira and gitlab integration

Add args on `maven site` 

    jira.projectKey         # jira projectKey
    jira.user               # jira username
    jira.password           # jira password


## GPG issue
```
gpg: signing failed: Inappropriate ioctl for device
```

To solve the problem, you need to enable loopback pinentry mode.

Add this to ~/.gnupg/gpg.conf:
```
use-agent 
pinentry-mode loopback
```
And add this to ~/.gnupg/gpg-agent.conf, creating the file if it doesn't already exist:
```
allow-loopback-pinentry
```

## Contribution to maven-build

### local install/deploy maven-build

    mvn -e -U clean install
    
    mvn -e -U -Dinfrastructure=local -Dsite=true -Dsite.path=oss clean install deploy site site:stage site:stage-deploy

### Pull request

Please open PR on develop branch.
