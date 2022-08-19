

# Two approaches to build multiple arch platform images

```bash
# for both approaches we need a local registry
docker run -d -p 5001:5000 -v registry_data:/var/lib/registry --restart=always --name registry registry:2
```

## Use BuildKit (buildx)
```bash
docker buildx create --name mybuilder --driver-opt network=host --use
docker buildx inspect --bootstrap

docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --tag localhost:5001/myappx:latest \
  --output=type=registry,registry.insecure=true ./alpine

# check the image manifest list from the registry:
docker buildx imagetools inspect localhost:5001/myappx:latest

docker run --platform linux/amd64 --rm localhost:5001/myappx
docker run --platform linux/arm64 --rm localhost:5001/myappx
```

## Use docker build (old-school)
```bash

# first let's build the multiple architectures and tag them appropriately:
docker build --platform linux/amd64 --tag localhost:5001/myapp:amd64 ./alpine/
docker build --platform linux/arm64 --tag localhost:5001/myapp:arm64 ./alpine/

# second push the images
docker push localhost:5001/myapp:amd64
docker push localhost:5001/myapp:arm64

# now let's combine the manifests into a single list and push it
docker manifest create localhost:5001/myapp:latest \
  --amend localhost:5001/myapp:amd64 \
  --amend localhost:5001/myapp:arm64 \
  --insecure

docker manifest push localhost:5001/myapp:latest

# check the image manifest list from the registry:
docker buildx imagetools inspect localhost:5001/myapp:latest
```


# references

- https://docs.docker.com/build/buildx/multiplatform-images/
- https://salesjobinfo.com/multi-arch-container-images-for-docker-and-kubernetes/