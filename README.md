# Personal Blog

This repository contains the source code for my personal blog, built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

## Prerequisites & Installation (Ubuntu)

These instructions will guide you through setting up the required tools on an Ubuntu-based system.

1.  **Install Git**
    ```bash
    sudo apt update
    sudo apt install git
    ```

2.  **Install Hugo**
    The site is built using a specific version of Hugo. To ensure compatibility, install version `0.154.4`:
    ```bash
    wget https://github.com/gohugoio/hugo/releases/download/v0.154.4/hugo_extended_0.154.4_linux-amd64.deb
    sudo dpkg -i hugo_extended_0.154.4_linux-amd64.deb
    ```

3.  **Install Dart-Sass**
    The theme requires `dart-sass` to compile stylesheets.
    ```bash
    sudo snap install dart-sass
    ```

## Local Development

1.  **Clone the Repository**
    Clone this repository to your local machine.
    ```bash
    git clone https://github.com/oxomichael/oxomichael.github.io.git
    cd oxomichael.github.io
    ```

2.  **Initialize Theme Submodule**
    The PaperMod theme is included as a Git submodule. Initialize it with the following command:
    ```bash
    git submodule update --init --recursive
    ```

3.  **Run the Hugo Server**
    Start the local development server to preview your site. The `-D` flag builds and shows draft posts.
    ```bash
    hugo server -D
    ```
    You can now view your site by opening **http://localhost:1313** in your web browser. Changes to content and configuration files will be reflected live.

## Creating New Content

To create a new blog post, use the `hugo new` command. For a multilingual site like this one, it's best to create the file in the correct language directory.

For example, to create a new post in French:
```bash
hugo new content/fr/posts/mon-nouvel-article.md
```

Or in English:
```bash
hugo new content/en/posts/my-new-post.md
```

## Deployment & GitHub Configuration

Deployment is handled automatically by a GitHub Actions workflow whenever changes are pushed to the `master` branch.

The workflow performs the following steps:
1.  Installs the correct versions of Hugo and other dependencies.
2.  Checks out the code, including the theme submodule.
3.  Builds the Hugo site into the `./public` directory.
4.  Deploys the built site to GitHub Pages.

### Repository Configuration

For the automatic deployment to work, you must configure your GitHub repository settings once:

1.  Navigate to your repository on GitHub.
2.  Go to **Settings** > **Pages**.
3.  Under the "Build and deployment" section, set the **Source** to **GitHub Actions**.

The workflow will now handle the build and deployment for you.
