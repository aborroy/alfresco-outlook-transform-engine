# Deploying Alfresco Outlook Transform Engine with Docker Compose

This project includes a Docker Compose template to provide a sample configuration that uses the [Alfresco Outlook Transform Engine](https://docs.alfresco.com/microsoft-outlook/latest/install/#install-transform-engine) to deliver high-fidelity preview for Outlook and RFC822 email messages.

* [.env](.env) specifies Alfresco Services version to be used by Docker Compose
* [compose.yaml](compose.yaml) describes Docker Compose deployment

>> Note this is a sample deployment designed for education purposes. When using Alfresco Transform Service in real world, additional requirements should impact in the design of the final deployment.

Docker Images from [quay.io](https://quay.io/organization/alfresco) are used, since this product is only available for Alfresco Enterprise customers. If you are Enterprise Customer or Partner but you are still experimenting problems to download Docker Images, contact [Alfresco Hyland Support](https://community.hyland.com) in order to get required credentials and permissions.


## Configuration details

Alfresco Outlook Transform Engine is deployed as an individual service.

```
  transform-outlook:
    image: quay.io/alfresco/transform-outlook:${TRANSFORM_ENGINE_OUTLOOK_TAG}
    environment:
      ACTIVEMQ_URL: "nio://activemq:61616"
      FILE_STORE_URL: "http://shared-file-store:8099/alfresco/api/-default-/private/sfs/versions/1/file"
      TRANSFORM_ENGINE_REQUEST_QUEUE: "org.alfresco.transform.engine.outlook.acs"
```

To ensure that *synchronous* preview renditions are working when using Share App, following environment variable must be added to `alfresco` service.

```
  alfresco:
    image: quay.io/alfresco/alfresco-content-repository:${ALFRESCO_TAG}
    environment:
      JAVA_OPTS: "
        -DlocalTransform.transform-outlook.url=http://transform-outlook:8090/
        "
```

When using ADW, renditions are generated *asynchronously*, so following configuration must be applied to `transform-router` service.

```
  transform-router:
    image: quay.io/alfresco/alfresco-transform-router:${TRANSFORM_ROUTER_TAG}
    environment:
      TRANSFORMER_URL_OUTLOOK: "http://transform-outlook:8090"
      TRANSFORMER_QUEUE_OUTLOOK: "org.alfresco.transform.engine.outlook.acs"  
```

>> Verify additional configurations details in [compose.yaml](compose.yaml) Docker Compose template

## Using

```
docker-compose up --build --force-recreate
```

## Service URLs

* Share: http://localhost:8080/share
* ADW: http://localhost:8080/workspace
* Transform Router: http://localhost:8095/
* Outlook Transform Engine: http://localhost:8091

>> Default credentials are `admin`/`admin`
