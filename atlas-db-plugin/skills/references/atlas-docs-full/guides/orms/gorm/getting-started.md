Automatic Schema Migration Planning for GORM | Atlas Guides

[Skip to main content](#__docusaurus_skipToContent_fallback)

v1.0 Atlas v1.0 is here! [Read the announcement →](/blog/2025/12/23/atlas-v1)

Copy page

[**GORM**](https://gorm.io) allows users to manage their database schemas using its [AutoMigrate](https://gorm.io/docs/migration.html#Auto-Migration) feature, which is usually sufficient during development and in many simple cases. However, many teams eventually need more control and decide to employ the [versioned migrations](/concepts/declarative-vs-versioned#versioned-migrations) methodology.

Once this happens, the responsibility of planning migration scripts and making sure they are in line with what GORM expects at runtime is moved to developers.

[**Atlas**](https://atlasgo.io) can help in these cases by automatically planning database schema migrations for developers using GORM by calculating the diff between the _current_ state of the database, and its _desired_ state defined by GORM models.

To use Atlas with GORM, users can utilize the [GORM Atlas Provider](https://github.com/ariga/atlas-provider-gorm), a small Go program that can load the schema of a GORM project into Atlas.

## Prerequisites[​](#prerequisites "Direct link to Prerequisites")


1.  A local [GORM](https://gorm.io) project

If you don't have a GORM project handy, you can use [go-admin-team/go-admin](https://github.com/go-admin-team/go-admin) as a starting point:
```codeBlockLines_AdAo
git clone git@github.com:go-admin-team/go-admin.git
```
2.  Atlas installed on your machine:

*   macOS + Linux
*   Homebrew
*   Docker
*   Windows
*   CI
*   Manual Installation

To download and install the latest release of the Atlas CLI, simply run the following in your terminal:
```codeBlockLines_AdAo
curl -sSf https://atlasgo.sh | sh
```
Get the latest release with [Homebrew](https://brew.sh/):
```codeBlockLines_AdAo
brew install ariga/tap/atlas
```
To pull the Atlas image and run it as a Docker container:
```codeBlockLines_AdAo
docker pull arigaio/atlasdocker run --rm arigaio/atlas --help
```
If the container needs access to the host network or a local directory, use the `--net=host` flag and mount the desired directory:
```codeBlockLines_AdAo
docker run --rm --net=host \  -v $(pwd)/migrations:/migrations \  arigaio/atlas migrate apply  --url "mysql://root:pass@:3306/test"
```
Download the [latest release](https://release.ariga.io/atlas/atlas-windows-amd64-latest.exe) and move the atlas binary to a file location on your system PATH.

**GitHub Actions**

Use the [setup-atlas](https://github.com/marketplace/actions/setup-atlas) action to install Atlas in your GitHub Actions workflow:
```codeBlockLines_AdAo
- uses: ariga/setup-atlas@v0  with:    cloud-token: ${{ secrets.ATLAS_CLOUD_TOKEN }}
```
**Other CI Platforms**

For other CI/CD platforms, use the installation script. See the [CI/CD integrations](/integrations#cicd-platforms) for more details.

If you want to manually install the Atlas CLI, pick one of the below builds suitable for your system.

### MacOS


![Download icon](/icons-docs/download.svg)

[AMD 64](https://release.ariga.io/atlas/atlas-darwin-amd64-latest) ([md5](https://release.ariga.io/atlas/atlas-darwin-amd64-latest.md5)/[sha256](https://release.ariga.io/atlas/atlas-darwin-amd64-latest.sha256))

![Download icon](/icons-docs/download.svg)

[ARM 64](https://release.ariga.io/atlas/atlas-darwin-arm64-latest) ([md5](https://release.ariga.io/atlas/atlas-darwin-arm64-latest.md5)/[sha256](https://release.ariga.io/atlas/atlas-darwin-arm64-latest.sha256))

### Linux


![Download icon](/icons-docs/download.svg)

[AMD 64](https://release.ariga.io/atlas/atlas-linux-amd64-latest) ([md5](https://release.ariga.io/atlas/atlas-linux-amd64-latest.md5)/[sha256](https://release.ariga.io/atlas/atlas-linux-amd64-latest.sha256))

![Download icon](/icons-docs/download.svg)

[ARM 64](https://release.ariga.io/atlas/atlas-linux-arm64-latest) ([md5](https://release.ariga.io/atlas/atlas-linux-arm64-latest.md5)/[sha256](https://release.ariga.io/atlas/atlas-linux-arm64-latest.sha256))

### Windows


![Download icon](/icons-docs/download.svg)

[AMD 64](https://release.ariga.io/atlas/atlas-windows-amd64-latest.exe) ([md5](https://release.ariga.io/atlas/atlas-windows-amd64-latest.exe.md5)/[sha256](https://release.ariga.io/atlas/atlas-windows-amd64-latest.exe.sha256))

3.  The [GORM Atlas Provider](https://github.com/ariga/atlas-provider-gorm)

Install the provider by running:
```codeBlockLines_AdAo
go get -u ariga.io/atlas-provider-gorm
```

## Standalone vs Go Program mode[​](#standalone-vs-go-program-mode "Direct link to Standalone vs Go Program mode")


The Atlas GORM Provider can be used in two modes:

*   **Standalone** - If all of your GORM models exist in a single package and either embed `gorm.Model` or contain `gorm` struct tags, you can use the provider directly to load your GORM schema into Atlas.
*   **Go Program** - If your GORM models are spread across multiple packages, or do not embed `gorm.Model` or contain `gorm` struct tags, you can use the provider as a library in your Go program to load your GORM schema into Atlas.

[

##### Standalone Mode


Continue setting up Atlas with GORM in Standalone Mode.

](/guides/orms/gorm/standalone)[

##### Go Program Mode


Continue setting up Atlas with GORM in Go Program Mode.

](/guides/orms/gorm/program)

*   [Prerequisites](#prerequisites)
*   [Standalone vs Go Program mode](#standalone-vs-go-program-mode)