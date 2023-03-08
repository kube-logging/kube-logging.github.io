# kube-logging.github.io - Logging operator documentation

This repository is the source of the Logging operator documentation, published at [https://kube-logging.github.io](https://kube-logging.github.io/docs/).

The documentation is built using [Hugo](https://gohugo.io/) and the [Docsy theme](https://www.docsy.dev/docs/)

## Using this repository

You can run the website locally using Hugo (Extended version).

### Prerequisites

To use this repository, you need the following installed locally:

- [npm](https://www.npmjs.com/)
- [Go](https://go.dev/)
- [Hugo (Extended version)](https://gohugo.io/)

Before you start, install the dependencies. Clone the repository and navigate to the directory:

```bash
git clone https://github.com/kubernetes/website.git
cd website
```

The Kubernetes website uses the [Docsy Hugo theme](https://github.com/google/docsy#readme). Even if you plan to run the website in a container, we strongly recommend pulling in the submodule and other development dependencies by running the following:

### Windows
```powershell
# fetch submodule dependencies
git submodule update --init --recursive --depth 1
```
