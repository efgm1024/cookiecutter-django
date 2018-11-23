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
  && apk add libffi-dev py-cffi \
  # Translations dependencies
  && apk add gettext \
  # https://docs.djangoproject.com/en/dev/ref/django-admin/#dbshell
  && apk add postgresql-client

# Requirements are installed here to ensure they will be cached.
COPY ./Pipfile /app/Pipfile
RUN mkdir -p /app/.venv && \
    cd /app && \
    pip install pipenv && \
    pipenv install --dev

COPY ./compose/production/django/entrypoint /entrypoint
RUN sed -i 's/\r//' /entrypoint && \
    chmod +x /entrypoint

COPY ./compose/local/django/start /start
RUN sed -i 's/\r//' /start && \
    chmod +x /start
{% if cookiecutter.use_celery == "y" %}
COPY ./compose/local/django/celery/worker/start /start-celeryworker
RUN sed -i 's/\r//' /start-celeryworker && \
    chmod +x /start-celeryworker

COPY ./compose/local/django/celery/beat/start /start-celerybeat
RUN sed -i 's/\r//' /start-celerybeat && \
    chmod +x /start-celerybeat

COPY ./compose/local/django/celery/flower/start /start-flower
RUN sed -i 's/\r//' /start-flower && \
    chmod +x /start-flower
{% endif %}

WORKDIR /app

ENTRYPOINT ["/entrypoint"]
