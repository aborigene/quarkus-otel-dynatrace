# Laboratório de Quarkus

Demo baseada em microserviços quarkus para demonstrar como monitorar esse tipo de tecnologia com Dynatrace

![Architecture](https://github.com/aborigene/quarkus-otel-dynatrace/blob/main/images/arquitetura.jpeg)

## Explicando a aplicação

A aplicação em si é bastante simples compostas de uma API rest que recebe carga de um load generator e envia requests a uma segunda API rest

1. Rest Client
2. Rest JSON

## Executar a Demo

Essa demo deve ser executada em um Kubernetes, ela pode ser executada em outros ambientes mas os artefatos de configuração terão que ser gerados. Os arquivos neste repo são para deploy em Kubernetes. A demo pode ser executada de duas formas: usando as imagens pré-construíadas e apenas fazendo o deploy da aplicação ou fazendo o build completo. Se você preferir usar as imagens pré-construídas, vá para o passo X, caso contrário comece o laboratório à partir do passo 1 abaixo.

1. Build do rest-client

Esta é uma aplicação muito simples baseada nos exemplos de quickstart do quarkus e alterada para atender esse laboratório. Para fazer o build vc precisa de um ambiente docker rodando e executar os comandos abaixo:

```
cd <repo_hom>/src/rest-client-quickstart
docker build . -t <docker hub repo>/rest-client-quickstart:v0.1
docker push <docker hub repo>/rest-client-quickstart:v0.1
```

2. Build do rest-json

Esta é uma aplicação muito simples baseada nos exemplos de quickstart do quarkus e alterada para atender esse laboratório. Para fazer o build vc precisa de um ambiente docker rodando e executar os comandos abaixo:

```
cd <repo_hom>/src/rest-json-quickstart
docker build . -t <docker hub repo>/rest-json-quickstart:v0.1
docker push <docker hub repo>/rest-json-quickstart:v0.1
```

3. Configurar coletor OpenTelemetry

Aqui temos duas opções: instalar um coletor ou usar um coletor já existente no cluster, escolher uma das opções 1.1 ou 1.2 abaixo:

  1. Instalar um coletor:

Entrar no Dynatrace e gerar um token para a ingestão de traces OpenTelemetry

Em seguida executar os comandos abaixo:

```
  cd <repo_home>/kubernetes/opentelemetry
  kubectl create ns otel
  kubectl -n otel create secret generic otel-collector-secret --from-literal "OTEL_ENDPOINT_URL=<TENANT-BASEURL>/api/v2/otlp" --from-literal "OTEL_AUTH_HEADER=Api-Token <API-TOKEN>" #preencher com os dados do seu ambiente e o token gerado anteriormente
  kubectl -n otel apply -f opentelemetry.yaml
```

  2. Já possui um collector

Editar o arquivo:

  <repo_home>/kubernetes/quarkus/rest-client-quickstart/rest-client-quickstart.yaml

Alterar o início do arquivo conforme abaixo:

```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: rest-client-quickstart-config
data:
  OTEL_ENDPOINT_URL: grpc://<nome do serviço>.<nome do namespace>:<porta>
```

3. Subir os serviços

Aqui é importante que se você construiu as imagens, que você altere os arquivos de deploy para apontar para as imagens que foram construídas:

Editar o arquivo:

```
vim <repo_home>/kubernetes/quarkus/rest-client-quickstart/rest-client-quickstart.yaml
```

Alterar a linha da imagem para apontar para a imagem criada:

```
    spec:
      containers:
        - name: rest-client-quickstart
          image: <docker hub repo>/rest-client-quickstart:v0.1
```

Em seguida fazer o mesmo para o outro serviço:

```
vim <repo_home>/kubernetes/quarkus/rest-client-quickstart/rest-json-quickstart.yaml
```

Alterar a linha da imagem para apontar para a imagem criada:

```
    spec:
      containers:
        - name: rest-client-quickstart
          image: <docker hub repo>/rest-json-quickstart:v0.1
```

Em seguida executar o deploy:

```
  cd <repo_home>/kubernetes/quarkus
  kubectl apply -f rest-client-quickstart/rest-client-quickstart.yaml # isso irá subir o primeiro serviço no namespace default
  kubectl apply -f rest-json-quickstart/rest-json-quickstart.yaml # isso irá subir o segundo serviço no namespace default, não trocar o namespace aqui
```

4. Resumo do ambiente

Aqui você terá o primeiro serviço monitorado puramente com OpenTelemetry já que é um serviço nativo e o segundo sendo monitorado com o OneAgent, já que é um serviço Java.

O serviço principal está exposto vai LoadBalancer, basta então mandar uma requisição à api exposta conforme abaixo:

```
curl http://<ip_real>:8086/extension/id/io.quarkus:quarkus-rest-client
```

5. Outros cenários

Para testar outros cenários basta setar a variável NATIVE em cada deploy. Esta variável já está setada no serviço client, e não está setada no outro serviço. Basta setar essa variável com qualquer valor para mudar de java para native.

### Tendo problemas ou dificuldades ou encontrou um bug?

Envie um email: [igor.simoes@dynatrace.com](mailto:igor.simoes@dynatrace.com), [krishnarupa@gmail.com](mailto:krishnarupa@gmail.com)
