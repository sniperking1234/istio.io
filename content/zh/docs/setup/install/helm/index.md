---
title: 使用 Helm 安装
linktitle: 使用 Helm 安装
description: 使用 Helm 在 K8s 集群中安装和配置 Istio。
weight: 30
keywords: [kubernetes,helm]
owner: istio/wg-environments-maintainers
test: yes
---

请遵循本指南使用 [Helm](https://helm.sh/docs/) 安装和配置 Istio 网格。

{{< boilerplate helm-preamble >}}

{{< boilerplate helm-prereqs >}}

## 安装步骤 {#installation-steps}

本节介绍使用 Helm 安装 Istio 的过程。Helm 安装的一般语法是：

{{< text syntax=bash snip_id=none >}}
$ helm install <release> <chart> --namespace <namespace> --create-namespace [--set <other_parameters>]
{{< /text >}}

该命令指定的变量如下：

* `<chart>`：一个打好包的 Chart 路径，也可以是一个未打包的 Chart 目录或 URL。
* `<release>`：一个用于标识和管理安装后的 Helm Chart 的名称。
* `<namespace>`：要安装 Chart 的命名空间。

您可以使用一个或多个 `--set <parameter>=<value>` 参数更改默认配置值。
或者可以使用 `--values <file>` 参数在一个自定义值文件中指定几个参数。

{{< tip >}}
您可以使用 `helm show values <chart>` 命令显示配置参数的默认值，或参考 `artifacthub` Chart
文档中的[自定义资源参数](https://artifacthub.io/packages/helm/istio-official/base?modal=values)、
[Istiod Chart 配置参数](https://artifacthub.io/packages/helm/istio-official/istiod?modal=values)
和 [Gateway Chart 配置参数](https://artifacthub.io/packages/helm/istio-official/gateway?modal=values)。
{{< /tip >}}

1. 安装 Istio Base Chart，它包含了集群范围的自定义资源定义 (CRD)，这些资源必须在部署 Istio 控制平面之前安装：

    {{< warning >}}
    执行修订版安装时，Base Chart 需要设置 `--set defaultRevision=<revision>` 值以使资源验证起作用。
    以下我们将安装 `default` 修订版，因此配置了 `--set defaultRevision=default` 参数。
    {{< /warning >}}

    {{< text syntax=bash snip_id=install_base >}}
    $ helm install istio-base istio/base -n istio-system --set defaultRevision=default --create-namespace
    {{< /text >}}

1. 使用 `helm ls` 命令验证 CRD 的安装情况：

    {{< text syntax=bash >}}
    $ helm ls -n istio-system
    NAME       NAMESPACE    REVISION UPDATED                                 STATUS   CHART        APP VERSION
    istio-base istio-system 1        2024-04-17 22:14:45.964722028 +0000 UTC deployed base-{{< istio_full_version >}}  {{< istio_full_version >}}
    {{< /text >}}

   在输出中找到 `istio-base` 的条目，并确保状态已被设置为 `deployed`。

1. 如果您打算使用 Istio CNI Chart，那您现在就必须这样操作。
   请参阅[通过 CNI 插件安装 Istio](/zh/docs/setup/additional-setup/cni/#installing-with-helm)了解更多信息。

1. 安装 Istio Discovery Chart，它用于部署 `istiod` 服务：

    {{< text syntax=bash snip_id=install_discovery >}}
    $ helm install istiod istio/istiod -n istio-system --wait
    {{< /text >}}

1. 验证 Istio Discovery Chart 的安装情况：

    {{< text syntax=bash >}}
    $ helm ls -n istio-system
    NAME       NAMESPACE    REVISION UPDATED                                 STATUS   CHART         APP VERSION
    istio-base istio-system 1        2024-04-17 22:14:45.964722028 +0000 UTC deployed base-{{< istio_full_version >}}   {{< istio_full_version >}}
    istiod     istio-system 1        2024-04-17 22:14:45.964722028 +0000 UTC deployed istiod-{{< istio_full_version >}} {{< istio_full_version >}}
    {{< /text >}}

1. 获取已安装的 Helm Chart 的状态，确保它已部署:

    {{< text syntax=bash >}}
    $ helm status istiod -n istio-system
    NAME: istiod
    LAST DEPLOYED: Fri Jan 20 22:00:44 2023
    NAMESPACE: istio-system
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    "istiod" successfully installed!

    To learn more about the release, try:
      $ helm status istiod
      $ helm get all istiod

    Next steps:
      * Deploy a Gateway: https://istio.io/latest/docs/setup/additional-setup/gateway/
      * Try out our tasks to get started on common configurations:
        * https://istio.io/latest/docs/tasks/traffic-management
        * https://istio.io/latest/docs/tasks/security/
        * https://istio.io/latest/docs/tasks/policy-enforcement/
        * https://istio.io/latest/docs/tasks/policy-enforcement/
      * Review the list of actively supported releases, CVE publications and our hardening guide:
        * https://istio.io/latest/docs/releases/supported-releases/
        * https://istio.io/latest/news/security/
        * https://istio.io/latest/docs/ops/best-practices/security/

    For further documentation see https://istio.io website

    Tell us how your install/upgrade experience went at https://forms.gle/99uiMML96AmsXY5d6
    {{< /text >}}

1. 检查 `istiod` 服务是否安装成功，确认其 Pod 是否正在运行:

    {{< text syntax=bash >}}
    $ kubectl get deployments -n istio-system --output wide
    NAME     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                         SELECTOR
    istiod   1/1     1            1           10m   discovery    docker.io/istio/pilot:{{< istio_full_version >}}   istio=pilot
    {{< /text >}}

1. （可选）安装 Istio 的入站网关：

    {{< text syntax=bash snip_id=install_ingressgateway >}}
    $ kubectl create namespace istio-ingress
    $ helm install istio-ingress istio/gateway -n istio-ingress --wait
    {{< /text >}}

    参阅[安装网关](/zh/docs/setup/additional-setup/gateway/)以获得关于网关安装的详细文档。

    {{< warning >}}
    网关被部署的命名空间不得具有 `istio-injection=disabled` 标签。
    有关更多信息，请参见[控制注入策略](/zh/docs/setup/additional-setup/sidecar-injection/#controlling-the-injection-policy)。
    {{< /warning >}}

{{< tip >}}
有关如何使用 Helm 后期渲染器自定义 Helm Chart 的详细文档，
请参见[高级 Helm Chart 自定义](/zh/docs/setup/additional-setup/customize-installation-helm/)。
{{< /tip >}}

## 更新 Istio 配置 {#updating-your-configuration}

您可以用自己的安装参数，覆盖掉前面用到的 Istio Helm Chart 的默认行为，
然后按照 Helm 升级流程来定制安装您的 Istio 网格系统。
至于可用的配置项，您可以通过使用 `helm show values istio/<chart>` 来找到配置。
例如 `helm show values istio/gateway`。

### 从非 Helm 安装迁移 {#migrating-from-non-helm-installations}

如果你要从使用 `istioctl` 安装的 Istio 版本迁移到 Helm（Istio 1.5 或更早版本），
则需要删除当前的 Istio 控制平面资源，并根据上面的说明，
使用 Helm 重新安装 Istio。在删除当前 Istio 时，
千万不能删掉 Istio 的自定义资源定义（CRD），以免丢掉您的自定义 Istio 资源。

{{< warning >}}
建议：从集群中删除 Istio 前，使用上面的说明备份您的 Istio 资源。
{{< /warning >}}

您可以按照 [Istioctl 卸载指南](/zh/docs/setup/install/istioctl#uninstall-istio)中提到的步骤进行操作。

## 卸载 {#uninstall}

您可以通过卸载上述安装的 Chart，以便卸载 Istio 和及其组件。

1. 列出在命名空间 `istio-system` 中安装的所有 Istio Chart：

    {{< text syntax=bash snip_id=helm_ls >}}
    $ helm ls -n istio-system
    NAME       NAMESPACE    REVISION UPDATED                                 STATUS   CHART         APP VERSION
    istio-base istio-system 1        2024-04-17 22:14:45.964722028 +0000 UTC deployed base-{{< istio_full_version >}}   {{< istio_full_version >}}
    istiod     istio-system 1        2024-04-17 22:14:45.964722028 +0000 UTC deployed istiod-{{< istio_full_version >}} {{< istio_full_version >}}
    {{< /text >}}

1. （可选）删除 Istio 的所有网关 Chart：

    {{< text syntax=bash snip_id=delete_delete_gateway_charts >}}
    $ helm delete istio-ingress -n istio-ingress
    $ kubectl delete namespace istio-ingress
    {{< /text >}}

1. 删除 Istio Discovery Chart：

    {{< text syntax=bash snip_id=helm_delete_discovery_chart >}}
    $ helm delete istiod -n istio-system
    {{< /text >}}

1. 删除 Istio Base Chart：

    {{< tip >}}
    从设计角度而言，通过 Helm 删除 Chart 并不会删除通过该 Chart 安装的 CRD。
    {{< /tip >}}

    {{< text syntax=bash snip_id=helm_delete_base_chart >}}
    $ helm delete istio-base -n istio-system
    {{< /text >}}

1. 删除命名空间 `istio-system`：

    {{< text syntax=bash snip_id=delete_istio_system_namespace >}}
    $ kubectl delete namespace istio-system
    {{< /text >}}

## 卸载稳定的修订版标签资源 {#uninstall-stable-revision-label-resources}

如果您决定继续使用旧的控制平面不更新，您可以通过第一次发布来卸载较新的版本及其标记
`helm template istiod istio/istiod -s templates/revision-tags.yaml --set revisionTags={prod-canary} --set revision=canary -n istio-system | kubectl delete -f -`。
您必须按照上述卸载步骤卸载 Istio 的修订版。

如果您使用就地升级安装了此版本的网关，则还必须手动重新安装上一个版本的网关，
移除以前的版本及其标记不会自动恢复以前就地升级的网关。

### （可选）删除 Istio 安装的 CRD {#deleting-customer-resource-definition-installed}

永久删除 CRD 会移除您在集群中已创建的所有 Istio 资源。
用下面命令永久删除集群中安装的 Istio CRD：

{{< text syntax=bash snip_id=delete_crds >}}
$ kubectl get crd -oname | grep --color=never 'istio.io' | xargs kubectl delete
{{< /text >}}

## 安装前生成清单 {#generate-a-manifest-before-installation}

您可以在安装 Istio 之前使用 `helm template` 子命令为每个组件生成清单。
例如，要为 `istiod` 组件生成可以使用 `kubectl` 安装的清单：

{{< text syntax=bash snip_id=none >}}
$ helm template istiod istio/istiod -n istio-system --kube-version {Kubernetes version of target cluster} > istiod.yaml
{{< /text >}}

生成的清单可用于检查具体安装了什么以及跟踪清单随时间的变化。

{{< tip >}}
您通常用于安装的任何其他标志或自定义值覆盖也应提供给 `helm template` 命令。
{{< /tip >}}

要安装上面生成的清单，它将在目标集群中创建 `istiod` 组件：

{{< text syntax=bash snip_id=none >}}
$ kubectl apply -f istiod.yaml
{{< /text >}}

{{< warning >}}
如果尝试使用 `helm template` 安装和管理 Istio，请注意以下注意事项：

1. 必须手动创建 Istio 命名空间（默认为 `istio-system`）。

1. 资源可能未按照与 `helm install` 相同的依赖顺序进行安装

1. 此方法尚未作为 Istio 版本的一部分进行测试。

1. 虽然 `helm install` 会自动从 Kubernetes 上下文中检测特定于环境的设置，
   但 `helm template` 无法做到这一点，因为它是离线运行的，
   这可能会导致意外结果。特别是，如果您的 Kubernetes 环境不支持第三方服务帐户令牌，
   您必须确保遵循[这些步骤](/zh/docs/ops/best-practices/security/#configure-third-party-service-account-tokens)。

1. 由于集群中的资源没有按正确的顺序可用，生成的清单的 `kubectl apply` 可能会显示瞬态错误。

1. `helm install` 会自动修剪配置更改时应删除的任何资源（例如，如果您删除网关）。
   当您将 `helm template` 与 `kubectl` 一起使用时，
   不会发生这种情况，必须手动删除这些资源。

{{< /warning >}}
