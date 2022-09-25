FROM klakegg/hugo:edge-ext-alpine
RUN apk update && apk --no-cache add git tzdata && \
  cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && \
  apk del tzdata && \
  git clone https://github.com/qqhann/hugo-primer.git themes/hugo-primer
