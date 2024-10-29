# Labels and Selectors



## 쿠버네티스 환경변수 <a href="#undefined" id="undefined"></a>

* 도커 이미지는 한번 빌드되면, 불변 이므로 유연한 설정의 필요성이 생긴다.
* 쿠버네티스는 YAML 파일과 환경변수를 분리할 수 있는 방식으로 `컨피그맵(ConfigMap)`과 `시크릿(Secret)` 오브젝트를 제공한다.

{% code title=" pod-definition.yaml" overflow="wrap" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080

      # 아래 env 부분에서 환경 변수를 설정한다.
      env:
        - name: APP_COLOR # 컨테이너가 사용할 수 있는 환경 변수의 이름
          value: pink # 환경 변수의 값
```
{% endcode %}

### 컨피그맵(ConfigMap)

* **Pod정의 YAML 파일  밖**에서 설정값을 관리할 수 있는 것이 필요하다.
* 컨피그맵은 **key-value의 형태**로 환경변수 데이터를 전달 할 수 있다.
* 컨피그맵은 **Pod가 생성될때 따로 주입**된다.



컨피그맵의 방식은 **선언형(declarative)**과 **명령형(imperative)**이 있다.

#### 명령형

```bash
kubectl create configmap {컨피그맵 이름} --from-literal={키}={값}
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod

kubectl create configmap {컨피그맵 이름} --from-file={파일의 경로}
kubectl create configmap app-config --from-file=app_config.properties
```

{% code title="app_config.properties" overflow="wrap" %}
```
APP_COLOR=blue
APP_MODE=prod
```
{% endcode %}

`--from-literal` 옵션으로 key-value형태의 환경변수에 대한 값을 넘겨 컨피그맵을 생성 할 수 있다. \
`--from-file` 옵션으로 미리 정의된 properties 형식의 환경 변수로  컨피그맵을 생성 할 수 있다.

#### 선언형

{% code title="config-map.yaml" overflow="wrap" %}
```yaml
apiVersion: v1
kind: ConfigMap # kind는 ConfigMap으로 설정한다.
metadata:
  name: app-config
data: # 아래 부분에 ConfigMap 내용을 적어준다.
  APP_COLOR: blue
  APP_MODE: prod
```
{% endcode %}

```bash
kubectl create -f config-map.yaml

kubectl get configmaps
kubectl describe configmaps
```



#### Pod에 주입하기

컨피그맵을 **Pod 에 주입**할 수 있다.

{% code title=" pod-definition.yaml" overflow="wrap" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080

    # 방법1
    envFrom:
  - configMapRef:
      name: app-config # ConfigMap YAML 파일.
      
    # 방법2 하나의 키 값만 가져오는 경우
    env:
      - name: APP_COLOR
        valueFrom:
          configMapKeyRef:
            name: app-config # ConfigMap YAML 파일.
            key: APP_COLOR # 가져오고자 하는 키 값.
            
    # 방법3 볼륨에서 전체 데이터를 파일로 가져오는 경우
    volumes:
    - name: app-config-volume
      configMap:
        name: app-config # ConfigMap YAML 파일.

```
{% endcode %}



### 시크릿

환경변수에는 `패스워드`와 같이 **민감한 정보**가 있을 수 있다.

{% code title="secret-data.yaml" overflow="wrap" %}
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw= # echo -n 'mysql' | base64
  DB_User: cm9vdA== # echo -n 'root' | base64
  DB_Password: cGFzd3Jk # echo -n 'passwrd' | base64
```
{% endcode %}



#### Pod에 주입하기

{% code title="pod-definition.yaml" overflow="wrap" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080

      # 방법 1
      envFrom:
        - secretMapRef:
            name: app-secret # Secret 정의 YAML.

      # 방법 2 하나의 키 값만 가져오는 경우
      env:
        - name: DB_Password
          valueFrom:
            SecretKeyRef:
              name: app-secret # Secret 정의 YAML.
              key: DB_Password # 가져오고자 하는 키 값.
      
      # 방법 3 전체 데이터를 파일로 가져옴
      volumes:
      - name: app-secret-volume
        secret:
          name: app-secret # Secret 정의 YAML.

```
{% endcode %}







## Problems



### Problem1

#### Q.We have deployed a number of PODs. They are labelled with `tier`, `env` and `bu`. How many PODs exist in the `dev` environment (`env`)?

Use selectors to filter the output\
\
A.&#x20;

```
kubectl get pods --show-labels
```



### Problem2

#### Q. How many PODs are in the `finance` business unit (`bu`)?

\
A.&#x20;

```
kubectl get pods --show-labels
```



### Problem3

Q. How many objects are in the `prod` environment including PODs, ReplicaSets and any other objects?



A.

```
kubectl get all --show-labels
kubectl get all --selector  env=prod
```



### Problem4

#### Q. Identify the POD which is part of the `prod` environment, the `finance` BU and of `frontend` tier?

A.

```
kubectl get all --selector  env=prod,bu=finance,tier=frontend
```



### Problem5

#### Q. A ReplicaSet definition file is given `replicaset-definition-1.yaml`. Attempt to create the replicaset; you will encounter an issue with the file. Try to fix it.

Once you fix the issue, create the replicaset from the definition file

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

A.

{% tabs %}
{% tab title="에러 YAML" %}
<pre class="language-yaml"><code class="lang-yaml">apiVersion: apps/v1
kind: ReplicaSet
metadata:
   name: replicaset-1
spec:
   replicas: 2
   selector:
      matchLabels:
        tier: front-end
   template:
     metadata:
       labels:
        <a data-footnote-ref href="#user-content-fn-1">tier: nginx</a>
     spec:
       containers:
       - name: nginx
         image: nginx
</code></pre>
{% endtab %}

{% tab title="수정 YAML" %}
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
   name: replicaset-1
   labels:
      tier: nginx
spec:
   replicas: 2
   selector:
      matchLabels:
        tier: front-end
   template:
      metadata:
        labels:
          tier: front-end
      spec:
       containers:
       - name: nginx
         image: nginx
```
{% endtab %}
{% endtabs %}



















[^1]: template 부분은 pod의 정의라고 생각하면 된다 selctor:matchLabels에  정의 된 것과 같은 것만 적용 되므로 일치 해야 된다
