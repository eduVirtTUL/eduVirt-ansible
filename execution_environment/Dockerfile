ARG PYTHON_VERSION='3.12'
ARG PIP_VERSION='24.0'
ARG PY='python3.12'
ARG ANSIBLE_RUNNER_VERSION='2.4.0'
ARG ANSIBLE_CORE_VERSION='2.18.0'

FROM alpine:3.20.3 AS base
ARG PYTHON_VERSION
ARG PIP_VERSION
ARG PY

USER root

RUN apk update && apk add --no-cache --no-interactive \
    python3=~$PYTHON_VERSION \
    py3-pip=~$PIP_VERSION \
    openssh-client-default \
    bash

CMD ["sh"]

FROM base AS system

USER root

WORKDIR /build
COPY system-requirements.txt .
RUN apk update && apk add --no-cache --no-interactive $(cat system-requirements.txt)

FROM system AS ansible
ARG ANSIBLE_CORE_VERSION
ARG ANSIBLE_RUNNER_VERSION
ENV PIP_BREAK_SYSTEM_PACKAGES=1

USER root

RUN $PY -m pip install --no-cache-dir ansible-core==$ANSIBLE_CORE_VERSION ansible-runner==$ANSIBLE_RUNNER_VERSION pyyaml

# Directory for executing Playbooks
WORKDIR /runner/

# Add non-root user
ARG USER=ansible
ARG GROUP=ansible
ARG UID=1000
ARG GID=1000
RUN addgroup ${GROUP} --gid ${GID} && \
    adduser  ${USER}  --uid ${UID} \
    --ingroup "${GROUP}" \
    --disabled-password && \
    chown ${USER}:${GROUP} /runner/

RUN chmod 777 /runner/ /home/ansible/


FROM ansible AS python
ENV PIP_BREAK_SYSTEM_PACKAGES=1

USER root
WORKDIR /build
RUN ls
COPY python-requirements.txt .

RUN $PY -m pip install --no-cache-dir -r python-requirements.txt


FROM python AS galaxy

USER root
WORKDIR /build
COPY galaxy-requirements.yaml .

RUN ansible-galaxy collection install -r galaxy-requirements.yaml --collections-path "/usr/share/ansible/collections"
RUN ansible-galaxy role install -r galaxy-requirements.yaml --roles-path "/usr/share/ansible/roles"

ENV HOME=/home/ansible
 
# Switch to non-root user
USER ${UID}:${GID}
WORKDIR /home/ansible
CMD ["bash"]