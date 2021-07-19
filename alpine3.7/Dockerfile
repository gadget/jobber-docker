FROM golang:1.11-alpine
MAINTAINER C. Dylan Shearer <dylan@nekonya.info>

ENV JOBBER_VER v1.4.4
ENV SRC_HASH 4ecdfbc9ca092e1514b78c7efc5660adec44163b5c33e36a7b147ec2683f7803

RUN apk update && \
    apk upgrade && \
    apk add --update make gcc musl-dev rsync grep ca-certificates openssl wget

WORKDIR /go_wkspc/src/github.com/dshearer
RUN wget "https://api.github.com/repos/dshearer/jobber/tarball/${JOBBER_VER}" -O jobber.tar.gz && \
    echo "${SRC_HASH}  jobber.tar.gz" | sha256sum -cw && \
    tar xzf *.tar.gz && rm *.tar.gz && mv dshearer-* jobber && \
    cd jobber && \
    make check && \
    make install DESTDIR=/jobber-dist/

FROM alpine:3.7

RUN apk add --no-cache curl
RUN mkdir /jobber
COPY --from=0 /jobber-dist/usr/local/libexec/jobberrunner /jobber/jobberrunner
COPY --from=0 /jobber-dist/usr/local/bin/jobber /jobber/jobber
COPY --from=0 /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=0 /jobber-dist/etc/jobber.conf /etc/jobber.conf
ENV PATH /jobber:${PATH}

ENV USERID 1000
RUN adduser -S -u "${USERID}" -G users jobberuser && \
    mkdir -p "/usr/local/var/jobber/${USERID}" && \
    chown -R jobberuser:users "/usr/local/var/jobber/${USERID}"

VOLUME /jobber-storage
USER jobberuser
ENTRYPOINT ["jobberrunner"]
CMD ["-u", "/usr/local/var/jobber/1000/cmd.sock", "/jobber-storage/.jobber"]
