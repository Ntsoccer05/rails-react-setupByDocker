FROM node:18.17.0-slim

ENV TZ Asia/Tokyo

WORKDIR /frontend

RUN apt-get update \
  && apt-get install -y

CMD ["cd","app"]
CMD ["npm","run","dev"]