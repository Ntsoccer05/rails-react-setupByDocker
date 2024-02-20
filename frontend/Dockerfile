FROM node:18.17.0-slim

ENV TZ Asia/Tokyo

WORKDIR /frontend/app

RUN apt-get update \
  && apt-get install -y

CMD ["npm","run","dev"]