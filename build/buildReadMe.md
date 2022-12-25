# Build an Overview for Docker Hub

This is an instruction for generating `Markdown` content for **Docker Hub** Overview from `README.md` in github repository. The mainly work is to transfer the image links from local relative paths to the absolute urls.

```shell
cat README.md | sed -e 's/](\.\/imgs\//](https:\/\/github.com\/zhaolj\/hadoop_docker\/raw\/main\/imgs\//g' > README2.md
```
