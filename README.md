# Endpoints GRPC multiple backends

Using [bookstore from endpoints samples](https://github.com/GoogleCloudPlatform/python-docs-samples/tree/master/endpoints/bookstore-grpc), and [helloworld service from GRPC examples](http://www.grpc.io/docs/quickstart/python.html).

#### Combining multiple GRPC services into a single service config

Replace YOUR_SERVICE_NAME to real one in `api_config.yaml`.

```
$ protoc --include_imports --include_source_info bookstore.proto helloworld.proto --descriptor_set_out=out.pb

$ gcloud beta service-management deploy out.pb api_config.yaml
```

Get service-name and config-id from the command output.

#### Deploy GRPC backends

```
$ kubectl create -f bookstore-svc.yaml
$ kubectl create -f helloworld-svc.yaml
```

#### Create custom nginx.conf and deploy to configMap

Note there should be `location` and `grpc_pass` for each backends. The path for each location is "proto-package" . "proto-service" (Service-Name in [GRPC Wire protocol](http://www.grpc.io/docs/guides/wire.html#appendix-a---grpc-for-protobuf))

```
location /helloworld.Greeter {
  grpc_pass helloworld:80 override;
}
location /endpoints.examples.bookstore.Bookstore {
  grpc_pass bookstore:80 override;
}
```

Deploy it to Kubernetes

```
$ kubectl create configmap nginxconf --from-file=nginx.conf
```

#### Create ESP service

Replace YOUR_SERVICE_NAME and YOUR_SERVICE_CONFIG_ID to real one in `esp-svc.yaml`.

```
$ kubectl create -f esp-svc.yaml
```
