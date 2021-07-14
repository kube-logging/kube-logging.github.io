---
title: Logging Operator frequently asked questions
shorttitle: FAQ
weight: 900
---



## How can I run the unreleased master version?

1. Clone the logging-operator repo.

    ```bash
    git clone git@github.com:banzaicloud/logging-operator.git
    ```

1. Navigate to the `logging-operator` folder.

    ```bash
    cd logging-operator
    ```

1. Install with helm

    - Helm v3

        ```bash
         helm upgrade --install --wait --create-namespace --namespace logging logging ./charts/logging-operator --set image.tag=master
        ```

## How can I support the project?

- Give a star to [this repository](https://github.com/banzaicloud/logging-operator) {{% emoji ":star:" %}}
- Add your company to the [adopters](https://github.com/banzaicloud/logging-operator/blob/master/ADOPTERS.md) list

  You can follow the procedure in this video to add your company to the list:
  [![video](http://img.youtube.com/vi/2iaK8adpwfk/0.jpg)](http://www.youtube.com/watch?v=2iaK8adpwfk)
