FROM  python:3.10.14-alpine3.20 AS base

# --- builder ---
FROM base AS builder

COPY setup.py ./
COPY nerve ./nerve

COPY examples/nerve-clt ./nerve-clt

RUN python setup.py bdist_wheel && cd nerve-clt && python setup.py bdist_wheel

# --- release ---
FROM base

WORKDIR /nerve

RUN mkdir -p pkg

COPY --from=builder dist/*.whl pkg
COPY --from=builder nerve-clt/dist/*.whl pkg

RUN pip3 install pkg/*.whl

ENTRYPOINT [ "nerve" ]
