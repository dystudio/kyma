FROM golang:1.10-alpine as builder

ENV SRC_DIR /go/src/github.com/kyma-project/kyma/tests/knative-build/
WORKDIR ${SRC_DIR}

COPY ./knative_build_test.go ${SRC_DIR}/
COPY ./before-commit.sh ${SRC_DIR}/
COPY ./Gopkg.lock ${SRC_DIR}/
COPY ./Gopkg.toml ${SRC_DIR}/

RUN apk add bash dep git && ${SRC_DIR}/before-commit.sh
RUN go test -c ./ -o /knative_build.test

FROM alpine:3.8

LABEL source = git@github.com:kyma-project/kyma.git

COPY --from=builder /knative_build.test /knative_build.test

ENTRYPOINT [ "/knative_build.test", "-test.v" ]
