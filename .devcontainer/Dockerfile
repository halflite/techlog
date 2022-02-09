FROM klakegg/hugo:0.92.1-alpine
RUN apk update && apk --no-cache add git tzdata && \
  cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && \
  apk del tzdata
