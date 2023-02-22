FROM node:lts-slim

RUN apt update \
  && apt install -y task-japanese locales locales-all \
  && locale-gen ja_JP.UTF-8 \
  && echo "export LANG=ja_JP.UTF-8" >> ~/.bashrc

WORKDIR /workspace

ADD .husky/pre-commit ./.husky/pre-commit
ADD package.json ./package.json
ADD yarn.lock ./yarn.lock

RUN chmod +x .husky/pre-commit && apt-get update && apt-get install -y git yarn && yarn

EXPOSE 8000
