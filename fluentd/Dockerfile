# fluentd/Dockerfile

FROM fluent/fluentd:v1.12.0-debian-1.0
USER root

# add plugins here
RUN gem install fluent-plugin-sentry

USER fluent

COPY fluent.conf /fluentd/etc/