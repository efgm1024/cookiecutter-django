FROM python:3.7-alpine

ENV PYTHONUNBUFFERED 1
ENV PIPENV_VENV_IN_PROJECT 1
ENV PIPENV_TIMEOUT 10000

RUN apk update \
  # psycopg2 dependencies
  && apk add --virtual build-deps gcc python3-dev musl-dev \
  && apk add postgresql-dev \
  # Pillow dependencies
  && apk add jpeg-dev zlib-dev freetype-dev lcms2-dev openjpeg-dev tiff-dev tk-dev tcl-dev \
  # CFFI dependencies
  && apk add libffi-dev py-cffi

RUN addgroup -S django \
    && adduser -S -G django django

# Requirements are installed here to ensure they will be cached.
COPY ./Pipfile /app/Pipfile
RUN mkdir -p /app/.venv && \
    cd /app && \
    pip install pipenv && \
    pipenv install

COPY ./compose/production/django/entrypoint /entrypoint
RUN sed -i 's/\r//' /entrypoint && \
    chmod +x /entrypoint && \
    chown django /entrypoint

COPY ./compose/production/django/start /start
RUN sed -i 's/\r//' /start && \
    chmod +x /start && \
    chown django /start
{% if cookiecutter.use_celery == "y" %}
COPY ./compose/production/django/celery/worker/start /start-celeryworker
RUN sed -i 's/\r//' /start-celeryworker && \
    chmod +x /start-celeryworker && \
    chown django /start-celeryworker

COPY ./compose/production/django/celery/beat/start /start-celerybeat
RUN sed -i 's/\r//' /start-celerybeat && \
    chmod +x /start-celerybeat && \
    chown django /start-celerybeat

COPY ./compose/production/django/celery/flower/start /start-flower
RUN sed -i 's/\r//' /start-flower && \
    chmod +x /start-flower
{% endif %}
COPY . /app

RUN chown -R django /app

USER django

WORKDIR /app

ENTRYPOINT ["/entrypoint"]
