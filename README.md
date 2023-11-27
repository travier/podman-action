# Container to run buildah/podman/skopeo commands in GitHub Actions

Based on the official [podman
container](https://quay.io/repository/podman/stable) with buildah and skopeo
included on top.

## How to use

Use the podman, buildah and skopeo actions from
[github.com/redhat-actions](https://github.com/redhat-actions) and specify this
container image for your jobs:

```
jobs:
  build-push-image:
    runs-on: ubuntu-latest
    container:
      image: quay.io/travier/podman-action
      options: --privileged
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Build container image
        uses: redhat-actions/buildah-build@v2
        with:
          image: foo
          tags: latest
          containerfiles: Containerfile
          layers: false
          oci: true

      - name: Push to Container Registry
        uses: redhat-actions/push-to-registry@v2
        id: push
        if: (github.event_name == 'push' || github.event_name == 'schedule') && github.ref == 'refs/heads/main'
        with:
          username: ${{ secrets.BOT_USERNAME }}
          password: ${{ secrets.BOT_SECRET }}
          image: foo
          registry: quay.io/bar
          tags: latest
```

## Verifying sigstore container signatures with podman

How to configure sigstore signature verification in podman:

```
$ sudo mkdir /etc/pki/containers
$ sudo cp quay-travier-containers.pub /etc/pki/containers/
$ sudo restorecon -RFv /etc/pki/containers

$ cat /etc/containers/registries.d/quay.io-travier.yaml
docker:
  quay.io/travier:
    use-sigstore-attachments: true
$ sudo restorecon -RFv /etc/containers/registries.d/quay.io-travier.yaml

$ cat /etc/containers/policy.json
...
    "transports": {
        "docker": {
            "quay.io/travier": [
                {
                    "type": "sigstoreSigned",
                    "keyPath": "/etc/pki/containers/quay-travier-containers.pub",
                    "signedIdentity": {
                        "type": "matchRepository"
                    }
                }
            ],
...
```

## License

See [LICENSE](LICENSE).
