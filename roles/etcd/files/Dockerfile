FROM    alpine as etcd-builder
ARG     VERSION='3.3.18'
RUN     mkdir -p /opt/etcd && apk add --no-cache curl wget
WORKDIR /opt/etcd
RUN     wget https://storage.googleapis.com/etcd/v${VERSION}/etcd-v${VERSION}-linux-amd64.tar.gz && \
        tar xf *.tar.gz && rm *.tar.gz && cp etcd-v${VERSION}-linux-amd64/etcd /etcd && \
        cp etcd-v${VERSION}-linux-amd64/etcdctl /etcdctl

FROM    alpine
COPY    --from=etcd-builder /etcd /usr/bin/etcd
COPY    --from=etcd-builder /etcdctl /usr/bin/etcdctl
CMD     [ "/usr/bin/etcd" ]