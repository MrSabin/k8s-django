FROM nginx/unit:1.25.0-python3.9
ENV PYTHONUNBUFFERED 1
ENV PYTHONDONTWRITEBYTECODE 1

WORKDIR /webapp
COPY requirements.txt /webapp/
RUN apt-get update && apt-get install -y python3-pip                            \
  && pip3 install --no-cache-dir -r requirements.txt                            \
  && apt-get remove -y python3-pip                                              \
  && apt-get autoremove --purge -y                                              \
  && rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/*.list
COPY unit_config.json /docker-entrypoint.d/
COPY src /webapp/src
WORKDIR /webapp/src
RUN SECRET_KEY=empty python manage.py collectstatic --noinput
EXPOSE 80
VOLUME ["/media"]
