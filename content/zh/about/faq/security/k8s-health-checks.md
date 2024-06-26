---
title: 启用双向 TLS 认证时应该如何使用 Kubernetes liveness 和 readiness 对服务进行健康检查？
weight: 50
---

如果启用了双向 TLS 认证，则来自 kubelet 的 HTTP 和 TCP
健康检查将不能正常工作，因为 kubelet 没有 Istio 颁发的证书。

有几种选择：

1. 使用 probe rewrite 将 liveness 和 readiness 的请求直接重定向到工作负载。
   有关更多信息，请参阅 [Probe Rewrite](/zh/docs/ops/configuration/mesh/app-health-check/#probe-rewrite)。

1. 使用单独的端口进行健康检查，并且仅在常规服务端口上启用双向 TLS。
   有关更多信息，请参阅 [Istio 服务的运行状况检查](/zh/docs/ops/configuration/mesh/app-health-check/#separate-port)。

1. 如果对 Istio 服务使用 [`PERMISSIVE` 模式](/zh/docs/tasks/security/authentication/mtls-migration)，
   那么他们可以接受 HTTP 和双向 TLS 流量。请记住，由于其他人可以通过 HTTP 流量与该服务进行通信，
   因此不强制执行双向 TLS。
