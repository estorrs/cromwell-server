FROM broadinstitute/cromwell:78-38cd360

RUN apt-get update && apt-get install -y vim wget curl git

RUN git clone https://github.com/vishnubob/wait-for-it.git
RUN mkdir cromwell-working-dir
WORKDIR cromwell-working-dir

COPY ./app-config /app-config

ENTRYPOINT ["/bin/sh", "-c"]
