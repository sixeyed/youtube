# Speed up your builds with BuildKit and buildx

> The video for this Docker Nugget is here: https://youtu.be/YX2BSioWyhI

> The docs for the demos are here: https://is.gd/cGD5Gl

## Intro

- Original build engine part of Docker
- Input: Dockerfile, output: Docker image
- Build cache for each stage but sequential processing

- BuildKit is a separate engine
- Input: multiple (inc. Dockerfile); output multiple (inc. Docker image)
- Opt-in with Docker CLI, limited options available
- Use with buildx to access all options

## BuildKit vs. `docker build`

```
$env:DOCKER_BUILDKIT='1'
```

> Build and test in parallel


## buildx

> Remote cache storage

## Hosted CI




## Links

- [BuildKit virtual meetup](https://www.docker.com/blog/january-virtual-meetup-recap/)
- [Multi-arch apps with buildx virtual meetup](https://www.docker.com/blog/docker-arm-virtual-meetup-multi-arch-with-buildx/)
- [BuildKit](https://github.com/moby/buildkit)
- [buildx docs](https://docs.docker.com/buildx/working-with-buildx/#high-level-build-options)
- [Learn Docker in a Month of Lunches](https://is.gd/diamol) - my book :)
