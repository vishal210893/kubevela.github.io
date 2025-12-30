---
title: "State of KubeVela 2025: A Year of Maturity, Community, and Looking Ahead"
author: Anoop Gopalakrishnan
author_title: KubeVela Team
author_url: https://github.com/anoop2811
author_image_url: https://avatars.githubusercontent.com/u/2038273?v=4
tags: [KubeVela, "2025", community, roadmap, CNCF, year-in-review]
description: "Reflecting on KubeVela's achievements in 2025 and our vision for the future of application delivery"
image: https://raw.githubusercontent.com/oam-dev/KubeVela.io/main/docs/resources/KubeVela-03.png
hide_table_of_contents: false
---

As we close out 2025, we want to take a moment to reflect on what has been a transformative year for KubeVela. From major releases to community growth, from enterprise adoption to exciting new features on the horizon—this year has reinforced our belief that making application delivery enjoyable is not just a tagline, but a mission worth pursuing.

<!--truncate-->

## 2025 By The Numbers

Before we dive into the details, here's what the KubeVela community accomplished this year:

| Metric           | 2025                        |
| ---------------- | --------------------------- |
| **Releases**     | [7 (v1.10.0 through v1.10.6)](https://github.com/kubevela/kubevela/releases) |
| **Commits**      | [124](https://github.com/kubevela/kubevela/compare/v1.9.13...v1.10.6) |
| **Contributors** | [33 unique contributors](https://github.com/kubevela/kubevela/graphs/contributors) |
| **Bug Fixes**    | [51](https://github.com/kubevela/kubevela/commits/master/?since=2025-01-01&until=2025-12-31) |
| **GitHub Stars** | [7,600+](https://github.com/kubevela/kubevela/stargazers) |
| **Forks**        | [960+](https://github.com/kubevela/kubevela/network/members) |

These numbers tell a story of a healthy, active open-source project—but the real story is in the features we shipped and the problems we solved.

## The 1.10 Release Series: Maturity in Focus

2025 was the year of the 1.10 release series, and our theme was clear: **maturity and reliability**. We listened to our users—platform engineers running KubeVela in production—and focused on the features that make their lives easier.

### Kubernetes Compatibility

We continue to invest in ensuring KubeVela works seamlessly with recent Kubernetes releases. Staying current with the Kubernetes ecosystem is critical for enterprise adoption, and we regularly update our dependencies, API versions, and test matrices to keep KubeVela on the leading edge.

### StatefulSet Support

One of our most requested features finally landed in v1.10.3: native **StatefulSet support** ([#6638](https://github.com/kubevela/kubevela/pull/6638)). Platform engineers can now define stateful workloads—databases, message queues, distributed caches—with the same elegance as stateless services. This opens up KubeVela to a whole new class of applications.

### CueX Compiler for Templating

Building on the CueX compiler framework originally developed for workflows, we expanded CueX support to **component and trait templating** ([#6720](https://github.com/kubevela/kubevela/pull/6720)). This extension unlocks new advanced capabilities and lays the groundwork for future templating improvements across KubeVela.

### Dependency-Aware Workflows

[Issue #6852](https://github.com/kubevela/kubevela/issues/6852) was one of our most upvoted issues—users were confused about why component `dependsOn` wasn't respected in workflows. In v1.10.4, we fixed this. KubeVela now dynamically adds workflow dependencies between steps according to component dependencies.

This was a small change in code but a big change in user experience. It's the kind of improvement that makes you wonder, "Why didn't it always work this way?" And that's exactly the point—**good software should be intuitive**.

### Enhanced Status Reporting

Platform engineers told us they needed more visibility into what's happening with their applications. The existing health checks were helpful, but they wanted more. So we added **Status Details**—a new field that captures structured diagnostic information about your components and traits.

```yaml
status:
  services:
    - name: my-service
      status:
        readyReplicas: "3"
        totalReplicas: "3"
        readyPercentage: "100"
        deploymentMode: "RollingUpdate"
```

This is the kind of information that makes debugging at 2 AM a little less painful.

### Application Status Metrics

Speaking of observability, we added new Prometheus metrics for application health:

- `kubevela_application_health_status`
- `kubevela_application_phase`
- `kubevela_application_workflow_phase`

Now you can set up alerts, build dashboards, and integrate KubeVela into your existing monitoring stack. We've kept these metrics low-cardinality to ensure they're cost-effective to collect and store.

### Security Enhancements

Security was a major theme in 2025. We shipped several improvements that make KubeVela safer for production use:

- **securityContext and podSecurityContext traits** ([#6666](https://github.com/kubevela/kubevela/pull/6666)): Configure pod and container security settings directly in your component definitions
- **Definition permission validation** ([#6876](https://github.com/kubevela/kubevela/pull/6876)): KubeVela now validates that your Definitions have the necessary permissions at application creation time, catching misconfigurations early
- **Fail-fast CUE validation** ([#6774](https://github.com/kubevela/kubevela/pull/6774)): Required parameters are now validated upfront, preventing deployments that would fail later

### Supply Chain Security

In an era where software supply chain attacks are increasingly common, we're proud to have shipped **signed releases, SBOMs, and SLSA provenance** for all KubeVela artifacts. When you pull a KubeVela container image, you can verify it came from us and see exactly what's inside.

### Semantic Versioning for Definitions

Platform engineers managing large definition libraries will appreciate the new **semantic versioning support for Definitions** ([#6648](https://github.com/kubevela/kubevela/pull/6648)). You can now version your ComponentDefinitions and TraitDefinitions, making it easier to evolve your platform APIs without breaking existing applications.

## Looking Ahead: The Future of Application Delivery

As we look beyond 2025, we find ourselves at an exciting inflection point. The landscape of software delivery is evolving rapidly, and we're paying close attention to the trends that will shape how developers and platform engineers work in the years to come.

### The Rise of Intelligent Platforms

The emergence of AI and large language models is transforming how we interact with complex systems. We're exploring how these capabilities could fundamentally change the KubeVela experience. Imagine describing what you need in natural language and having an intelligent system generate the right configuration, diagnose issues before they become problems, or suggest optimizations based on real-world patterns.

We're not building AI for the sake of AI—we're asking ourselves: _What if deploying and operating applications could be as simple as having a conversation?_

### Lowering Barriers, Not Just Abstracting Complexity

One consistent theme in our community feedback is the desire for simpler paths to productivity. Whether you're a developer who just wants to ship code or a platform engineer building golden paths for your organization, KubeVela should meet you where you are.

We're investing in new ways to author and share platform building blocks—approaches that leverage familiar tools and languages while maintaining the power and flexibility that KubeVela offers today.

### Beyond Deployment: Intelligent Operations

The future of application delivery isn't just about getting code to production—it's about running it well. We're thinking deeply about how KubeVela can provide proactive insights: cost awareness before you deploy, security posture at a glance, performance optimization suggestions based on actual usage patterns.

These are explorations, not promises. We're in the early stages of understanding how these capabilities could benefit our community. But we're genuinely excited about the possibilities, and we believe the intersection of platform engineering and intelligent tooling will define the next era of cloud-native development.

**Stay tuned.** We'll share more as these ideas take shape.

## Community Highlights

Open source is about people, and we've been fortunate to have an incredible community this year.

### New Contributors

We welcomed **33 unique contributors** in 2025, from first-time contributors fixing typos to seasoned developers implementing major features. Every contribution matters, and we're grateful for each one.

### Community Meetings

Our monthly community meetings continue to be a highlight. Each month, we showcase new features, review upcoming milestones, and engage in Q&A. If you haven't joined one, we'd love to see you there. Check our [community calendar](https://github.com/kubevela/community#community-meetings) for details.

### KubeCon North America 2025

KubeCon North America 2025 in Atlanta was a milestone moment for our community. We presented [Democratizing Platform Engineering and Introducing the Component Contributor Architecture](https://www.youtube.com/watch?v=0yW79y48U3k), sharing our vision for how KubeVela enables organizations to build platforms that empower every team to contribute—not just consume.

We also hosted a **Project Pavilion booth** throughout the conference, which gave us invaluable face time with the community. Platform engineers, developers, and operators from organizations of all sizes stopped by to share their experiences, challenges, and ideas for the future. These conversations are the lifeblood of open source—hearing directly from users about what's working and what needs improvement helps us build for the real world. If you visited us at the booth, thank you. Your feedback is already shaping our roadmap.

### CNCF Incubation

As a CNCF Incubating project, we continue to work closely with the foundation and the broader cloud-native ecosystem. We're committed to the open governance and vendor-neutral approach that makes CNCF projects trustworthy for enterprise adoption.

## Lessons Learned

Looking back at 2025, a few lessons stand out:

**1. Listen to your users.** The best features we shipped this year—StatefulSet support, dependency-aware workflows, enhanced status reporting, security traits—all came from user feedback. Keep those GitHub issues coming.

**2. Boring is good.** We didn't ship flashy new features every month. Instead, we focused on stability, compatibility, and reliability. For a project used in production, that's exactly what matters.

**3. Documentation is a feature.** We're the first to admit our documentation needs work. Simplifying the getting-started experience and making it easier for new users to succeed with KubeVela remains a priority for us.

## Thank You

To everyone who used KubeVela in 2025, filed an issue, submitted a PR, joined a community meeting, or just gave us a star on GitHub—**thank you**.

Building an open-source project is a marathon, not a sprint. We're in this for the long haul, and we're grateful to have you on this journey with us.

## Get Involved

Want to be part of the story? Here's how:

- **Star us on GitHub**: [github.com/kubevela/kubevela](https://github.com/kubevela/kubevela)
- **Join our Slack**: [#kubevela on CNCF Slack](https://cloud-native.slack.com/archives/C01BLQ3HTJA)
- **Attend a community meeting**: Every month, all are welcome
- **Contribute**: Check out our [good first issues](https://github.com/kubevela/kubevela/labels/good%20first%20issue)

Here's to making shipping applications more enjoyable. 🚀

---

_The KubeVela Team_

_December 2025_
