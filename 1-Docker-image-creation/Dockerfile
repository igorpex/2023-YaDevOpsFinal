ARG GOARCH=amd64
FROM busybox:1.35.0-uclibc as busybox
FROM gcr.io/distroless/base-debian12:latest-${GOARCH}
# Copy the static shell into base image.
COPY --from=busybox /bin/sh /bin/sh

# Copy all necessary executables into distroless image.
COPY --from=busybox /bin/mkdir /bin/mkdir
COPY --from=busybox /bin/adduser /bin/adduser
# COPY --from=busybox /bin/chpasswd /bin/chpasswd
ENTRYPOINT ["/bin/sh", "/entrypoint.sh"]

# Create non-root user
RUN adduser bingo --disabled-password
# Create folder for config and give rights
RUN mkdir /opt/
RUN mkdir /opt/bingo/
RUN chown bingo /opt/bongo/

# Create folder for logs and give rights
RUN mkdir /opt/bongo/logs/dff21986f6/
RUN chown bingo /opt/bongo/logs/dff21986f6/

# Switch user
USER bingo
# Copy app
COPY src/app/ .
# Copy default config (working config will be mapped later)
COPY src/config/ /opt/bingo/
EXPOSE 33719

CMD ["./bingo run_server"]