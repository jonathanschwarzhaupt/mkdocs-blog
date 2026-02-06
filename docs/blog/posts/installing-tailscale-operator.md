---
date:
  created: 2026-02-06
tags:
  - k8s
  - GitOps
  - Secret management
categories:
  - Homelab
links:
- External links:
  - Setup Tailscale Operator: https://tailscale.com/docs/features/kubernetes-operator#setup
  - Expose clutser workload to Tailnet: https://tailscale.com/docs/features/kubernetes-operator/how-to/cluster-ingress#prerequisites
---

# Learnings: Installing the Tailscale Operator with Flux

After integrating the [Integrating the 1password operator in my kubernetes cluster](./installing-1password-operator.md) in my homelab, I planned to take on the next challenge. I wanted to make full use of Tailscale and its managed WireGuard platform <!-- more --> that appears to be be aimed at enthusiasts and homelabbing at the free tier.

My goal was to expose cluster workloads to my tailnet. Or in plain english: Access my apps, securely, only on my personal devices.

---

## My approach 

I followed the official documentation and was able to set up the operator and migrate my first service in under three hours.

After completing the steps in the Tailscale admin dashboard, mainly creating the ACL tags, it was time to "translate" the Helm commands the documentation shows into the `repository.yaml` and `release.yaml` that Flux requires.

Given I had deployed a few helm repositories already, and the 1password operator recently, I could make good reference of my previous work. The challenging part was in correctly creating and referencing the OAuth credentials from Tailscale. During this process I was very happy that I created the 1password operator previously, as it really made it a breeze compared to my previous sops workflow.

With the operator deployed successfully (with the tailscale namespace created), I could put it to use! I first annotated the existing linkding service to verify that linkding would show up as a machine in my Tailnet. After that test went successfully and I could access linkding via HTTP, I created an ingress resource with the Tailscale incress class. With the incress resource created, I could access linkding via HTTPS with TLS certificates automatically managed by Tailscale.

## Implementing Tailscale Operator with GitOps 
### Repository.yaml

A simple resource definition that adds the Helm repository to my cluster.

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: tailscale
  namespace: tailscale
spec:
  interval: 24h
  url: https://pkgs.tailscale.com/helmcharts

```

### Release.yaml

The actual chart that should be deployed. By specifying `version: "1.x"` I let Flux automatically update minor and patch versions.

The key with this resource was to correctly reference and pass the OAuth credentials to the fields the chart expects. I could follow the  `valuesFrom` "pattern" that I used for the 1password operator previously - it worked like a charm.

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: tailscale
  namespace: tailscale
spec:
  interval: 30m
  chart:
    spec:
      chart: tailscale-operator
      version: "1.x"
      sourceRef:
        kind: HelmRepository
        name: tailscale
        namespace: tailscale
      interval: 12h

  valuesFrom:
    - kind: Secret
      name: tailscale-credentials
      valuesKey: username
      targetPath: oauth.clientId
    - kind: Secret
      name: tailscale-credentials
      valuesKey: password
      targetPath: oauth.clientSecret
```

### Linkding ingress

The last resource left to define was the ingress resource for the application I wanted to expose to my devices.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: linkding
spec:
  defaultBackend:
    service:
      name: linkding
      port:
        number: 9090
  ingressClassName: tailscale
  tls:
    - hosts:
        - linkding
```

The `defaultBackend` routes in the absence of `rules` all requests to the specified service (by name).

## Outlook

And that's it! Overall it was a breeze to set up the Tailscale operator. I now plan to migrate my other services off of Cloudflare tunnels and to the Tailscale operator. I can sleep a bit better at night knowing that - even though the applications in my homelab all use strong passwords - my devices are the only ones that can access my applications. 

It is another layer of defense, one I highly recommend.

With this new setup up and running, I have ideas and plans to extend and manifest it even further. Since I also have pihole running as my DNS resolver, I plan to add custom DNS records to access my applications via funny names: Why not something like: "bookmarks.jonathan.home". 

Another topic I would like to go deeper into is strengthening the security of my Tailnet and its devices using Tailscale's ACL rules. As of now, every device that I own can "contact" every device. Ideally I would like to be able to run applications for my parents and properly separate traffic.
