# Sent2Vec with Kubernetes

![](arch.png)

## Usage

Creation:

~~~bash
kubectl apply -f .
~~~

Deletion:

~~~bash
kubectl delete -f .
~~~

## Notes

- The cluster must have at least one node with at least **6.25 GB memory** (see the memory request of the container in [backend.yml](backend.yml)).
- It may take up to **10 minutes** to download the `sent2vec-server` image, if it is not yet present locally on the nodes. 
- It takes around **3 minutes** for the `sent2vec-server` container to load the model. You can check the progress in the log output:

    ~~~bash
    kubectl logs -f --timestamps=true deployment/sent2vec-engine
    ~~~

## Dependencies

The application requires the following external services:

- RabbitMQ server
- Amazon S3 bucket

The RabbitMQ server is for the communication between the Web UI and the engine. The Amazon S3 bucket is used to store data by the engine.

See below how to specify the credentials for these services.

## Secrets

### Define a Kubernetes Secret

For the deployment to work, you must first specify the credentials for the above services in an **additional Kubernetes file**. The file must have the following content (it can have any name, but `secrets.yml` makes sense):

~~~yaml
apiVersion: v1
kind: Secret
metadata:
  name: sent2vec-secrets
  labels:
    app: sent2vec
type: Opaque
data:
  rabbitmq-uri: <BASE64>
  s3-bucket-name: <BASE64>
  aws-access-key-id: <BASE64>
  aws-secret-access-key: <BASE64>
~~~

You have to replace `<BASE64>` with the [Base64](https://en.wikipedia.org/wiki/Base64) encoding of the individual secrets.

You can get the Base64 encoding of any string with:

~~~bash
echo -n foo | base64
~~~

### Example

For example, if your RabbitMQ URI is `amqp://user:password@host/vhost`, then execute the following:

~~~bash
echo -n amqp://user:password@host/vhost | base64
~~~

This returns the Base64 string `YW1xcDovL3VzZXI6cGFzc3dvcmRAaG9zdC92aG9zdA==`, which you have to put as the value of the `rabbitmq-uri` key in the file:

~~~yaml
...
  rabbitmq-uri: YW1xcDovL3VzZXI6cGFzc3dvcmRAaG9zdC92aG9zdA==
...
~~~

Do the same with the AWS access key ID and AWS secret access key of an AWS account, and the name of a S3 bucket in this AWS account. This S3 bucket is used as a database by the `sent2vec-server` container.

### Run

After creating the secrets file, just run the command indicated above to deploy and start the entire application:

~~~bash
kubectl apply -f .
~~~

### Important

Don't forget the `-n` option of `echo` when creating the Base64 encodings. Otherwise, the newline added by `echo` is also included in the encoding, which causes the application to crash.
