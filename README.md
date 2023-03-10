### Deploy Microservices with Helmfile

### Technologies Used:

Kubernetes, Helm, Helmfile

### Project Description:

1- Create 1 shared Helm Chart for all microservices, to reuse common Deployment and Service configurations for the services

2- Deploy Microservices with Helm

3- Deploy Microservices with Helmfile

### Instructions:

###### Step 1: Create a helm chart structure

```
helm create microservices
```

###### Step 2: Create two helm chart template : deployment and service

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appName }}
spec:
  replicas: {{ .Values.appReplicas }}
  selector:
    matchLabels:
      app: {{ .Values.appName }}
  template:
    metadata:
      labels:
        app: {{ .Values.appName }}
    spec:
      containers:
      - name: {{ .Values.appName }}
        image: "{{ .Values.appImage }}:{{ .Values.appVersion }}"
        ports:
        - containerPort: {{ .Values.containerPort }}
        env:
        {{- range .Values.containerEnvVars}}
        - name: {{ .name }}
          value: {{ .value | quote }}
        {{- end}}

```

```

258 bytes
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.appName }}
spec:
  type: {{ .Values.serviceType }}
  selector:
    app: {{ .Values.appName }}
  ports:
  - protocol: TCP
    port: {{ .Values.servicePort }}
    targetPort: {{ .Values.containerPort }}
```

###### Step 3: Create a default value for microservices

```
appName: servicename
appImage: gcr.io/google-samples/microservices-demo/servicename
appVersion: v.0.0.0
appReplicas: 1
containerPort: 8080
containerEnvVars:
- name: ENV_VAR_ONE
  value: "valueone"
- name: ENV_VAR_TWO
  value: "valuetwo"

servicePort: 8080
serviceType: ClusterIP

```

###### Step 4: Create two helm chart templates for redis: deployment and service

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: { { .Values.appName } }
spec:
  replicas: { { .Values.appReplicas } }
  selector:
    matchLabels:
      app: { { .Values.appName } }
  template:
    metadata:
      labels:
        app: { { .Values.appName } }
    spec:
      containers:
        - name: { { .Values.appName } }
          image: "{{ .Values.appImage }}:{{ .Values.appVersion }}"
          ports:
            - containerPort: { { .Values.containerPort } }
          volumeMounts:
            - name: { { .Values.volumeName } }
              mountPath: { { .Values.containerMountPath } }
      volumes:
        - name: { { .Values.volumeName } }
          emptyDir: {}
```

```
apiVersion: v1
kind: Service
metadata:
  name: { { .Values.appName } }
spec:
  type: ClusterIP
  selector:
    app: { { .Values.appName } }
  ports:
    - protocol: TCP
      port: { { .Values.servicePort } }
      targetPort: { { .Values.containerPort } }
```

###### Step 5: Create a default value for redis

```
appName: redis
appImage: redis
appVersion: alpine
appReplicas: 1
containerPort: 6379
volumeName: redis-data
containerMountPath: /data

servicePort: 6379
```

###### Step 6: Install helmfile tool

```
brew install helmfile
```

###### Step 7: Create a helmfile

```
releases:
  - name: rediscart
    chart: charts/redis
    values:
      - values/redis-values.yaml
      - appReplicas: "1"
      - volumeName: "redis-cart-data"

  - name: emailservice
    chart: charts/microservice
    values:
      - values/email-service-values.yaml

  - name: cartservice
    chart: charts/microservice
    values:
      - values/cart-service-values.yaml

  - name: currencyservice
    chart: charts/microservice
    values:
      - values/currency-service-values.yaml

  - name: paymentservice
    chart: charts/microservice
    values:
      - values/payment-service-values.yaml

  - name: recommendationservice
    chart: charts/microservice
    values:
      - values/recommendation-service-values.yaml

  - name: productcatalogservice
    chart: charts/microservice
    values:
      - values/productcatalog-service-values.yaml

  - name: shippingservice
    chart: charts/microservice
    values:
      - values/shipping-service-values.yaml

  - name: adservice
    chart: charts/microservice
    values:
      - values/ad-service-values.yaml

  - name: checkoutservice
    chart: charts/microservice
    values:
      - values/checkout-service-values.yaml

  - name: frontendservice
    chart: charts/microservice
    values:
      - values/frontend-values.yaml
```

###### Step 8: Create a cluster with 3 node on Linode

###### Step 9: Download microservice_kubeconfig.yaml and change its permissions

```
chmond 600 microservice_kubeconfig.yaml
```

###### Step 10: Deploy microservice via helmfile

```
helmfile sync
```

###### Step 11: Open the load balancer ip in browser(as the frontend use loadbalancer type for external access)

###### Step 11: Check current release

```
helmfile list
```

###### Step 12: Uninstall release

```
helmfile destroy
```
