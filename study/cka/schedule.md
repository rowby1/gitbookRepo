# Schedule

## Schedule의 작동방식

* **pod를 정의하는  yaml파일**에서  `spec:nodeName`  을 참조
* **지정 되있지 않다면** 쿠버네티스가 **자동**으로 추가

{% code title="pod-definition.yaml" overflow="wrap" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: nginx
    labels: 
        name: nginx
spec:
    containers:
    - name: nginx
      image: nginx
      ports:
          - containerPort: 8080
    nodeName: node02
```
{% endcode %}

#### 이미 생성된 Pod를 Node에할당하기&#x20;

{% code title="pod-bind-definition.yaml" overflow="wrap" %}
```yaml
apiVersion: v1
kind: Binding
metadata:
    name: nginx
target:
    apiVersion: v1
    kind: Node
    name: node02
```
{% endcode %}

위 형식  으로 api  요청

{% code overflow="wrap" %}
```bash
curl --header " Content-Type:application/json" --request POST --data '{"apiVerson" : "v1", "kind": "Binding" ...}'
http://$SERVER/api/v1/namespace/default/pods/$PODNAME/binding/
```
{% endcode %}

## Problems

### Problem.2

#### Q. What is the status of the created POD?

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

A. Pending

### Problem.3

#### Q. Why is the POD in a pending state?

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Node가 할당되지 않음. 즉, `스케줄링`되지 않은 상태\
A. no scheduler present

### Problem.4

Q. Manually schedule the pod on `node01`.

> Delete and recreate the POD if necessary.

A. 이미  생성된 pod가  있고, 새로운  내용을  적용시켜야한다.

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash"><strong>vi nginx.ymal # spec:nodeName부분 수정
</strong><strong>kubectl delete pod &#x3C;pod-name>
</strong>kubectl apply -f &#x3C;yaml-file>.yaml
</code></pre>

#### - 수정된 YAML 파일 적용 (Deployment/ReplicaSet가 없는 경우)

Pod는 **불변(immutable)**이므로 기존 Pod 삭제 후 **수정된 YAML으로 새로운 Pod 생성**

pod-definition.yaml 의 spec:nodeName 부분을 수정

{% code title="pod-definition.yaml" overflow="wrap" %}
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: nginx
    labels: 
        name: nginx
spec:
    containers:
    - name: nginx
      image: nginx
      ports:
          - containerPort: 8080
    nodeName: node02
```
{% endcode %}

#### - Deployment/ReplicaSet가 있는 경우

Pod가 단순한 독립 객체가 아니라 `Deployment`, `ReplicaSet`, `StatefulSet`과 같은 컨트롤러에 의해 관리된다면 해당 **리소스의 YAML 수정이 필요**

```bash
kubectl apply -f <deployment-file>.yaml
```

<details>

<summary>Deployment 수정시 Rolling Update</summary>

Deployment는 수정된 스펙에 맞춰 기존 Pod를 순차적으로 삭제하고 새로운 Pod 생성 (Rolling Update)

</details>

#### - 기존 Pod를 편집하는 방법

```bash
kubectl edit pod <pod-name>
```

### Problem.5

Q. Now schedule the same pod on the `controlplane` node.

[#problem.4](schedule.md#problem.4 "mention")과 상동.



