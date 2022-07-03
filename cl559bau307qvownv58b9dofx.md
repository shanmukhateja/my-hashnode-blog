## MyStudio IDE: Cross-Compile Rust+GTK for Windows

Hello

In this post, I'll show you how I compiled [MyStudio IDE](https://github.com/shanmukhateja/mystudio-ide) project (Rust + GTK) for Windows using GitHub Actions and [Docker](https://www.docker.com).

### Tools Used
1. GitHub Actions
2. Docker
3. [cargo-release](https://github.com/crate-ci/cargo-release)

### Why GitHub Actions?

I felt it would be easier to integrate as the project is hosted on GitHub.

### Why Docker?

1. GitHub Actions does not have a way (as of writing) to run CI actions locally. 

> Note: There is a third-party software [act](https://github.com/nektos/act/) which tries to smooth things over but its feature set is far from an actual CI environment.

2. It allows me to run CI builds locally for testing the builds quickly and allows me to improve them in the future.

### Why cargo-release?

cargo-release is a critical piece I need to deploy new releases seamlessly with a single command.

I have setup my project workflow such that project versioning, git tags, project releases and compiling Windows binaries are automated via a single CLI command.    

### GitHub Actions workflow file

You can find latest version [here](https://github.com/shanmukhateja/mystudio-ide/blob/master/.github/workflows/build.yml)

Let's look at the code first, then I'll explain it.

```yml
# .github/workflows/build.yml
on:
  push:
    tags:
      - "mystudio-ide-*"

name: Build

env:
  CI_TOKEN: ${{ secrets.CI_TOKEN }}

jobs:
  build-win:
    name: build-win
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v3

      - name: Get the version
        run: echo ::set-output name=APP_VERSION::${GITHUB_REF/refs\/tags\//}
        id: get_version

      - name: Setup docker image
        run: docker build . -t local

      - name: Build app
        run: docker run -v $PWD:/src/ -e APP_VERSION=${{  steps.get_version.outputs.APP_VERSION  }} local

      - name: Create release
        uses: softprops/action-gh-release@v1
        id: create_release
        with:
          tag_name: ${{  steps.get_version.outputs.APP_VERSION  }}
          token: ${{ env.CI_TOKEN  }}
          draft: false
          prerelease: false
          generate_release_notes: true
          fail_on_unmatched_files: true
          files: |
            release/${{  steps.get_version.outputs.APP_VERSION  }}-win64.zip
```

Let's quickly go-over what's happening here:

```yml
on:
  push:
    tags:
      - "mystudio-ide-*"
```

I am setting a trigger condition for this workflow. In my case, it will be triggered EVERY TIME a new git [tag](https://git-scm.com/book/en/Git-Basics-Tagging) has been pushed to repository with "mystudio-ide-" as prefix.

```yml
env:
  CI_TOKEN: ${{ secrets.CI_TOKEN }}
```

I create an environment variable `CI_TOKEN` with an [Actions secrets](https://docs.github.com/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets) with [access token](https://github.com/settings/tokens/new) as its contents.

```yml

jobs:
  build-win:
    name: build-win
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v3

      - name: Get the version
        run: echo ::set-output name=APP_VERSION::${GITHUB_REF/refs\/tags\//}
        id: get_version
```
Here, we define a `build-win` block with a job named `build-win` that runs on `ubuntu-latest` image and has a few "steps" or instructions to follow for the job.

`Check out code` - We are informing CI to clone the repo to working directory inside the CI node. *We need source code of the project to build, don't we?*  

`Get the version` - I create an environment variable `APP_VERSION` by doing a string replace on `GITHUB_REF` variable (which GitHub Actions provides) that triggered this workflow. 

It creates an "output" from within a job that can be shared across other jobs within a build. [Learn More](https://docs.github.com/en/actions/using-jobs/defining-outputs-for-jobs)

This was necessary as GitHub Actions (as of writing) does not provide a *sane* tag name inside `${{ github.ref }}` variable. [Credit](https://github.community/t/how-to-get-just-the-tag-name/16241)

For example:

**Tag Name:**         `mystudio-v1.1.4`

**${{ github.ref }}**  `refs/tags/mystudio-v1.1.4`

**APP_VERSION**      `mystudio-v1.1.4`

`Setup docker image` - This step invokes Docker to create a Docker image named "local" in the CI. 
It also compiles an external app called `pe-util` available [here](https://github.com/gsauthof/pe-util) which can list out shared objects (DLLs) of a Windows binary.
This will be used in the later steps.

`Build app` - This step runs the Docker image "local" to compile the binary, copy its assets using `pe-util` and MINGW64 and create the zip file for it.

`Create release` - This steps creates a GitHub release for the Git tag and uploads the zip file created previously as the release's asset. *(pretty neat, isn't it?)*


### Cross-compile with GA & Docker

Let's go over how we accomplish cross-compiling software to target Windows binaries from a Linux environment.

```yml

      - name: Setup docker image
        run: docker build . -t local

      - name: Build app
        run: docker run -v $PWD:/src/ -e APP_VERSION=${{  steps.get_version.outputs.APP_VERSION  }} local
```

Before I explain the next steps (which use Docker to build an image "local" inside CI and start the compilation process), here's [the original article](https://nivethan.dev/devlog/cross-compiling-rust-gtk-projects-for-windows.html) describing the procedure which I adapted for my project. 

My additions involve usage of `Papirus theme` for Windows builds and hard dependency to `GtkSourceview4` which is the default text editor entity in the project.

Here's the full file:
```Dockerfile
# Dockerfile
FROM fedora:latest

ENV PROJECT_HOME=/src

ENV MINGW_GTKSOURCEVIEW4_FILENAME=mingw-w64-x86_64-gtksourceview4-4.8.3-1-any.pkg.tar.zst
ENV APP_NAME=mystudio-ide.exe

#
# Set up system
#
WORKDIR /root
RUN dnf -y update
RUN dnf clean all
RUN dnf install -y git cmake file gcc make man sudo tar zstd nano
RUN dnf install -y gcc-c++ boost boost-devel

#
# Add GtkSourceView
#
RUN dnf install -y gtksourceview4 gtksourceview4-devel

#
# Add Papirus icon
#
RUN dnf install -y papirus-icon-theme

#
# Build peldd to find dlls of exes
#
RUN git clone https://github.com/gsauthof/pe-util
WORKDIR pe-util
RUN git submodule update --init
RUN mkdir build

WORKDIR build
RUN cmake .. -DCMAKE_BUILD_TYPE=Release
RUN make

RUN mv /root/pe-util/build/peldd /usr/bin/peldd
RUN chmod +x /usr/bin/peldd

#
# Install Windows libraries
#
RUN dnf install -y mingw64-gcc 
RUN dnf install -y mingw64-freetype 
RUN dnf install -y mingw64-cairo 
RUN dnf install -y mingw64-harfbuzz 
RUN dnf install -y mingw64-pango 
RUN dnf install -y mingw64-poppler 
RUN dnf install -y mingw64-gtk3
RUN dnf install -y mingw64-winpthreads-static 
RUN dnf install -y mingw64-glib2-static 
RUN dnf install -y mingw64-libxml2-static
RUN dnf install -y mingw64-librsvg2-static

#
# Install rust
#
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y
RUN $HOME/.cargo/bin/rustup update

#
# Set up rust for cross compiling
#
RUN $HOME/.cargo/bin/rustup target add x86_64-pc-windows-gnu
ADD cargo-win.config $HOME/.cargo/config
ENV PKG_CONFIG_ALLOW_CROSS=1
ENV PKG_CONFIG_PATH=/usr/x86_64-w64-mingw32/sys-root/mingw/lib/pkgconfig/
ENV GTK_INSTALL_PATH=/usr/x86_64-w64-mingw32/sys-root/mingw/

#
# Setup the mount point
#
VOLUME $PROJECT_HOME
WORKDIR $PROJECT_HOME

#
# Add package.sh
#
ENV PACKAGED_DIR=release
ADD build-win.sh /usr/bin/package.sh
RUN chmod +x /usr/bin/package.sh


#
# Build and package executable
#
CMD ["/usr/bin/package.sh"]

```

In `Dockerfile`, we setup the Docker image with [MINGW-w64](https://www.mingw-w64.org/). 

It enables using GCC compiler for producing Windows binaries from a Linux environment.
Later, we'll configure it beforehand so Rust compiler can produce Windows binaries as intended.

You can find the latest version of `package.sh` a.k.a build-win.sh [here](https://raw.githubusercontent.com/shanmukhateja/mystudio-ide/master/build-win.sh) that compiles the code and creates a zip file for it.

```sh
#!/bin/bash

#
# Extract mingw64-gtksourceview4 to MINGW64
#

cd /tmp

echo "Download and extract $MINGW_GTKSOURCEVIEW4_FILENAME"
curl https://repo.extreme-ix.org/msys2/mingw/mingw64/$MINGW_GTKSOURCEVIEW4_FILENAME -o $MINGW_GTKSOURCEVIEW4_FILENAME
tar --zstd -xf $MINGW_GTKSOURCEVIEW4_FILENAME mingw64 --transform 's/mingw64/mingw/'
cp -rf /tmp/mingw/** /usr/x86_64-w64-mingw32/sys-root/mingw/
rm -rf /tmp/$MINGW_GTKSOURCEVIEW4_FILENAME /tmp/mingw
echo "Finished.."

#
# Build app
#

echo "Building app.."
cd $PROJECT_HOME

$HOME/.cargo/bin/cargo build --target=x86_64-pc-windows-gnu --release

echo "Finished.."

#
# Package app
#

echo "Removing previous assets..."
rm -rf $PACKAGED_DIR
echo "Copying assets..."
mkdir -p $PACKAGED_DIR

# Copy exe
cp target/x86_64-pc-windows-gnu/release/$APP_NAME $PACKAGED_DIR/$APP_NAME

# Copy $APP_NAME required DLLs
/usr/bin/peldd $PACKAGED_DIR/$APP_NAME -t --ignore-errors | xargs cp -t $PACKAGED_DIR/
# Copy librsvg-2-2.dll to release/
cp -r $GTK_INSTALL_PATH/bin/librsvg-2-2.dll $PACKAGED_DIR/

# Asset dir: share
mkdir -p $PACKAGED_DIR/share/{glib-2.0/schemas,gtk-3.0,gtksourceview-4,themes}

# Asset dir: share/glib-2.0
cp -r $GTK_INSTALL_PATH/share/glib-2.0/schemas $PACKAGED_DIR/share/glib-2.0

# Asset dir: share/icons
cp -r $GTK_INSTALL_PATH/share/icons $PACKAGED_DIR/share/icons
cp -r /usr/share/icons/{hicolor,Papirus} $PACKAGED_DIR/share/icons

# Asset dir: lib
mkdir -p $PACKAGED_DIR/lib
cp -r $GTK_INSTALL_PATH/lib/gdk-pixbuf-2.0 $PACKAGED_DIR/lib

# Fix path inside loaders.cache
sed -i 's/\.\.\/lib/lib/g' $PACKAGED_DIR/lib/gdk-pixbuf-2.0/2.10.0/loaders.cache

# Copy gtksourceview assets
cp -r /usr/share/gtksourceview-4/** $PACKAGED_DIR/share/gtksourceview-4/

# Create GTK settings.ini file
cat << EOF > $PACKAGED_DIR/share/gtk-3.0/settings.ini
[Settings]
gtk-theme-name = Adwaita
gtk-icon-theme-name = Papirus
gtk-font-name = Segoe UI 10
gtk-xft-rgba = rgb
gtk-xft-antialias = 1
EOF

mingw-strip $PACKAGED_DIR/*.dll
mingw-strip $PACKAGED_DIR/$APP_NAME
echo "Finished.."

echo "Make zip file"
cd $PACKAGED_DIR
zip -9 -r -q "$APP_VERSION-win64.zip" .
echo "Finished"

```

This shell script performs 3 tasks:

1. Fetch MINGW64 variant of`GtkSourceview4`from an external source and extract its contents to MINGW64 directories so it'll be visible during compilation.
This is needed because Fedora 36 (latest as of writing) doesn't include the package in their repositories along with other MINGW64 ones.

2.  Compile the actual application using `cargo` by specifying the compiler.

3. Once the Windows binary is produced, we copy it to `$PACKAGED_DIR`

4. Gather all DLLs used by this application, create a GTK config file with the application theme, cursor theme, and some Windows specific additions to be used.
It also copies over said theme, icons and other assets needed to run MyStudio IDE application.

5. We later run `mingw-strip` on the application and its DLLs in an effort to discard "symbols" which reduces their file size.
It works similarly to Linux's [strip](https://www.man7.org/linux/man-pages/man1/strip.1.html) command.

6. Lastly, we create a zip file based on the contents of `$PACKAGED_DIR` and exit the container.

### It runs!


![ss_win_mys.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1656847451575/fH9wkHV5p.jpg align="center")

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Mystudio IDE update:<br><br>It runs on Windows x64!!<br><br>Many thanks to several guides on the interwebs for compiling <a href="https://twitter.com/rustlang?ref_src=twsrc%5Etfw">@rustlang</a> and <a href="https://twitter.com/GTKtoolkit?ref_src=twsrc%5Etfw">@GTKtoolkit</a> <br>I used <a href="https://twitter.com/Docker?ref_src=twsrc%5Etfw">@Docker</a> btw!<br><br>Needs fix for incorrect icons &amp; Segoe UI font.<br><br>blog post soon-ish 😁 <a href="https://t.co/qmKWec3M4C">pic.twitter.com/qmKWec3M4C</a></p>&mdash; Surya Teja (@shanmukhateja94) <a href="https://twitter.com/shanmukhateja94/status/1537366293124648960?ref_src=twsrc%5Etfw">June 16, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 


### Knows Issues

- There's a 7-year old bug in GTK where the API doesn't return a valid MIME type for a given filename under Windows.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">TIL : <a href="https://twitter.com/GTKtoolkit?ref_src=twsrc%5Etfw">@GTKtoolkit</a> `g_content_type_guess` does not work under <a href="https://twitter.com/hashtag/Windows?src=hash&amp;ref_src=twsrc%5Etfw">#Windows</a> It returns file extension of input file. <br><br>This is a 7+ years old bug. Now I gotta rethink my approach on GtkTreeView for Windows.<a href="https://twitter.com/rustlang?ref_src=twsrc%5Etfw">@rustlang</a> <a href="https://twitter.com/hashtag/mystudio_ide?src=hash&amp;ref_src=twsrc%5Etfw">#mystudio_ide</a> <a href="https://t.co/30Q9qWiJjq">https://t.co/30Q9qWiJjq</a></p>&mdash; Surya Teja (@shanmukhateja94) <a href="https://twitter.com/shanmukhateja94/status/1537855056388628480?ref_src=twsrc%5Etfw">June 17, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

- The app cannot lazy-load project entities into Workspace Explorer so it will hang forever if it finds a `node_modules` or `.git` directory.

> This happens on both Linux and Windows and I plan on addressing it in the future.

- Cannot preserve UTF16 encoding with latest release.

> A temporary workaround is to reject saving files if its not UTF8. You can find this change [here.](https://github.com/shanmukhateja/mystudio-ide/commit/f07e8a9b939c7497d5394a91cf538ba0aa74d4a1)

- A console window is visible along with the application which I didn't fix since its a quick way to investigate issues. 

- We currently don't have logging in the application.   

### Conclusion

Thanks for reading! Head over to [Releases]() section and test it for yourself. 

Leave a comment on your thoughts. I'd definitely want to know any improvements you may have.

Don't forget to give this post a Like (on the right).

You can also @ me on [Twitter](https://twitter.com/shanmukhateja94)  