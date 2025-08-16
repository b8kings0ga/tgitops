# tgitops

this repo is for save all gitops related files and documents.



## Structure

The directory structure is as follows:

```
.
├── envs/
│   └── dev/
│       └── us-east-1/
│           └── my-app-a/
│               └── values.yaml
├── appsets/
│   └── dev/
│       ├── services.yaml
│       └── crossplane.yaml
├── crossplane/
│   └── databases/
│       └── my-app-a-db.yaml
├── vclusters/
│   └── dev-vcluster-manifest.yaml
├── charts/
│   └── golden-charts/
│       └── Chart.yaml
└── README.md
```

For demo purposes, we have a single `dev` environment in `us-east-1` with a single application `my-app-a`.

## helm charts

we put the golden-charts in the charts/ directory. this chart is used as a common base chart for all applications, will be in standard helm chart format, with minimal configuration.

## vclusters

each real applications will be running on vcluster of the real kubernetes local cluster. 


## appsets
this is the argocd application sets configuration. for 