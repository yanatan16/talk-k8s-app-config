## Application Configuration Strategies In Kubernetes

_Jon Eisen_

Colorado Kubernetes Meetup

March 31, 2016

http://joneisen.me/talk-k8s-app-config

----

## 3 Core Problems in Cluster Management

- Service Discovery
- Application Monitoring
- Application Configuration

----

## 3 Core Problems in Cluster Management

- <span style="text-decoration:line-through">Service Discovery</span>
- <span style="text-decoration:line-through">Application Monitoring</span>
- **Application Configuration**

We'll talk about App Config today.

----

## Application Configuration Defined

**Required**: Get configs into app

**Optional**:

- Limit app-specific code
- Reload app on config change
- Easy to use system

----

## Some Solutions

- Bundle configs into image
- Etcd
- Kubernetes Secrets
- Kubernetes (v1.2) ConfigMap

----

### Bundle Configs into Image

```
COPY config.yml /app/config.yml
```
```
ENV MY_VAR=yoyoma
```

____


### Bundle Configs into Image

- Images can be shipped and run easily.
- Versions config with image

---

- K8s doesn't support different image per pod in RC
- Requires build server even at low scale
- Requires many images per app (storage cost)
- Manual reload-on-change

----

### Consistent Database

Like etcd or Zookeeper

```
/configs/DB_HOST
/configs/DB_PORT
/configs/app/SPECIAL_APP_VAR
```

____

### Consistent Database

- Store whatever you want in `etcd`
- Reload-on-change easy to implement

---

- Requires app-specific code or special container in pod
- Works outside of k8s
- Must use raw `etcdctl` or ZKUI to edit

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

----

### Using Secrets

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


### Using Secrets

- Use them as files ready for use in a program (e.g. ssh keys)
- Use them as environment configurations

```
KEY1=VALUE1
KEY2=VALUE2
```

*Protip*: You can read in environment variables using a Docker `ENTRYPOINT`

____

### Using Secrets

- **Built-in** to Kubernetes v1.0.
- Automagically mount into your containers
- Stores arbitrary data

---

- No reload-on-change
- Requires base64 encoding to use (until v1.2)
- Doesn't support environment variables natively

----

### ConfigMap

```
kubectl create configmap my-config \
  --from-literal me.how=very \
  --from-literal me.type=charm
```

Or just use the regular `create -f my-config.yaml`

---

```
env:
  - name: MY_CONFIG_HOW
    valueFrom:
      configMapKeyRef:
        name: my-config
        key: me.how
  - name: MY_CONFIG_TYPE
    valueFrom:
      configMapKeyRef:
        name: my-config
        key: me.type
```
____


### ConfigMap

A [feature](http://kubernetes.io/docs/user-guide/configmap/) in Kubernetes v1.2

Like Secrets, but:

- Directly supports environment variables
- No base64 cumbersomeness
- Reload-on-change capability

----

## Conclusion

- Bundling configs into your images is too much work!
- Using etcd or ZK is powerful but a lot of work
- Secrets are great, but have problems
- Upgrade to v1.2 to get `ConfigMap`!

----

# Thanks!

Jon Eisen, @jm_eisen

Activision Analytic Services

Blog: http://joneisen.me


<img src="img/tyson-yoga.jpg" style="float:left" height="250px">
<img src="img/tyson-love.jpg" style="float:right" height="250px">
