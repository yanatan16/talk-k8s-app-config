## Application Configuration Strategies In Kubernetes

_Jon Eisen_

Colorado Kubernetes Meetup

March 31, 2016

http://joneisen.me/talk-k8s-app-config

----

## Core Problems in Cluster Management

- Service Discovery
- Application Monitoring
- Application Configuration

----

## Application Configuration Defined

1. Get configs into app
2. Limit app-specific code
3. Reload app on config change
4. Easy to use system

----

## The Solutions

- Bundle configs into image
- Etcd
- Kubernetes Secrets
- Kubernetes ConfigMap

----

### Bundle Configs into Image

Pro: Images can be shipped and run easily

---

Cons:
- K8s doesn't support different image per pod in RC
- Requires build server at low scale
- Requires many images per app
- Hard to have reload-on-change

----

### Etcd

Pros:
- Store whatever you want in `etcd`
- Reload-on-change easy to implement

Cons:
- Requires app-specific code or special container in pod
- Works outside of k8s
- Must use raw `etcdctl` to edit

____

#### `confd`

`confd` can configure and reload an application from etcd keys.

```ini
[template]
src = "nginx.conf.tmpl"
dest = "/etc/nginx/nginx.conf"
keys = [
  "/nginx",
]
check_cmd = "/usr/sbin/nginx -t -c {{.src}}"
reload_cmd = "/usr/sbin/service nginx restart"
```

How to get `confd` config into the container?

----

### Secrets

Pros:
- **Built-in** to Kubernetes v1.0.
- Automagically mount into your containers
- Stores arbitrary data

Cons:
- No reload-on-change
- Requires base64 encoding to use (until v1.2)
- Doesn't support environment variables natively

____

#### Using Secrets

To create:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: dmFsdWUtMg0K
  username: dmFsdWUtMQ0K
```

To mount as a volume:
```yaml
volumes:
  - name: foo
  - secret:
    secretName: mysecret
```

____

#### Using Secrets

- Use them as files ready for use in a program (e.g. ssh keys)
- Use them as environment configurations

```
KEY1=VALUE1
KEY2=VALUE2
```

*Protip*: You can read in environment files using a Docker `ENTRYPOINT`

----

### ConfigMap

A [proposed feature](https://github.com/eBay/Kubernetes/blob/master/docs/proposals/configmap.md) for Kubernetes v1.2

Like Secrets, but:

- Directly supports environment variables
- No base64 cumbersomeness
- Reload-on-change capability

----

## Conclusion

- Bundling configs into your images is too much work!
- Using etcd is powerful but a lot of work
- Secrets are great, but have problems
- Get **stoked for ConfigMap**!

----

# Thanks!

Jon Eisen
@jm_eisen
http://joneisen.works
Blog: http://joneisen.me
