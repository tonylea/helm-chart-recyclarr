# Changelog

All notable changes to this project will be documented in this file. See [commit-and-tag-version](https://github.com/absolute-version/commit-and-tag-version) for commit guidelines.

## 1.0.0 (2026-11-22)

### Features

* run Recyclarr v8's sync on a schedule, as a non-root CronJob ([d3f516e](https://github.com/tonylea/helm-chart-recyclarr/commit/d3f516e91aac41ef615f1976ccd98eb19709b342))

### Bug Fixes

* default to a daily run at 04:00, London time ([0c168ac](https://github.com/tonylea/helm-chart-recyclarr/commit/0c168ac4556a45e245993c45fe53e103e625224b))
* retry a failed run with Kubernetes' default back-off ([5b07b22](https://github.com/tonylea/helm-chart-recyclarr/commit/5b07b2241ec60c9861fced03f73125ce1eeb4b89))

### Documentation

* describe the chart, its values and how to contribute ([81afafa](https://github.com/tonylea/helm-chart-recyclarr/commit/81afafa6b153c8d1753fe472056610657e14df48))
