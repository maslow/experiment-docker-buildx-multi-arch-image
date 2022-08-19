# syntax=docker/dockerfile:1-labs
FROM alpine:3.15

COPY <<-"SCRIPT" /info.sh
		#!/bin/sh
		echo "I am running under machine type (architecture):"
		uname -m
SCRIPT
RUN chmod +x /info.sh

CMD ["/info.sh"]
