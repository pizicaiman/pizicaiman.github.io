---
permalink: /
title: ""
excerpt: ""
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

{% if site.google_scholar_stats_use_cdn %}
{% assign gsDataBaseUrl = "https://cdn.jsdelivr.net/gh/" | append: site.repository | append: "@" %}
{% else %}
{% assign gsDataBaseUrl = "https://raw.githubusercontent.com/" | append: site.repository | append: "/" %}
{% endif %}
{% assign url = gsDataBaseUrl | append: "google-scholar-stats/gs_data_shieldsio.json" %}

# 🎉 About Me
<span class='anchor' id='about-me'></span>

1、践行 Cloud Native IT 理念，深入掌握 DevOps 文化与实践，熟练设计并落地基于 GitLab CI/Jenkins + Helm + Argo CD 的 CI/CD 流水线；主导微服务架构治理，推动服务拆分、接口标准化与容器化改造，实现从代码提交到生产部署的端到端自动化交付。

2、具备大规模 Kubernetes 多集群生产维护经验，在 AWS、阿里云等主流公有云环境中，落地弹性负载（HPA/VPA）、动态调度（亲和性/反亲和性/taint-toleration）、自动伸缩（基于 CPU/内存/自定义指标）、分布式链路追踪（Jaeger/OpenTelemetry）、服务网格（Istio/Linkerd）及全栈可观测性（Prometheus + Grafana + Loki + Tempo）等云原生核心能力，支撑高并发电商核心业务稳定运行。

3、高度关注容器系统安全与服务间通信安全，实施 Pod 安全策略（PodSecurity）、网络策略（NetworkPolicy）、镜像漏洞扫描（Trivy）及 TLS 双向认证（mTLS）；在服务网格中强化服务间身份认证与细粒度访问控制，确保微服务通信符合零信任安全模型。

# 🔥 News
- *2022.02*: &nbsp;🎉🎉 Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus ornare aliquet ipsum, ac tempus justo dapibus sit amet. 
- *2022.02*: &nbsp;🎉🎉 Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus ornare aliquet ipsum, ac tempus justo dapibus sit amet. 

# 📝 Publications 

<!-- 
这是在 Markdown 文件中嵌入的 HTML 代码块。
在 Markdown 里，普通的 Markdown 语法（如#标题、- 列表、[链接](url)等）主要用于文本格式化；
但是，当你需要实现更复杂的布局、样式，或者 Markdown 本身不支持的效果时，就会直接写原生 HTML标签。
例如：
<div> ... </div>

同时，这个例子中 `<div class='paper-box-text' markdown="1">` 这种写法的意思是：
- 在 Jekyll（或部分支持该写法的 Markdown 渲染器，如 kramdown）里，给 div 标签加上属性 `markdown="1"`，就代表 **div 内部的内容会被当作 markdown 再渲染**。
- 比如你可以在该 div 里正常写 Markdown 语法（如粗体、链接、列表），最终会被转成对应的 HTML。

所以，这种写法本质上是为了在 HTML 容器中继续用 markdown 风格写内容，同时还能通过 HTML 标签实现样式或布局自定义扩展。
-->
<div class='paper-box'>
  <div class='paper-box-image'>
    <div>
      <div class="badge">CVPR 2016</div>
      <img src='images/20246241456216835.jpg' alt="sym" width="100%">
    </div>
  </div>
  <div class='paper-box-text' markdown="1">
[Deep Residual Learning for Image Recognition](https://openaccess.thecvf.com/content_cvpr_2016/papers/He_Deep_Residual_Learning_CVPR_2016_paper.pdf)

**Kaiming He**, Xiangyu Zhang, Shaoqing Ren, Jian Sun

[**Project**](https://scholar.google.com/citations?view_op=view_citation&hl=zh-CN&user=DhtAFkwAAAAJ&citation_for_view=DhtAFkwAAAAJ:ALROH1vI_8AC) <strong><span class='show_paper_citations' data='DhtAFkwAAAAJ:ALROH1vI_8AC'></span></strong>
  - Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus ornare aliquet ipsum, ac tempus justo dapibus sit amet. 
</div>
</div>

- [Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus ornare aliquet ipsum, ac tempus justo dapibus sit amet](https://github.com)

- [负责多地域 Kubernetes 集群的统一管理与维护，覆盖核心组件（kube-apiserver、etcd、scheduler）版本升级、CNI 网络插件（Calico/Cilium）配置、CSI 存储集成、日志采集（Filebeat → logtail）及服务发现（CoreDNS + Kubernetes Service）等全栈能力建设。](https://github.com)

- [主导传统业务架构向云原生演进，完成数十个核心业务模块的容器化改造，基于 Kubernetes 实现标准化部署、任务调度与生命周期管理，提升交付效率与资源利用率。](https://github.com)

- [推动集团运维自动化与 GitOps 落地，基于 Git 作为唯一事实源，结合 Helm 应用模板与 Kubernetes API，构建自动化 CI/CD 流水线，实现应用版本的可审计、可回滚、高频安全迭代。](https://github.com)

- [设计并实现跨地域多集群服务发现与流量调度方案，通过统一网关（ALB/Ingress）集中管理南北向流量，集成 JWT/OAuth 等认证鉴权机制，保障服务访问安全与路由一致性。](https://github.com)
  
- [主导资源调度与成本优化策略，包括事件驱动弹性扩缩容（基于自定义指标）、kube-scheduler 调优、命名空间级 ResourceQuota 治理、Pod 安全策略（PodSecurity）、节点资源超卖、在线/离线混部调度、应用潮汐部署策略、Spot 实例整合等，在保障 SLO 前提下显著提升资源 ROI。](https://github.com)

- [构建集群与业务资源水位量化监控与告警体系，建立容量规划模型、混沌工程演练机制与预案自动化执行流程，从可靠性、可观测性与资源效率三位一体视角，实现风险前置识别与主动干预。](https://github.com)

- [倡导并推进集团 SRE 体系建设，定义核心业务 SLI/SLO/SLA 标准，设计告警降噪规则（如基于 PromQL 的动态阈值）与故障自愈流程（如异常 Pod 自动重建），有效缩短 MTTR 超 50%。](https://github.com)

- [持续跟踪云原生生态前沿技术（如 Cilium、Kueue、Ray on K8s），结合业务负载特征评估新架构组件可行性，并推动在 AI 训练、批处理等场景的试点落地。](https://github.com)

- [积极探索 LLM 在智能运维（AIOps）中的应用潜力，通过本地实验（基于开源模型如 Qwen2-7B + vLLM + RAG 架构），验证其在 Prometheus 告警解读、Kubernetes 事件分析、运维知识问答等场景的可行性；强调“检索增强”对降低幻觉、提升可操作性的价值，为未来 AIOps 落地积累技术认知。](https://github.com)

# 🏅 荣誉与奖项

<div class="honors-list">
  <ol>
    <li>
      <strong>显著提升应用部署效率与资源利用率：</strong><br>
      通过标准化容器镜像构建、Helm Chart 模板化部署及 GitOps 自动化流水线，将新服务上线周期从数小时缩短至 10 分钟以内；结合资源请求（<code>requests</code>）与限制（<code>limits</code>）的精细化配置、命名空间配额管理及垂直/水平资源优化策略，集群整体 CPU 与内存利用率提升 40% 以上，有效降低基础设施成本。
    </li>
    <li>
      <strong>实现分钟级弹性扩缩容，敏捷应对业务高峰：</strong><br>
      基于 Kubernetes HPA（Horizontal Pod Autoscaler）及 KEDA（Kubernetes Event-driven Autoscaling），构建多维度弹性伸缩机制，支持 CPU/内存、自定义业务指标（如消息队列积压、API QPS）等触发条件；在大促、秒杀等高并发场景下，服务可在 2–3 分钟内完成自动扩容，峰值承载能力提升 3 倍，同时空闲时段自动缩容，避免资源浪费。
    </li>
    <li>
      <strong>大幅降低运维人力成本，显著缩短故障恢复时间（MTTR）：</strong><br>
      通过集成 Prometheus、Alertmanager、Loki 与 Grafana 构建统一可观测性体系，实现日志、指标、告警联动；结合自动化自愈策略（如 Pod 自动重启、节点驱逐、服务熔断）与标准化应急响应流程，平均故障定位与恢复时间（MTTR）缩短 60% 以上；日常运维操作（如版本升级、配置变更）100% 自动化，释放 70% 重复性人力投入。
    </li>
    <li>
      <strong>为微服务架构演进与云原生技术体系落地奠定坚实基础：</strong><br>
      容器化改造过程中同步推进应用解耦与接口标准化，为后续服务网格（如 Istio/Linkerd）接入、分布式追踪、混沌工程等云原生能力预留架构接口；同时建立 DevOps 协作规范与平台工程（Platform Engineering）能力，支持多团队在统一 Kubernetes 平台上高效、安全地交付业务，加速企业整体云原生转型进程。
    </li>
  </ol>
</div>

# 📖 Educations
- *2019.06 - 2022.04 (now)*, Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus ornare aliquet ipsum, ac tempus justo dapibus sit amet. 
- *2015.09 - 2019.06*, Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus ornare aliquet ipsum, ac tempus justo dapibus sit amet. 

# 💬 Invited Talks
- *2021.06*, Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus ornare aliquet ipsum, ac tempus justo dapibus sit amet. 
- *2021.03*, Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus ornare aliquet ipsum, ac tempus justo dapibus sit amet.  \| [\[video\]](https://github.com/)

# 💻 Internships
- *2019.05 - 2020.02*, [Lorem](https://github.com/), China.