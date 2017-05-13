# Releasing

## Source code

Edit varable `$VERSION` in the following files:

  * `script/pgsqlms`
  * `lib/OCF_Directories.pm.PL`
  * `lib/OCF_Functions.pm`
  * `lib/OCF_ReturnCodes.pm`

In `Build.PL`, search and edit the following line:

```
dist_version       => '1.0.0'
```

In `resource-agents-paf.spec`:
  * update the tag in the `_tag` variable (first line)
  * update the version in `Version:`
  * edit the changelog
    * date format: `LC_TIME=C date +"%a %b %d %Y"`

In `debian/`, edit the `changelog` file

## Tagging and building tar file

```
TAG=v1.0.0
git tag $TAG
git push --tags
git archive --prefix=PAF-$TAG/ -o /tmp/PAF-$TAG.tgz $TAG
```

## Release on github

  - go to https://github.com/dalibo/PAF/tags
  - edit the release notes for the new tag
  - set "PAF $VERSION" as title, eg. "PAF 1.0.0"
  - here is the format of the release node itself:
    YYYY-MM-DD -  Version X.Y.Z
    
    Changelog:
      * item 1
      * item 2
      * ...
      
      See http://dalibo.github.io/PAF/documentation.html
  - upload the tar file
  - save

## Building the RPM file

### Installation

```
yum group install "Development Tools"
yum install rpmdevtools
useradd makerpm
```

### Building the package

```
su - makerpm
rpmdev-setuptree
git clone https://github.com/dalibo/PAF.git
spectool -R -g PAF/resource-agents-paf.spec
rpmbuild -ba PAF/resource-agents-paf.spec
```

Don't forget to upload the package on github release page.

## Building the deb file

### Installation

```
apt-get install dh-make devscripts libmodule-build-perl resource-agents
```


### Building the package

Package to install on your debian host to build the builder environment

```
VER=1.0.0
wget "https://github.com/dalibo/PAF/releases/download/v${VER/\~/_}/PAF-v${VER/\~/_}.tgz" -O resource-agents-paf_${VER}.orig.tar.gz
mkdir resource-agents-paf-$VER
tar zxf resource-agents-paf_${VER}.orig.tar.gz -C "resource-agents-paf-$VER" --strip-components=1
cd resource-agents-paf-${VER}
debuild -i -us -uc -b
```

Don't forget to upload the package on github release page.

## Documentation

Update the "quick start" documentation pages with the links to the new packges  

## Debian repository

The Debian repository is handled with `aptly`:

~~~
apt-get install aptly
~~~

Here is how the jessie repository has been created:

~~~
git checkout gh-pages 
cat <<'EOF' > aptly.conf 
{
  "rootDir": "./apt",
  "downloadConcurrency": 4,
  "downloadSpeedLimit": 0,
  "architectures": ["i386","amd64","all"],
  "dependencyFollowSuggests": false,
  "dependencyFollowRecommends": false,
  "dependencyFollowAllVariants": false,
  "dependencyFollowSource": false,
  "gpgDisableSign": false,
  "gpgDisableVerify": false,
  "downloadSourcePackages": false,
  "ppaDistributorID": "",
  "ppaCodename": "",
  "skipContentsPublishing": false,
  "S3PublishEndpoints": {},
  "SwiftPublishEndpoints": {}
}
EOF

# create the repo
aptly repo create -distribution=jessie -comment="PostgreSQL Automatic Failover - Debian 8 repository" -config=aptly.conf jessie-paf

# import the package
aptly repo add -config=aptly.conf jessie-paf /tmp/resource-agents-paf_2.1.0-1_all.deb

# create a snapshot of the repo
aptly snapshot create -config=aptly.conf paf-2.1.0 from repo jessie-paf

# publish it to apt/public
aptly publish snapshot -config=aptly.conf paf-2.1.0

# push it to github
git add apt/public
git commit
git push
~~~

To use the repository:

~~~
wget --quiet -O - 'http://pgp.mit.edu/pks/lookup?op=get&search=0xC5619F680828C222' | apt-key add -
deb http://blog.ioguix.net/PAF/apt/public/ jessie main
apt-get update
apt-get install resource-agents-paf
~~~

Here is how the wheezy repository has been created:

~~~
git checkout gh-pages
aptly repo create -distribution=wheezy -comment="PostgreSQL Automatic Failover - Debian 7 repository" -config=aptly.conf wheezy-paf
aptly repo add -config=aptly.conf wheezy-paf /tmp/resource-agents-paf_1.1.0-1_all.deb
aptly snapshot create -config=aptly.conf paf-1.1.0 from repo wheezy-paf
aptly publish snapshot -config=aptly.conf paf-1.1.0
git add apt/public/
git ci
git push
~~~

To use the repository:

~~~
wget --quiet -O - 'http://pgp.mit.edu/pks/lookup?op=get&search=0xC5619F680828C222' | apt-key add -
deb http://blog.ioguix.net/PAF/apt/public/ wheezy main
apt-get update
apt-get install resource-agents-paf
~~~

TODO:
* create a gpg key for the PAF project
* create a keyring package
* how to create/import a new version of the package?
* how to deal with multiple versions of the package?
* create repo for other distro
