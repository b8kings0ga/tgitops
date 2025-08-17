# tgitops

this repo is for save all gitops related files and documents.



## Structure

The directory structure is as follows:

```
.
├── envs/
│   ├── dev/
│   │   ├── us-east-1/
│   │   │   ├── apps/
│   │   │   │   └── my-app-a/
│   │   │   │       └── values.yaml
│   │   │   ├── crossplane/
│   │   │   │   └── my-app-a-db/
│   │   │   │       └── postgres.yaml
│   │   │   └── vclusters/
│   │   │       └── my-team-a-vcluster/
│   │   │           └── vcluster-manifest.yaml
│   │   └── eu-west-1/
│   │       └── ...
│   └── prod/
│       └── ...
```

For demo purposes, we have a single `dev` environment in `us-east-1` with a single application `my-app-a`.

## helm charts

we put the golden-charts in the charts/ directory. this chart is used as a common base chart for all applications, will be in standard helm chart format, with minimal configuration.

## vclusters

each real applications will be running on vcluster of the real kubernetes local cluster. 


## appsets
this is the argocd application sets configuration. for 