# Recyclarr Helm Chart

![Downloads](https://img.shields.io/github/downloads/tonylea/helm-chart-recyclarr/total)

Deploys [Recyclarr](https://github.com/recyclarr/recyclarr) v8 from its official image, [`ghcr.io/recyclarr/recyclarr`](https://github.com/recyclarr/recyclarr/pkgs/container/recyclarr).

Recyclarr runs as a Kubernetes CronJob: each run is one `recyclarr sync` that exits when it's done, and a run never starts while the last one is still going.
Recyclarr's state, which records the custom formats it created, is kept on a persistent volume at `/config`.

## Values

| Value | Default |
| --- | --- |
| `image.tag` | the chart's `appVersion` |
| `user` | `1000` |
| `group` | `1000` |
| `schedule` | `@daily` |
| `timeZone` | `Etc/UTC` |
| `configs` | none; at least one required |
| `env` | none |
| `resources` | none |
| `persistence.existingClaim` | none; the chart creates a claim |
| `persistence.storageClassName` | the cluster's default class |
| `persistence.size` | `1Gi` |
| `affinity` | none |

`configs` maps a file name to the content of a [Recyclarr config file](https://recyclarr.dev/); each is mounted read-only in `/config/configs`, where Recyclarr loads every file.
`env` takes Kubernetes env entries, so an API key can come from a Secret and be read in a config with `!env_var`.
`timeZone` is used for the schedule and given to Recyclarr as `TZ`.

## Fixed settings

- The image's built-in cron mode isn't used: the CronJob runs `recyclarr sync` directly, so each run is a Job whose success or failure Kubernetes reports.
- Recyclarr runs as `user` and `group`, never root, with a read-only root filesystem and no capabilities.
  `/tmp` and the data directory, which holds the guide clones and logs, are emptyDirs, filled again on each run.
- The config files are mounted read-only, so they can only be changed through `configs`.
  Commands that edit them, such as `recyclarr config create`, don't work.

## Requirements

- Sonarr or Radarr, reachable from the namespace, and a Secret with each one's API key.
- A storage class for the claim the chart creates, the cluster's default unless named.
  The state has to persist: `delete_old_custom_formats` relies on it to know which custom formats are Recyclarr's.

## Usage

```sh
helm repo add recyclarr https://tonylea.github.io/helm-chart-recyclarr
helm install recyclarr recyclarr/recyclarr --values my-values.yaml
```

```yaml
schedule: "0 4 * * *"
timeZone: Europe/London
env:
  - name: SONARR_API_KEY
    valueFrom:
      secretKeyRef:
        name: sonarr
        key: api-key
configs:
  sonarr.yml: |
    sonarr:
      series:
        base_url: http://sonarr:8989
        api_key: !env_var SONARR_API_KEY
        quality_definition:
          type: series
```

## Development

Pull requests run:

- `helm lint --strict`, and the rendered chart checked by kubeconform against Kubernetes 1.37.
- helm-unittest's tests in `tests/`.
- A fresh install with chart-testing in a kind cluster, against the claim in `ci/kind`.
  kind binds a claim only once a pod uses it, and the CronJob starts none at install, so the claim is made outside the release.

Commits follow Conventional Commits.
Releases are versioned and tagged automatically from `main`, and published to the Helm repository on this repo's GitHub Pages.
